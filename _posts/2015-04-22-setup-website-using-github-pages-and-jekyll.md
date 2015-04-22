---
layout: post
title: "How I Set Up My Website Using GitHub Pages and Jekyll"
author: "RP"
date: "2015/04/22"
output:
  html_document:
    keep_md: true
comments: True
tags: [jekyll, github pages, website, lanyon]
---
I had been thinking of spiffying up my plain-looking WordPress-hosted blog for some time now and a few back I got to do just that. Actually I did more than that -- I ended up building a brand new website! While searching online for the how-tos I discovered [GitHub Pages](https://pages.github.com/) and [Jekyll](http://www.jekyllrb.com/). GitHub Pages allows you to host your website for free and Jekyll lets you build static websites. It costs me ~$50 per year to host my blog on Wordpress so the free hosting on GitHub sounded like a great deal. On top of it the Jekyll website generator would allow the look and feel of the blog -- the design -- to be completely under my control. And so I took on the project. The project required [git](http://git-scm.com/) which was already installed on my desktop PC.

##The Lanyon Theme
There are several website [themes](http://jekyllthemes.org/) for building a Jekyll website. I wanted a minimalistic look and started with the [Poole](http://www.getpoole.com/) theme which provides a basic template for a website. Part way through I switched to the [Lanyon](http://lanyon.getpoole.com/) theme because it emphasizes content by hiding the navigation in a side drawer. It is minimalist not only in looks but also in features, providing only a template: it does not feature, for examples, "extras" such as an _Archive_ page, or a _Comments_ facility, or social media buttons, and so on. But that can be easily addressed since adding those extras is straightforward, and there are many resources on the web that show you how to do just that.

##Website Build Steps
I built my website using these main steps:

* Started by setting up a repository using instructions on [GitHub Pages](https://pages.github.com/)
* Downloaded theme files from the [Lanyon](https://github.com/poole/lanyon) repository on GitHub by clicking the `Download Zip` button on the right-hand side. Extracted files from the downloaded zip file into the _username_.github.io directory created in the first step.
* Changed directory to _username_.github.io and in a command terminal ran these commands:
<pre>
$ cd <i>username</i>.github.io
<i>username</i>.github.io$ jekyll build
<i>username</i>.github.io$ jekyll serve
</pre>
Next, I opened up a browser and went to http://localhost:4000 to look at my website. That was it! My website was hosted locally on my computer and ready to be tweaked.

I then made design changes -- added pages, changed fonts, tweaked colors, etc -- and when I was satisfied 'pushed' all of the changes on to my repository on GitHub. Once the changes were in GitHub my official website was accessible on the Internet.

##Website Design Changes
Over the past few days I have made a number of changes to the Lanyon theme which I list below, and where available, provide the how-to links. The code for this site is available on [my GitHub repo](https://github.com/ragupappu/ragupappu.github.io/).

* Added _About_, and _Archive_ pages using [Joshua Lande's](http://joshualande.com/jekyll-github-pages-poole/) Poole post
* Added [Disqus](https://disqus.com/) and [Google Analytics](http://www.google.com/analytics/) again using Joshua Lande's post
* Added a _Tags_ page using Michael Lanyon's [post](http://blog.lanyonm.org/articles/2013/11/21/alphabetize-jekyll-page-tags-pure-liquid.html) about tags.
* Changed the landing page to show excerpts of posts instead of full posts
* Added icon links for Twitter, GitHub, etc. to the [sidebar](https://github.com/ragupappu/ragupappu.github.io/blob/master/_includes/sidebar.html). Used font-awesome CSS file from Michael Lanyon's GitHub [repo](https://github.com/lanyonm/lanyonm.github.io).
* Added tags below title of post
* Added social media 'Share' links (Twitter, Facebook, Google+) to the bottom of each post. Kanishk Kunal explains how to add Share buttons to a Jekyll blog in this [post](http://codingtips.kanishkkunal.in/share-buttons-jekyll/).
* Converted CSS stylesheets to Sass stylesheets using file structures similar to those in Michael Lanyon's GitHub [repo](https://github.com/lanyonm/lanyonm.github.io).
* Designed my favicon (the tiny icon that shows on the tab of your browser) at [Favicon and App Icon Generator](http://www.favicon-generator.org/) and replaced the original Lanyon theme favicon with the new one.
* The Lanyon theme is a fixed-width two-column design. Changed widths for @media CSS statements in main.scss to percentage values (by replacing em values) to convert the website to a liquid layout. This [article](http://maxdesign.com.au/articles/liquid/) by Max Design explains how to do liquid layouts.

Tools are available to migrate posts from Wordpress to other sites but since I had only a few blog posts I simply copied them over from Wordpress and saved them into the <i>_posts</i> folder as .md files. I embedded YAML front-matter in each file and changed the name of the files to be in the format required by Jekyll (YEAR-MONTH-DAY-title.md).

After pushing the initial set of website changes on to my GitHub repository, [ragupappu.github.io](http://ragupappu.github.io/), I linked my personal domain [ragupappu.com](https://www.ragupappu.com) to it. My domain is hosted by [Namecheap](http://www.namecheap.com) and I used David Ensinger's instructions [here](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) to link the two.

##Quick Links
* [GitHub Pages]()
* [Jekyll](http://jekyllrb.com/)
* [Jekyll Documentation](http://jekyllrb.com/docs/home/)
* The [Lanyon](http://lanyon.getpoole.com/) theme
* My GitHub [repository](http://github.com/ragupappu/ragupappu.github.io)
* [Favicon Generator](http://www.favicon-generator.org/)
* Joshua Lande's [Poole](http://joshualande.com/jekyll-github-pages-poole/) post
* Michael Lanyon's personal blog GitHub [repo](https://github.com/lanyonm/lanyonm.github.io)
