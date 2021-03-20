---
title: "Using a proportional font for markdown"
type: post
date: 2021-03-20T12:00:00+08:00
---

Two weeks ago, I switched to using a proportional font for editing markdown text. I've been using vscode's inbuilt markdown previewer a lot. Continually staring at it's proportionally spaced output probably triggered a flash of inspiration.

I figured markdown in a more readable font would be a nicer experience. And since I've started blogging, I've been writing a lot of prose in markdown so it makes sense to give it a try.

It took me a day or two to get used to it. But now, I like it, and I'm staying with it. Occasionally, the proportional spacing gets in the way, but overall, it's been a great net positive. I encourage everyone to try it.

Fortunately for me, it's easy to do this in vscode so it affects markdown files only. You want something like this in your `settings.json` file:

```json
    "[markdown]": {
        "editor.fontFamily": "-apple-system, BlinkMacSystemFont, 'Segoe WPC', 'Segoe UI', system-ui, 'Ubuntu', 'Droid Sans', sans-serif",
    },
```

where `fontFamily` is a list of fonts. I just copied the `fontFamily` value from the inbuilt markdown previewer.

Note that I use a proportional font for markdown only. I think this is a clear proposition.

I'm still using monospace for other stuff. I think proportional fonts for coding is not so clear cut. If you are interested in it, here's some discussions from the past:

* [Does anyone prefer proportional fonts?](https://softwareengineering.stackexchange.com/questions/5473)
* [Programming with proportional fonts is great](https://news.ycombinator.com/item?id=1056683) ([web archive](https://web.archive.org/web/20130401074244/http://m-mz.posterous.com/programming-with-proportional))
