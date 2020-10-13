---
title: "Making the Same Stuff With Different Tools"
date: "Sun, Aug 25 2019"
layout: post
tags: C haxe flixel lua tiled
---

## A Zelda-like in C

In 2015, I began writing a Zelda-like called Wizwag. I
mentioned this project before in
[this post]({% post_url 2018-04-06-intimacy-of-bugs %}) about
corner-walking in 2D Zeldas and Zelda-likes. I also wrote
about the Haxe version's lighting system in
[this post.]({% post_url 2018-03-15-haxeflixel-lighting %})

I used C99 and Lua 5.2, wrote my own soft renderer, wrote my
first audio playback system, and my first multi-phase
physics engine. For those reasons, it was a valuable
learning experience. I also wanted to accurately capture the
"feel" of Gameboy Color era Zelda titles in the artwork and
gameplay.

I opted to use the supertile concept from those games, and
implemented a map editor in the game itself, since most
convenient tilemap editors don't support supertiles and
faking it is tedious. The problem was that I didn't _also_
write a tool to create the supertiles, so I manually made my
tilesets (yeesh).

Anyway, things got reasonably far along on the toolchain
front, less so the content front, aside from art and
tilesets:

![gif of in-game dialogue](/img/zeldascreens/transition-quantize.gif)

Entity scripting was made as painless as possible, with as
much live-reloading as I could figure out at the time.

Here's what an entity """prefab""" looked like in the game:

```lua
-- the file defining collectible money items --

local sprites = {
    rect.mindim(tile8(1,8), V2(8,8)),
    rect.mindim(tile8(0,9), V2(8,8)),
    rect.mindim(tile8(1,9), V2(8,8)),
    rect.mindim(tile8(2,8), V2(16,16)),
}

local spritei = math.random(1,#sprites)

return {
    type        = ETYPE_ITEM,
    facing      = FACING_DOWN,
    vram        = WIZ_VRAM_ITEM,
    sprite      = sprites[spritei],
    bb          = rect.mindim(v2(-4,-8), v2(8,8)),
    material    = EMAT_BOUNCY,

    triggers = {
        awake = function(self)
            self:jump(math.random(16,32))
            self:loadsound('cash.ogg')
            self.triggered = false

            self.value = 1
            if spritei == #sprites then
                self.value = 5
            end
        end,

        collide = function(self, other)
            if (self.triggered) then return end
            if (self.alive and other.player) then
                self.triggered = true

                Player.ChangeGold(self.value)
                self:sprite(rect.mindim(V2(0,0),V2(1,1)))
                self:sfx('cash.ogg', 0.8, math.random(75,125)/100)

                self:killafter(2)
            end
        end,
    }
}
```

And here's a snippet from the physics engine, which applied
material properties like ice, pits, and conveyors:

```cpp
void wizphys_apply_tile_materials(wiz_map *Map)
{
    rectangle MapBounds = rect_mindim(V2(0,0), V2(Map->w, Map->h));
    u8 Tile = 0;
    wiz_tile *Type = 0;

    wizphys_manifold *M;
    wiz_entity *E;

    for (int i=0; i<arraylen(Map->ents); ++i) {
        E = Map->ents + i;
        if (!flagcheck(E->flags, EFLAG_EXISTS)) continue;

        /* The tile material only matters for grounded entities */
        if (!E->offground) {
            v2 Resident = V2(E->p.x/16, E->p.y/16);

            /* Check map bounds */
            if (rect_contains(MapBounds, Resident))
            {
                /* Get the tile type */
                Tile = Map->tiles[Resident.x + Resident.y * Map->w];
                Type = Map->ts.types + Tile;

                /* Call the handler */
                wizphys_tilemat_handler Handler = __wizpyhs_mats[Type->material];
                if (Handler) Handler(E);

                /* Update entity material */
                E->tmat = Type->material;

                /* Call Lua hook */
                if (E->tmat != E->tmatPrev) {
                    wiz_entity_call_tmat(E, E->tmatPrev, E->tmat);
                    E->tmatPrev = E->tmat;
                }
            }
        }
    }
}
```

To be quite honest, it's a little embarrassing looking at my
old code. I'd like to think I have a cleaner style these
days.

## A Zelda-like in HaxeFlixel

Eventually life got a little too busy and I didn't have the
time for Wizwag. Fast-forward to a year or so later, when I
wanted to experiment with Haxe and HaxeFlixel. I decided to
see how it would compare to my from-scratch attempt in C,
and started rewriting.

