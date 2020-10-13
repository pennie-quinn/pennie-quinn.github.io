---
title: "Let's Write a 3D Game in JS"
date: "Thu, Aug 15, 2019"
layout: post
---

## Introduction

Recently, I recieved some career advice and resume help, and
the person helping me said that it would improve my game dev
resume to have at least one 3D project on it.

This sounds like good advice, especially given how niche,
meaning ultra low-fi, my 2D projects have been.

I was suggested to use Unity, given its ubiquity, but I soon
will be doing most work remotely on an iPad between classes,
and won't have hours to spend in Unity at my desk.

Lately, I have been experimenting with game dev with JS.
Coming from Native Programming Land, it has been interesting
(and at times irritating).

So, why not both make my first 3D game _and_ learn some JS
game dev, together?

## Tools & Environment

I recently set up a [simple environment][1] for writing
games in [CoffeeScript][2], with [Phaser 3][3]. I did so
because I don't like using massive build systems or curating
labyrinthine dependency trees. :)

Since I already have that, the next question is: _Which of
the seemingly many WebGL libraries and game frameworks for
JS do I try out?_

[1]: https://github.com/pennie-quinn/phaser-template
[2]: https://coffeescript.org
[3]: https://phaser.io

After checking most of the entries on [this list][4], I
decided [three.js][5] looks the best (and the most recently
worked on). It provides no integrated physics engine or
modeling capabilities, leaving the fun stuff for me to
figure out.

[4]: https://github.com/collections/javascript-game-engines
[5]: https://github.com/mrdoob/three.js

## Setting Up

Let's grab our files:

```fish
cd ~/projects
git clone https://github.com/pennie-quinn/phaser-template.git 3dgame
cd 3dgame
wget -O src/three.min.js http://threejs.org/build/three.min.js
```

And let's paste the three.js Hello World code into our
`src/app.coffee`, in an inline js block:

```coffeescript
import './three.min.js'

`
var camera, scene, renderer;
var geometry, material, mesh;

init();
animate();

function init() {
	camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 10 );
	camera.position.z = 1;

	scene = new THREE.Scene();

	geometry = new THREE.BoxGeometry( 0.2, 0.2, 0.2 );
	material = new THREE.MeshNormalMaterial();

	mesh = new THREE.Mesh( geometry, material );
	scene.add( mesh );

	renderer = new THREE.WebGLRenderer( { antialias: true } );
	renderer.setSize( window.innerWidth, window.innerHeight );
	document.body.appendChild( renderer.domElement );
}

function animate() {
	requestAnimationFrame( animate );

	mesh.rotation.x += 0.01;
	mesh.rotation.y += 0.02;

	renderer.render( scene, camera );
}
`
```

Ok, that works.

![hello world screenshot](/img/3d/hello.png)

Now let's convert that to CoffeeScript. First we need to put
the calls to `init` and `animate` at the _end_ of the file,
since the functions won't be hoisted.

Then, we mostly need to pour lots of syntactic sugar, with
consideration given to how variables and scoping work in
CoffeeScript :)

```coffeescript
state = {}

init = ->
    state.camera = new THREE.PerspectiveCamera 70, window.innerWidth / window.innerHeight, 0.01, 10
    state.camera.position.z = 1

    state.scene = new THREE.Scene()

    geometry = new THREE.BoxGeometry 0.2, 0.2, 0.2
    material = new THREE.MeshNormalMaterial()

    state.mesh = new THREE.Mesh geometry, material
    state.scene.add state.mesh

    state.renderer = new THREE.WebGLRenderer  { antialias: true }
    state.renderer.setSize window.innerWidth, window.innerHeight

    document.body.appendChild state.renderer.domElement

animate = ->
    requestAnimationFrame animate

    state.mesh.rotation.x += 0.01
    state.mesh.rotation.y += 0.02

    state.renderer.render state.scene, state.camera


init()
animate()
```

And let's go ahead and OOP it up a bit, for cleanliness:

```coffeescript
class Game
    constructor: ->
        @camera = new THREE.PerspectiveCamera 70, window.innerWidth / window.innerHeight, 0.01, 10
        @camera.position.z = 1

        @scene = new THREE.Scene()

        geometry = new THREE.BoxGeometry 0.2, 0.2, 0.2
        material = new THREE.MeshNormalMaterial()

        @mesh = new THREE.Mesh geometry, material
        @scene.add @mesh

        @renderer = new THREE.WebGLRenderer  { antialias: true }
        @renderer.setSize window.innerWidth, window.innerHeight

        document.body.appendChild @renderer.domElement

    animate: ->
        @mesh.rotation.x += 0.01
        @mesh.rotation.y += 0.02
        @renderer.render @scene, @camera


game = new Game()

update = ->
    requestAnimationFrame update
    game.animate()

update()
```

We could handle our variables in a more data-driven,
separated way, but I want to experiment with classes for
now. If it becomes a problem when we want to do live reloads
or something, we can fix it. :)

## Getting Started

First, let's try loading a model!

```fish
wget -O src/OBJLoader.js https://raw.githubusercontent.com/mrdoob/three.js/master/examples/jsm/loaders/OBJLoader.js
wget -O src/GLTFLoader.js https://raw.githubusercontent.com/mrdoob/three.js/master/examples/jsm/loaders/GLTFLoader.js
mkdir assets
```

I can't get either of these loaders to work with given
examples. A console dump reveals that the models are, in
fact, being loaded. I tried scaling loaded models by a large
factor, just in case, but that yielded nothing as well.
