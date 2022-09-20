---
title: "Build RKey Tech"
date: 2022-03-02T06:31:37-07:00
draft: true
description: "Build rkey.tech project"
---
[Matching Blog Post](https://rkey.online/posts/buildopekkttech/)

{{% pageinfo %}}
This document is old and probably not accurate anyway as I initially built the site using the Congo theme and the changed to the GeekDocs theme but just modified from memory on what was different. This will be completely redone in the near future to reflect the Docsy build and deployment. So stay tuned for that. 
{{% /pageinfo %}}

Update 19-July-2022: When I first built my site I built it using the Congo theme with Hugo. I did that because I could not figure out how to build my preferred theme GeekDocs correctly. Well is was a simple Markdown error on my part so now RKey.Tech will be a GeekDocs theme and my personal only site RKey.Online will use ~~Congo~~ GeekBlog theme. 

Create new Hugo project
```shell
hugo new site rkey.tech
```
cd into new project and initialize
```shell
cd rkey.tech
git init
```
Verify and initialize 'go'
```shell
go version
  go version go1.17.7 linux/amd64
hugo mod init rkey.tech

```
I initially tried using webpack, but my luck putsing with npm on Linux is no better than the luck I have on FreeBSD. I have come to realize Java freaking hates me!!
If you want to use the theme from a cloned branch instead of a release tarball you’ll need to install webpack locally and run the build script once to create all required assets. (This did not work for me)
```shell
# install required packages from package.json
npm install

# run the build script to build required assets
npm run build

```
So I just I just downloaded the pre-release bundle 
```shell
mkdir -p themes/hugo-geekdoc/
curl -L https://github.com/thegeeklab/hugo-geekdoc/releases/latest/download/hugo-geekdoc.tar.gz | tar -xz -C themes/hugo-geekdoc/ --strip-components=1
```

From here I just copied the files and directories from the theme directry to the site directory.  
NOTE: You should only need to do this for files you are modifying from the downloaded theme. I guess the saying do as I/they say not as I do is appropiate here. 
```shell
cd themes/hugo-geekdoc/
cp -a archetypes assets data i18n images layouts static ../../
```

From there it is just a matter of configuring toml or yaml files. 

The first one to tackle is config.toml in the site root. I left this mostly as is. 
```shell
cat config.toml
baseURL = 'https://rkey.tech/'
languageCode = 'en-us'
title = 'RKey Tech'
#theme = "hugo-geekdoc"

pluralizeListTitles = false

# Geekdoc required configuration
pygmentsUseClasses = true
pygmentsCodeFences = true
disablePathToLower = true
geekdocFileTreeSortBy = "date"
geekdocSearchShowParent = true

# Required if you want to render robots.txt template
enableRobotsTXT = true

# Needed for mermaid shortcodes
[markup]
  [markup.goldmark.renderer]
    # Needed for mermaid shortcode
    unsafe = true
  [markup.tableOfContents]
    startLevel = 1
    endLevel = 9

[taxonomies]
   tag = "tags"
```

Then I added my deployment script.
```shell
cat deploy.sh
rm -rf public/
rm public.tar
HUGO_ENV="production" hugo --gc || exit 1
echo OK, now that stuff is built
rsync -azP --delete public/ caddy:/home/opekktar/rkey.tech/
echo OK, now that stuff is uploaded
echo ======================================
echo Done
echo ======================================
```

I then made the following modification to i18n/en.yaml for footer brandinig.
```yaml
footer_build_with: >
  Copyright 2022 - Robert Key
  for <img alt="RKey.tech" src="/images/rkeytechlogostny.png.webp" width="50" height="25"<a> and <a href="https://rkey.online" target="_blank"><img alt="RKey.Online" src="/images/rkeyonline.png.webp" width="60" height="60"></a>
footer_legal_notice: Legal Notice
footer_privacy_policy: Privacy Policy
footer_content_license_prefix: >
  Content licensed under
  ```

For my links in the header I modified layouts/partials/head/custom.html and added the following to this blank file.
```html
<!-- You can add custom elements to the page header here. -->
<a href="https://rkey.online" target="_blank"><img alt="RKey.Online" src="/images/rkeyonline.png.webp" width="70" height="70"></a>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<a href="https://gitlab.com/RWK3y" target="_blank"><img alt="GitLab" src="/images/icons8-gitlab-70.png.webp" width="30" height="30"></a>
<a href="https://github.com/RWK3y" target="_blank"><img alt="GitHub" src="/images/icons8-github.svg" width="30" height="30"></a>
<a href="https://bsd.network/@R0bWK3y" target="_blank"><img alt="BSDNetwork@Mastodon" src="/images/mastodon.png.webp" width="30" height="30"></a>
<a rel="me" href="https://twitter.com/RWK3y" target="_blank"><img alt="Twitter" src="/images/icons8-twitter-circled.svg" width="30" height="30"></a>
<a rel="me" href="https://facebook.com/FB.RWK3y" target="_blank"><img alt="Facebook" src="/images/icons8-facebook.svg" width="30" height="30"></a>
<a rel="me" href="https://www.instagram.com/fb.rwk3y/" target="_blank"><img alt="Instagram" src="/images/icons8-instagram.svg" width="30" height="30"></a>
<a rel="me" href="https://www.reddit.com/user/RWK3y" target="_blank"><img alt="Reddit" src="/images/icons8-reddit.svg" width="30" height="30"></a>
<a href="https://infosec.exchange/@rkey" target="_blank"><img alt="InfoSecExchange@Mastodon" src="/images/mastodon.png.webp" width="30" height="30"></a>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<a href="https://www.standwithukraine.how/" target="_blank">StandWithUkraine<img src="/images/icons8-ukraine-70.png.webp" width="30" height="30"></a>
<a href="https://protectdemocracy.org/" target="_blank">ProtectDemocracy<img src="/images/icons8-usa-70.png.webp" width="30" height="30"></a>
```

From there I just extract my favicons I created at <a href="https://cthedot.de/icongen/" target="_blank">icongen</a>
```shell
cd static/favicons
unzip icongen.zip
```

All my images for the site are kept in static/images for brand images I usually go to <a href="https://icons8.com/" target="_blank">Icons8</a>. I generally strive to download ```.svg``` files, barring that I will download ```.png``` files and then use either ```convert``` on the command line from the ***ImageMagick*** program or more recently I discovered ***libwebp*** and now use ```cwebp``` to convert images from the command line. 

The Geedocs theme turned out to be the perfect solution for my online documentation. My origional theme used Congo, which is a beautiful theme, but not suited quite as well as the Geekdocs theme for documentation purposes. I really like the layout better for even using as a <a href="https://rkey.online/" target="_blank">blog</a>.