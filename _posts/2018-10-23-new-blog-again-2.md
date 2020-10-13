---
title: "A New Blog Again... Again!"
date: "Tue, 23 Oct, 2018"
layout: post
tags: meta MoonScript
---

Hahahaaaaaaa yeah, seriously. When I was first working on [PQ93](http:/pennie.itch.io/pq93), I was researching scripting languages. I like Lua, but I've used it _far_ too often and I wanted to try something else. Then I discovered [MoonScript](http://moonscript.org).

As a friend later remarked: _MoonScript is bae._

Since making MoonScript the official language for PQ93 cartridges, I've really come to like it. It has the pros of Lua without the cons, and a simple, sexy syntax, too.

Then I discovered that [Leafo](http://leafo.net) has also written a [site generator](http://leafo.net/sitegen) in MoonScript. _mmf_

So I went ahead and set that up and used it to rewrite my blog! Looking at old code of `leafo.net` for usage examples (scant docs!), I ended up trying [SASS](http://sass-lang.com) as well, which is _also_ bae.

Being a lethargic, depressive person (most of the time), I realized I probably wouldn’t update the blog, even with the rewrite, if I had to maintain the feed file and copy/paste metadata to new posts. So I began writing a `bash` script to create new posts. This is what I had done for previous iterations of the blog--use `sed` and a template file and blahblahblah.

Then I realized it would be simpler and better to just write _that_ in MoonScript, too. It took a few minutes and works fine:

```MoonScript
args={...}

if #args < 2
	print'usage:\n\tmoon post.moon my-post "My Dope Post"'
	return -1

feed = "feed_posts.moon"

is_md = (s) -> #s>3 and s\sub(#s-3,#s) == '.md'
no_md = (s) -> s\sub(1,#s-3)

title = args[2]
infile = "posts/" .. args[1]\lower!
infile ..= '.md' unless is_md infile

header = [[
{
	title: "%s"
	date: "%s"
	layout: post
}

]]\format title, os.date "%a, %b %d, %Y"

entry = [[
	{
		date:  date %s
		title: "%s"
		link:  "https://pennie-quinn.github.io/posts/%s.html"
		description: ""
	}
]]\format os.date("%Y, %m, %d"), title, no_md infile

-- check if file already exists
overwrite = false
with file=io.open infile, "r"
	if file
		\close!
		print "#{infile} already exists! Do you want to overwrite it? (y/N)"
		with r = io.read! do return unless (r == 'y' or r == 'Y')
		overwrite = true

-- write the file
with file=io.open infile, "w"
	\write header
	\close!

-- add the post to feed_posts.moon
if not overwrite
	local lines
	with io.open feed, "r"
		lines = [l for l in \lines!]
		\close!
	with io.open feed, "w+"
		for i=1,#lines-1 do \write lines[i],"\n"
		\write entry
		\write lines[#lines]
		\close!

-- open the post in Focused
os.execute "open -a Focused \"$(pwd)/#{infile}\""
```

So, now, I can just say:
```Bash
$ moon post.moon foo “Bar”
```

I _did_ go ahead and wrap it in a `bash` script, so I can say `./post foo “Bar”`, but that’s just excessive laziness. ;)

New langs, new look, easier maintenance + posting. Nice!