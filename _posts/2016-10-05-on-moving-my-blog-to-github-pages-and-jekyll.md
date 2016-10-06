---
title: On moving my blog to GitHub Pages and Jekyll
date: 2016-10-05 22:39
categories: [Techical-Howto]
tags: [Github, Jekyll]
---

So I have migrated my blog from Wordpress.com to Github Pages and Jekyll. More on that in [the previous post](http://blog.itaysk.com/2016/10/04/i-have-moved-my-blog-to-github-pages-and-jekyll).

In this post I will document the journy from a technical perspective.

## Starting off with Jekyll
As you know I am not a Ruby guy and never has been. Moreover, "I am a PC" and Ruby is not so great on Windows.
Luckily, Windows 10 is actually also Ubuntu nowadays! It's not a VM, it's actually Ubuntu ABI compatibility. It's called WSL (Windows subsystem for Linux), and it's awesome.

So I have used bash on Windows to: 

- install [RVM (ruby version manager)](http://rvm.io/)
- use RVM to get the latest Ruby
- use RVM to create a gemset dedicated to Jekyll and blog stuff.
- install Jekyll, create a new blog and play with it

In every step I looked for the Linux instructions and followed them blindly. It usually just works which is really amazing. 2 caveat2 I found was:

- when you install RVM it adds some stuff to your bash profile settings. This works only of you are logged in to bash which is not happening by default. This is a very common error with Ubuntu (not Windows specific) and there are [many different suggestions](http://stackoverflow.com/questions/12258343/rvm-is-not-a-function) how to fix it. What I like to do it just `bash --login` in the beginning of my session which is easy and always works.
- when you run Jekyll with `Jekyll serve` it won't work on bash on Windows because it's by default watching the file system for changes and that's not yet working with WSL. It is coming though. As a workaround I run `Jekyll serve --no-watch` which worked great for me. There is a fix inbound for this, [see here for reference](https://github.com/Microsoft/BashOnWindows/issues/216). 
- I used VS Code to develop the files that were created in WSL. That works great if your working directory is in a NTFS drive like  `C:\` and not so great if you are working off of the Ubuntu file system. [See here for reference
](https://github.com/Microsoft/BashOnWindows/issues/552).

Once you have everything setup, and you read your way through [jekyllrb.com](http://jekyllrb.com), you get the hang of the workflow and start writing some posts. Since this is an existing blog, I had to migrate content from Wordpress.com to Jekyll.

## Wordpress.com Migration

Migrating the content from Wordpress.com was harder that I thought it would be. There are a several tools out there, including an [official Jekyll importer](http://import.jekyllrb.com/docs/wordpressdotcom/), but I ended up using [wpXml2Jekyll](https://github.com/theaob/wpXml2Jekyll), **plus a lot of manual works**.
There was quite a lot of manual editing, fixing inline html, standardizing FrontMatter headers, and so on. I didn't mind the manual work because I also wanted to review my old posts but you should be prepared that there will be some tedious tasks.

I did used some PowerShell scripting to help with all that work. I was joyfully reminded of how proficient PowerShell is, I just love this tool. I am definitely not a PowerShell expert, and it's not a tool that I use often, but yet it was so easy and fun to use. What I like best about PowerShell is the discoverability and consistency of commands. You can always sort of guess your way through, or auto complete and hack something that works pretty quickly, without even looking at docs. Also, since you are piping object and not text, you are gathering useful techniques that you can apply everywhere. 
I used PowerShell to do some bulk tasks like download all images from the old blog, rename images based on a new naming convention, clean up generated posts, parse the Wordpress exported xml, and more...

One thing I decided to change during the migration was images. In Wordpress I hand picked names for images 
as I uploaded them to the Media Library. This time have decided to stick to a convention where image names follow this pattern: `concat(post_date,'-',url_safe(post_title),'_',index)`. For example: `2015-08-03-azure-load-balancer-in-resource-manager-arm_7.png`. Again - a combination of PowerShell scripting and manual editing.

Other than the posts, I also migrated the comments. Wordpress.com has it's integrated commenting system and in the new blog I used [Disqus](http://disqus.com). I exported the comments via the wp admin portal, this gives you an xml with all the content, including posts, images and whatnot. 
Disquss can import that, but one issue I had was that WP exports post urls ending with a slash so this post's title would have been "bla-bla/moving-my-blog-to-github-pages-and-jekyll/". The tailing slash didn't play nice with disqus so I had to scan the xml before importing it and removing this extra slash.

## Jekyll customization

I wanted to preserve URLs of existing posts from the old blog, I have them in tweets and URL shorteners. Jekyll's default URL pattern in not compatible with Wordpress.com's because it adds the category to the beginning of the path. I didn't want that so I changed it.
Basically Jekyll's documentation will tell you to define a permalink in the FrontMatter section in your post, but I didn't want to manually craft a slug for every post I write so I added this as a default  configuration in the `_config.yml` so that it is inherited to all posts.

As for the design of the blog - I spent some time looking for a theme. What I looked for is a simple and clean design, and a dominant author information section in the post layout (I believe this professional blog is a tool to develop my career, and I think that someone who is reading a post should at least know who wrote it).
I found these themes which I liked:

- [Hyde](https://github.com/poole/hyde)
- [Lanyon](https://github.com/poole/lanyon)
- [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes)

I spent some time studying them and testing how my blog will look with them. I also tried customizing them with minor adjustments that I wanted which wasn't very easy. This was when I realized that I should add bootstrap css as a requirement from the theme I would use because it is super easy to use and tweak. 
Eventually I ended up creating my own theme since I knew exactly what I wanted, and it was simple enough to create.
You can [check out this blog on github](https://github.com/itaysk/itaysk.github.io) for a look behind the scenes of the theme but I will also package it into a gem and publish it on RubyGems.

## What's missing

- GitHub Pages still doesn't support SSL for custom domains. I can live with that now but as the industry is moving to SSL everywhere future I might need at alternative such as a CDN for SSL termination. [More on this here](https://github.com/isaacs/github/issues/156).
- Jekyll still hasn't made their mind on what are the boundaries between a theme and a site and some features are missing in themes that prevents me to refactor all the design out of my blog into a standalone theme. [More on this here](https://github.com/jekyll/jekyll/issues/5341).
