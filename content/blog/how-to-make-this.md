---
title: "How to make this site"
summary: Guide for creating a blog with Hugo and GitHub Pages
slug: hugo-github-pages-blog-tutorial
aliases:
  - /blog/how-to-make-this
date: 2023-07-30
tags:
  - Web
  - DevOps
  - Tutorial
---

As is tradition, the first post must explain how the blog was set up.

## What and Why

I wanted a static site for speed, simplicity, and cost. After searching around I
settled on [Hugo](https://gohugo.io/) with the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. This is the
first time I've used Hugo and so far it seems nice, but I don't like how the
content can be very coupled to some themes. That might be necessary to get
something unique out of a generic static site generator, but it makes it hard to
change themes later. I don't really need something unique though, I just want to
write posts in markdown and have them show up on the internet. I chose PaperMod
because even though it does have some theme-specific features, I don't have to
use those, and it's a simple, clean base. I wanted to make some small tweaks, so
I went ahead and [forked it](https://github.com/stephentuso/hugo-PaperMod).

## How

> You should be familiar with git and the command line to follow this

### Setup repo

Install hugo - on MacOS it's just:

```txt
brew install hugo
```

Let's set up a new site and add the theme.

```txt
hugo new site my-site
cd my-site
git init
git submodule add --depth=1 https://github.com/stephentuso/hugo-PaperMod.git themes/PaperMod
```

> Use the SSH url for the theme if you have your own fork, so that you can push
> your changes

There used to be a flag on `hugo new site` to choose config format, but it seems
to have been taken out. I prefer YAML over the default TOML, so delete
`hugo.toml` and add the following in a new file `config.yml`, with the
appropriate values:

```yaml
baseURL: https://<yourusername>.github.io/
languageCode: en-us
title: My Site
theme: PaperMod
```

### Deploy

We'll be using GitHub Actions to deploy the site to GitHub Pages. I'm using
Pages because it's free for public repos, and Actions because it integrates with
Pages easily (and is also free). Make a repo on your GitHub called
`<yourusername>.github.io`, and add it as a remote to your local repo.

Now for the Actions config, add the following to `.github/workflows/hugo.yml`:

```yaml
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.115.4
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run:
          "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci ||
          true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

That was just the suggested workflow from GitHub, I only had to update the Hugo
version, which you may need to do as well. It's somewhat self explanatory, but
it just installs Hugo and its dependencies, builds the site, and pushes to
Pages.

### Content

Now we're ready to actually work on the site. Start the dev server and open your
browser to [http://localhost:1313](http://localhost:1313)

```sh
hugo serve -D
```

`-D` is to include draft posts, which is what we're going to add now:

```sh
hugo new content blog/first-post.md
```

This will add a markdown page at `content/blog/first-post.md` with some
frontmatter like this:

```txt
---
title: "First Post"
date: 2023-07-30T14:25:24-04:00
draft: true
---
```

Now you can write your post beneath that, and remove `draft: true` when you're
ready to publish it. My only gripe with that is that there doesn't seem to be a
built in way to update the date to match the publish time instead of when you
created the file. There is a variable `publishDate` that you can add, it just
isn't automated.

The full list of variables are
[here](https://gohugo.io/content-management/front-matter/#front-matter-variables),
but some useful ones are:

- `tags`: Array of tags that will show up at the bottom, and let viewers find
  related posts
- `slug` and `url`: Change the path of a post, useful if you want the post URL
  to differ from the title.
  [More here](https://gohugo.io/content-management/urls)
- `aliases`: If you update a page URL, you can add the old URL as an alias to
  redirect to the new one
- `publishDate`: If this is set to the future, the post won't be published until
  then

Once you're ready to publish, just `git commit` and `git push` to the `main`
branch and the site will be deployed by the Action we set up earlier! You'll be
able to view it at https://\<username\>.github.io

If you have questions about anything, the code
[is here](https://github.com/stephentuso/stephentuso.github.io), or leave a
comment below!
