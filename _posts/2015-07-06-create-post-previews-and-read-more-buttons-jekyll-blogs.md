---
layout: post
title: Create Post Previews and Read More Buttons for Jekyll Blogs
---

<!-- links -->
[poole]: http://getpoole.com

<!-- post -->
I used a handy Jekyll template called [Poole][poole] when I originally set up my blog. It's great because it creates all the pages you need to get a Jekyll site running, but it also masks the underlying machinery that is Jekyll. For example, Poole, by default, shows the full contents of each post on the main page (the index.html). That means if your post is a lengthy one, the reader might not even notice that there are multiple posts on that same page!

Fortunately, there's a very simple solution to this by defining the `excerpt_separator` property in your config.yml file.

<!--excerpt-->

Somewhere in your index.html file, you might have code that looks similar to this, which says "for each post, create a post title, add the post date, and add the post contents":

{% raw %}
```html
{% for post in paginator.posts %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date: '%b %-d, %Y' }}</span>

    {{ post.content }}
  </div>
{% endfor %}
```
{% endraw %}

As you might know, the {% raw %}`{{ post.content }}`{% endraw %} variable represents the entire post. There is actually another useful variable available in Jekyll called {% raw %}`{{ post.excerpt }}`{% endraw %}, that stores the portion of your post, starting from the beginning of the post to wherever the `excerpt_separator` is encountered in the post (or end of the post if `excerpt_separator` is not in your post).

I set my `excerpt_separator` property to `<!--excerpt-->` in my config.yml by adding this line:

```yaml
excerpt_separator: "<!--excerpt-->"
```

Stick that string where you want the preview of your post to end:

```
---
layout: post
title: Title of My Blog Post
---

Here is a sentence that will show up in the preview.

And another sentence that will also show up in the preview.

<!--excerpt-->

Reader has to click into the post to see this sentence.
```

And in your index.html file, you would modify it as follows:

{% raw %}
```html
{% for post in paginator.posts %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date: '%b %-d, %Y' }}</span>

    {{ post.excerpt }}
    {% if post.content contains site.excerpt_separator %}
      <a href="{{ site.baseurl }}{{ post.url }}">Read more</a>
    {% endif %}
  </div>
{% endfor %}
```
{% endraw %}

The last three lines after {% raw %}`{{ post.excerpt }}`{% endraw %} simply check if the post contains whatever string you have designated as the `excerpt_separator`, and if so, adds a "Read more" button. If not, the entire post will be visible on the home page and there is no need for a "Read more". The user can still click on the post title to reach the individual post, just like any other post.