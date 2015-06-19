---
layout: post
title: Creating a Blog using GitHub Pages and Jekyll
---

<!-- links -->
[github]: https://github.com
[github pages]: https://pages.github.com

[git immersion]: http://gitimmersion.com

[jekyll]: http://jekyllrb.com
[jekyll quickstart]: http://jekyllrb.com/docs/quickstart/
[jekyll structure]: http://jekyllrb.com/docs/structure/

[poole]: http://getpoole.com
[poole demo]: http://demo.getpoole.com

[markdown]: http://daringfireball.net/projects/markdown/

[font awesome]: http://fontawesome.io

[highlightjs]: https://highlightjs.org/
[highlightjs demo]: https://highlightjs.org/static/demo/

<!-- post -->

This blog was created using [GitHub pages] [github pages] and [Jekyll] [jekyll]. The process was pretty straight forward, but there are several components to creating a complete blog and I wanted to document all the helpful links and resources that I relied on.

## Step 0: GitHub ##

First off, you need to know some git and have a GitHub account. In short, **git** is an open source version control system and **GitHub** is an online git repository service. If you are not familiar, there are plenty of resources to help you out. Start with this [tutorial] [git immersion] and the [GitHub] [github] website.

## Step 1: GitHub Pages ##

**GitHub Pages** is a free website hosting service from GitHub. Once you have a GitHub account and know the basic git commands, follow the steps on [Github Pages] [github pages] to get your website started.

After you've completed those steps, you now have a repository on GitHub named _username.github.io_ and a website at _http:&#x2F;&#x2F;username.github.io_! All the changes you push to your repository will be instantly reflected on your website. You should now have a running website that says "Hello World".

Note that if you are creating a website for a project instead of your user account, some of the steps will be different. I'll only cover the user site process in this post, but the GitHub Pages site has very clear instructions on creating a project site.

## Step 2: Jekyll ##

**Jekyll** is a static site generator. It takes text files in a template directory and converts them into the files you need for your website. This is ideal for a blog since it's static and consists of many pages with mostly formatted text. I write my posts in text files using [Markdown] [markdown] syntax and Jekyll will convert them into proper HTML files.

To create your Jekyll directory, follow the quick-start guide on [Jekyll] [jekyll quickstart]. You should have cloned your GitHub repository onto your local computer, so navigate to that folder (which is empty aside from the index.html file that says "Hello World") and run the following:

    ~$ gem install jekyll
    ~$ jekyll new .

Now you have Jekyll installed and ready to go, but your folder still only has that index.html file!

See [here] [jekyll structure] for the basic file structure you need for Jekyll. It's a good idea to follow the documentation on the Jekyll website to create your website. You can follow the detailed documentation to set up each folder and file, but if you want to get to a minimally functional blog as soon as possible, you might prefer to jump in with a Jekyll template as I did.

## Step 3: Poole ##

**Poole** is a functioning template site built for Jekyll. Check out the [website] [poole] and the [demo] [poole demo] to see what a basic Jekyll blogging site will look like. There are also two themes based on Poole available, called _Hyde_ and _Lanyon_. There are demos for each of them, so try them out and see which one you prefer (you can completely modify the functionality and look of your site later if you so please, but that will require some knowledge in HTML/CSS). I chose to go with Lanyon for the sidebar. Head over to the respective GitHub repository for the template of your choice and download the files into your local repo.

Now you can see how all the pieces are used by Jekyll to create a website. I spent some time studying and playing around with the files in the template to get an understanding of how things worked. You can go into the folders and modify/add/remove files to transform the template into your personal blog.

You can use Jekyll's `serve` command to preview your website on a development server. Once you are happy with what you have, you can commit your changes and push them onto your GitHub repo and your updated site will be live.

## Step 4 (Optional): Custom URL ##

If you want a custom URL instead of relying on the default _username.github.io_, then follow the steps in this [post](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) to set it up. Note that in the "Set up the DNS" section of the blog post, you should type the IP addresses as shown - those are the GitHub Pages servers, as described [here](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/).

You should have a fully functional blog at this point at your own custom domain name. I hope my notes and small collection of links were helpful in that process. Once you get into the rhythm of typing up blog posts and pushing them to GitHub, you'll quickly realize the benefits of using Jekyll. Read on for more helpful tools that I used to create my blog.

## More ##

### Icons ###

[Font Awesome] [font awesome] has a collection of scalable vector icons that you can use for your blog. I used it for all the icons you see in my sidebar (at the time of this writing). You can use them by just adding this line in your html:

    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">

Simply add the proper class name to some `<i>` tags and marvel at the beauty of your website. Here is a cube <i class="fa fa-cube"></i>, using the code:

<pre><code><i class="fa fa-cube"&gt;</i&gt;</code></pre>

### Animating Sidebar Icon ###

I really wanted an animating "hamburger" button that revealed the sidebar. I found [these sample ](http://codepen.io/designcouch/pen/Atyop) hamburger icons that animated to <i class="fa fa-remove"></i>'s using only CSS (and jQuery for the mouse click event). I ended up going with a modified version of the third icon.

Here is [my animating icon](http://codepen.io/wezleytsai/pen/QbqbVa) on CodePen if you want to check it out.

### Highlighting Code ###

The content of my blog is related to software development, so I have a lot of code blocks. You can use Markdown to create `pre` and `code` tags easily, but it won't help you highlight key words for whatever language your code is in. Enter [Highlight.js] [highlightjs], which searches your html for `<pre><code>` tags, detects the language automatically, and highlights syntax for you. Add it to your blog with these lines of code (this would use the "default" theme):

    <link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.6/styles/default.min.css">
    <script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.6/highlight.min.js"></script>
    <script>hljs.initHighlightingOnLoad();</script>

There are tons of themes to choose from - check out the [live demo] [highlightjs demo]. I'm using the theme _Hybrid_.

## Resources ##

- [GitHub Pages] [github pages]
- [Jekyll] [jekyll]
- [Poole] [poole]
- [Markdown] [markdown]
- [Font Awesome] [font awesome]
- [Highlight.js] [highlightjs]
- Here's a great [post](http://joshualande.com/jekyll-github-pages-poole/) with a walkthrough of setting up a blog with GitHub pages, jekyll, poole, and more.