This time, I wanted to see if I could get by with a 3rd
party map editor, and tried my go-to choice,
[Tiled.](https://thorbjorn.itch.io/tiled)

Sadly, this meant either remaking my supertiles into
ordinary ones:

![screencap of a tileset](/img/zeldascreens/ts_overworld.png)

Wizwag was always planned to have a big, visually contiguous
overworld, which scrolled screen-by-screen, befitting its
inspirations. The easiest way to accomplish this would have
been to make a giant map in Tiled, and split it into screens
programmatically, but this felt icky and disorganized.

How would I determine which objects belonged to which rooms?
What if an object's origin was _outside_ its resident room?

At the time, Tiled didn't have the "connected maps" feature,
so I made the map the size of one chunk, then made layer
groups and shifted them into place. Each room group was
named with its location in the world map grid.

![tiled screenshot](/img/zeldascreens/tiled.png)

The default `.tmx` loader in Haxeflixel couldn't accomodate
this, naturally, so I went and wrote one that could load the
basic information about all the rooms (without loading
_everything_), then provide a "finalized" room on request.

Then, I developed the lighting system explored in
[HaxeFlixel Lighting Adventure]({% post_url 2018-03-15-haxeflixel-lighting %}),
which I have been happy to see used in several other
projects by other devs. :)

Then, I needed to figure out how to do Zelda-style scrolling
screen transitions. Before long, I had written this proof of
concept, which could use some cleanup:

```haxe
//  1) move the player one tile in that direction
//  2) load the new room
//  3) [do cool camera slide]
//  4) unload old room

// first, we need to determine which room to try to load
var loadDirection = "";
if (_player.x > floor.width - 4)
    loadDirection = "r";
if (_player.x < 0)
    loadDirection = "l";
if (_player.y > floor.height - 4)
    loadDirection = "d";
if (_player.y < 0)
    loadDirection = "u";

if ("" != loadDirection)
{
    trace("moving to: " + loadDirection);

    // check if our map has a room we can go to
    var newID:String = _map.getAdjacentRoomID(_roomID, loadDirection);
    try {
        // load the new room
        _newRoom = _map.loadRoom(newID);

        // animation time
        var tweenLength = 0.5;

        if (_newRoom != null) {
            trace("-- got new room --");
            _roomID = newID;

            // set up new room
            _roomContainer.add(_newRoom);
            _player.scrollFactor.set(0,0);

            // the "floor" is the tile map itself, sans entities.
            // we move the new room's tilemap off-screen before the animation.
            var floor = _room.floor;
            switch (loadDirection) {
                case "r":
                    _newRoom.floor.x = floor.x + floor.width;
                    FlxTween.tween(_player, {x:8}, tweenLength);
                case "l":
                    _newRoom.floor.x =  floor.x - _newRoom.floor.width;
                    FlxTween.tween(_player, {x:floor.width - 8}, tweenLength);
                case "d":
                    _newRoom.floor.y = floor.y + floor.height;
                    FlxTween.tween(_player, {y:8}, tweenLength);
                case "u":
                    _newRoom.floor.y = floor.y - _newRoom.floor.height;
                    FlxTween.tween(_player, {y:floor.height - 8}, tweenLength);
            }

            // now, we need to pause input.
            Input.enabled = false;

            // if another room transition is happening for some reason, we need to stop it.
            if (null != _tween) {
                _tween.cancel();
                _tween.destroy();
            }

            // gather all the attributes we need to animate.
            _prevColor = FlxG.camera.color;

            // the animation itself
            _tween = FlxTween.color(null, tweenLength, FlxG.camera.color, _newRoom.tint,
                {onUpdate:function(t:FlxTween) {
                    // sliiide to the left, sliiide to the right
                    _lighting.color = FlxColor.interpolate(_room.tint, _newRoom.tint, t.percent);
                    _lighting.alpha = FlxMath.lerp(_room.darkness, _newRoom.darkness, t.percent);

                    var midp0 = _room.floor.getMidpoint();
                    var midp1 = _newRoom.floor.getMidpoint();
                    FlxG.camera.focusOn(new FlxPoint(
                        FlxMath.lerp(midp0.x, midp1.x, t.percent),
                        FlxMath.lerp(midp0.y, midp1.y, t.percent)
                    ));
                },
                onComplete:function(t:FlxTween) {
                    // put things back
                    _player.scrollFactor.set(1,1);

                    // set up lighting in the new room
                    _lighting.remove(_room.lights);
                    _room.lights.destroy();

                    _lighting.color = _newRoom.tint;
                    _lighting.alpha = _newRoom.darkness;

                    _roomContainer.remove(_room);
                    _room.destroy(); // greatly improved memory performance
                    _room = _newRoom;
                    _newRoom = null;

                    // finalize room, load objects and physics and stuff
                    _room.finalize();

                    // add new lights to lighting system
                    _lighting.add(_room.lights);

                    // resume
                    Input.enabled = true;
                }});
        }
    } catch (e:Dynamic) trace(Std.string(e));

} // end if ("" != loadDirection)
```

You can see the transitions in action here:

<iframe width="640" height="480" src="https://www.youtube.com/embed/xB5aWEyNH8s" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

If I return to this project eventually, I will likely create
a more general solution to this problem that I can share
properly, as I did with the lighting system.
