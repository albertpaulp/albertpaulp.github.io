---
layout: post
title:  "Setting up a static blog with Github Pages"
date:   2018-08-10 14:07:23 +0530
categories: tech
---

I wanted to start blogging recently, it's been sometime since I created a blog so I've done some research to
get started. There is a plethora of ready to use blogging platforms available and older ones like Blogger and Wordpress
are never been more easier before.

In the end, I decided to go with **static blog** created by
**[Jekyll](https://jekyllrb.com/){:target="_blank"}** and hosted on
**[Github Pages](https://pages.github.com/){:target="_blank"}**.

### Why going back to bare metal static html ?

* Zero maintenance !! Write, push/publish and forget.
* No worries about server management/security.
* Zero bucks required if hosted with Github, save that monthly $10 server bill for beer :beers:.
* Performance is blended in by nature, forget about query optimisation/pruning/indexing or whatever.
* Inherently simple architecture, everything is just HTML and some markup that's it.
* Plenty of plugins for features requiring a backend, like using [Disqus](https://disqus.com/){:target="_blank"} for user
  interaction/comments.

I still need some kind of server to handle web traffic and serve HTML right ? Now that's where
[Github Pages](https://pages.github.com/){:target="_blank"}🥁 comes in, you just have to provide static HTML/Markup
files Github will host that free of cost, and deals with traffic management(Unlimited bankwidth), redirection and HTTPS.
All hail :octocat:.

### How to create one for yourself (Get it up in 25 minutes or less !! )

1. Create a Github repository with `your-username.github.io` and create directory named `your-username` in your local.
  * Add Github [remote](https://help.github.com/articles/adding-a-remote/){:target="_blank"} to your local repo.
2. Setting up blog structure,
  * You can either use handcrafted html files or use a static CMS like
    [Jekyll](https://jekyllrb.com/){:target="_blank"}(recommended) to manage your blog.
  * If you prefer to create your own HTML with CSS/JS then add/commit those files to repository and push to `master`
    branch.
  * If you are opting for Jekyll, install Jekyll with their
    [installation doc](https://jekyllrb.com/docs/installation/){:target="_blank"}, then
    [build your blog](https://jekyllrb.com/docs/usage/){:target="_blank"}, commit and push it to master branch of
    `your-username.github.io` repository we created.
  * Go to settings page of `your-username.github.io` repo, it should say something like _"Your site is published at
    `your-username.github.io`"_. You can try that link in your browser and make sure site is up and running.
3. Adding a custom domain,
  * I've bought mine from Bigrock.com, you can use any domain registrar like AWS Route 53 etc.
  * We can connect our custom domain to GitHub hosted page using 2 mechanisms(there are other options as well, but sake
    of simplicity), either using _A records_ or _CNAME_.
  * I would prefer using _CNAME_ mechanism due to various advantages over _A Records_.
  * In order to add _CNAME_ to your new domain, find Domain management console or get help from registrar for the right
    place to add _CNAME_.
  * I use `www.albertpaulp.com` subdomain as entrypoint to my blog, if you want similar configuration,
    Add _CNAME_ with host as `your-domain.com` and value as `your-username.github.io`, also add `www.your-domain.com`
    (notice I've added `www`) and value as `your-username.github.io`. In a nutshell, you just told your domain registrar
    where to send traffic if somebody typed in your custom domain `your-domain.com`.
  * Now create a file named _CNAME_(full caps without any extension) in your repo and add `www.your-domain.com` as
    plain text content, this is for Github to match your local page with custom domain.
    See [CNAME](https://github.com/albertpaulp/albertpaulp.github.io/blob/master/CNAME){:target="_blank"} of this blog.
  * That's it, done. Now wait for DNS Propagation, it can take up to a day max, usually this is done in 1 - 2 hours.
4. Adding a free SSL certificate with [Let's Encrypt](https://letsencrypt.org/){:target="_blank"} !
  * If you ever had to manually setup a _Let's Encrypt_ certificate on your server, you won't believe how easy
    to set up HTTPS for your custom domain here, it's as simple as clicking ***"Enforce HTTPS"*** button in your
    repository settings.

_Once everything is done, it'll look something like this,_

  ![custom domain github pages](/assets/images/github_page_blog.png)

Enjoy your new shiny, slick and secure blog !! :collision:

![Alt Text](https://media.giphy.com/media/3otPomFXNTRLvrhedq/giphy.gif)
