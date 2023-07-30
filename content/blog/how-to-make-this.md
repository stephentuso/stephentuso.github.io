---
title: "How to make this"
date: 2023-03-02T23:49:21-05:00
draft: true
tags:
  - Web
  - DevOps
  - Tutorial
---

As is tradition, the first post must explain how the blog
was set up.

## What and Why

I wanted a static site for speed, simplicity, and cost. After searching around I settled on [Hugo](https://gohugo.io/) with the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. This
is the first time I've used Hugo and so far it seems nice, but I don't like how the content can be very coupled to some themes. That might be necessary to get something unique out of a generic
static site generator, but it makes it hard to change themes later if you want to. I don't really need something unique though, I just want to write posts in markdown and have them show up on the internet. I chose PaperMod because even though it does have some theme-specific features, I don't have to use those, and it's a simple, clean base. I wanted to make some small tweaks, so I went ahead and forked it.

## How

### Setup

Install hugo - on MacOS it's just:

```sh
brew install hugo
```

Let's set up a new site and add the theme.

```sh
hugo new site my-site
cd my-site
git init
git submodule add --depth=1 https://github.com/stephentuso/hugo-PaperMod.git themes/PaperMod
```
> Use the SSH url for the theme if you have your own fork, so that you can push your changes

There used to be a flag on `hugo new site` to choose config format, but it seems to have been taken out. I prefer YAML over the default TOML, so delete `hugo.toml` and add the following in a new file `hugo.yaml`, with the appropriate values:

```yaml
baseURL: https://my-site.com/
languageCode: en-us
title: My Site
theme: PaperMod
```

### Github Actions



### Workflow

Now you can start the dev server:

```sh
hugo serve -D
```

`-D` is to include draft posts, which is what we're going to add now:

```sh
hugo new content blog/first-post.md
```

This will add a markdown page at `content/blog/first-post.md` with some frontmatter like this:

```;
---
title: "First Post"
date: 2023-07-30T14:25:24-04:00
draft: true
---
```

Now you can write your post beneath that, and remove `draft: true` when you're ready to publish it. My only gripe with that is that there doesn't seem to be a built in way to update the date to match the publish time instead of when you created the file. There is a variable `publishDate` that you can add, it just isn't automated.


