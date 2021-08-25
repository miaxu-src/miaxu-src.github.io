---
category: jekyll
title: Running Jekyll in Docker
---

I recently need to migrate my blog system from Mac to Windows.
Not willing to suffer all the hassles of setting up everything from scratch,
so I came up with a one step solution of using Docker.

Here is how:
`docker container run --name miaxu-src -p 80:4000 -v <path>:/site bretfisher/jekyll-serve`

A few quick notes about the above command:

1. The `-v` parameter specifies the path to the directory of your blogs.
If you are using a Linux and you are already in that directory,
then you can use `-v $(pwd)` for convenience.

2. The `-p` parameter specifies the published ports on the host and the docker container.
You can type "http://localhost" in your host's browser to check your post in real time.

<strong>Reference</strong>

https://github.com/BretFisher/jekyll-serve
