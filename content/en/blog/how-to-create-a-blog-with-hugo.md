---
title: 'How to Create a Blog With Hugo'
date: 2024-03-13T21:13:55+02:00
---

## Prerequisites

#### Install Hugo

On Debian, we can install Hugo from the official repository:

```
$ sudo apt-get install hugo
```

## Setup

### Create a Site

Create the directory structure for your blog in the `ilanco.dev` directory.
```
$ hugo new site ilanco.dev
```

Change into the new directory `ilanco.dev` and initialize a new git repository.
```
$ cd ilanco.dev
$ git init
```

### Add a Theme

We're going to add the [`hugo-blog-awesome`](https://github.com/hugo-sid/hugo-blog-awesome) theme to our blog as a module.
```
$ hugo mod init github.com/USER/REPO
$ hugo mod get github.com/hugo-sid/hugo-blog-awesome
```

Replace USER/REPO with your GitHub username and repository name where this project will live.

For Hugo to use the theme, add the following to the main configuration `hugo.toml`:
```
[module]
  [[module.imports]]
    path = "github.com/hugo-sid/hugo-blog-awesome"
```

### Run the Hugo Development Server

```
$ hugo server -D -F
```
> _The `-D` argument tells Hugo to include drafts, while the `-F` argument instructs Hugo to include future dated posts that are published_

Next, open your browser and navigate to [http://localhost:1313](http://localhost:1313).
