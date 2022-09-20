---
title: Setup GitLab Repo
date: 2022-03-03T12:17:04.000Z
draft: true
description: Setup GitLab Repo
---

[Matching Blog Post](https://rkey.online/posts/setuprepo/)

Set global variable first.

```shell
git config --global user.name "Robert Key"
git config --global user.email "rkey@rkey.tech"
```

Then I create a blank project on GitLab. I do not initialize the repo, as that would cause issue when uploading this project for the first time. With the empty repo created I then initialize and upload my project.

```shell
git init --initial-branch=main
git remote add origin git@gitlab.com:rkey/rkey.tech.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

I have an install script I do not want in the repo and I also do not want the public directory in the repo either. So my .gitignore file looks like this:

```shell
bsh ➜  cat .gitignore
public
.hugo_build.lock
deploy.sh
public.tar
```
