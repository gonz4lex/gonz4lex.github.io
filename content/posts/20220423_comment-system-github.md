title: A comment system for static websites using Github Discussions
date: 2022-4-23 13:31
category: web dev
tags: python
slug: github-comment-system
author: Alex Gonzalez
summary: Easily add comments to your static website with giscus and GitHub Discussions! 
status: published


I've been searching for an easy alternative to a comment system for this [blog](https://www.alexgonzalezc.dev), which is built with [Pelican](https://blog.getpelican.com/) and served through Github Pages. Being a static website, traditional server-based solutions are not a good fit or they require too much setup work that I really didn't want to do. 
So after many months procrastinating on this, I finally sat down and got it work in less than an hour. Easy!

## Setting up

I looked mainly at two solutions, [utterances](https://utteranc.es) (based on Github issues) and [giscus](https://giscus.app) which is still under active development since it's based on a newer feature, Github Discussions. So naturally, I chose the more unstable and experimental solution.

The setup is extremely straight-forward and can be found either on the app website or the [repo](https://github.com/giscus/giscus), so make sure to check those sources as well.

Start by heading to https://giscus.app to begin the process.

## Make your website _giscus-ready_

Your website's repo needs to be on Github, for obvious reasons, and needs:

- __public__ access, so discussions can be viewed
- the [giscus app](https://github.com/apps/giscus) __installed__
- the __Discussions__ feature [turned on](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/enabling-or-disabling-github-discussions-for-a-repository)

You can simply follow the guidelines in the website to generate you definitive `script` tag which just needs to be plugged into your HTML template, and it looks like this:

```html
<script src="https://giscus.app/client.js"
        data-repo="[ENTER REPO HERE]"
        data-repo-id="[ENTER REPO ID HERE]"
        data-category="[ENTER CATEGORY NAME HERE]"
        data-category-id="[ENTER CATEGORY ID HERE]"
        data-mapping="pathname"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
```

## Plug-and-play

The only necessary step left is to go into your desired template (`article.html` in my case) and just paste that snippet of code wherever I want my comments to show up:

```html
<!-- article.html -->
  ...
  <div class="article_text">
    {{ article.content }}
  </div>
  <div class="article_meta">
      ...
  </div>

  {% if GISCUS_ENABLED %}
  <div id="article_comments">
    <div id="comment_thread">
    </div>
    <script src="https://giscus.app/client.js" data-repo="gonz4lex/gonz4lex.github.io"
    data-repo-id="MDEwOlJlcG9zaXRvcnkyMjU5MTg3MDM=" data-category="Announcements"
    data-category-id="DIC_kwDODXc-784COr2Q" data-mapping="pathname" data-reactions-enabled="1" data-emit-metadata="0"
    data-input-position="bottom" data-theme="light" data-lang="en" data-loading="lazy" crossorigin="anonymous" async>
      </script>
  </div>
  {% endif %}
```

I added an option `GISCUS_ENABLED` flag on my settings file, but that's entirely optional.

After that, just reload and regenerate your site, and comments should show up on your posts!

## Some closing thoughts 

Even if this isn't a big change or project, it's another reminder of why it's almost always better to dive right away into demos or proofs of concepts rather than getting stuck on an endless loop of _analysis paralysis_. If I hadn't waited so long because I wasn't sure how to _best_ tackle this feature, I would had probably finished months ago.

However, before implementing this feature yourself, make sure it works by trying out the comment section below!


