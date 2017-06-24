---
title: Jekyll and AWS, creating a static site at scale
layout: post
categories: intermediate
---

Using Jekyll, Amazon s3, and Amazon CloudFront to host a static site is quick, easy, and can scale at low cost.

## Jekyll
I won't go into to much detail here, the folks that created Jekyll do a really good job explaining it [here.](https://jekyllrb.com/docs/home/)
But I do feel that I should point out a few things that were pain points for me.  
1. You will want to either install a new theme or move the default one to customize it. The default theme's  location can be found with `bundle show <theme_name>`. Copy the contents of that folder into your jekyll site directory and you will be able to edit it from there. You can find the name of the theme you are using in `_config.yml`.
2. The default config has an error in it with ruby 2.4.0 there is a `gems` section in `_config.yml`  under the comment `# Build settings` this should be `plugins`. It will keep bundler from complaining about things.
3. Plugins to add (I'll break these down in a moment): 
* add these to the Gemfile under `group :jekyll_plugins do` with `gem "<plugin-name>"`
* jekyll-assets (makes it easier for static files to be cached)
* jekyll-archives (helps you filter by category and tag)
* jekyll-compose (helper for generating pages, posts, and drafts, and for publishing and unpublishing posts)
* jekyll-admin (if you want a cms style editor that you run locally to create site content)

### Escaping liquid template tags
I'm adding this here because I ran into it writing this post. To escape liquid template tags you'll have to put `{{'{{"'}} "}}` around the opening tag and the content of the tag before the closing tag like this:  
`{{'{{"{% if statement'}}"}} %}`  
which will print this:  
`{{"{% if statement"}} %}`  
Replace the double-quotes `"` with single-quotes `'` if the tag you are escaping has double-quotes in it.

### [jekyll-assets](https://github.com/jekyll/jekyll-assets) 
I like to use hashes in my static files such as css, javascript, and images to avoid having problems with caching.  Using hashes will allow those files to be cached aggressively without hitting problems with cache invalidation.  But there isn't really a good way to have jekyll do that out-of-the-box.  

Enter the plugin jekyll-assets. It will build all assets in the `_assets` folder and add the hash to the file-name. It's kinda nice. 
If you are using a theme with sass you will need to move the sass|scss files to `_assets/css` (in the default theme they are stored in `_sass`). That's it. You can modify away and the changes will get picked up by the plugin and they will be compiled and placed in the appropriate location in `_site` whe you build or serve the site.

One thing to note is with this you'll want to make sure you set `JEKYLL_ENV=production` when you build or you won't get the hash in the files. 

### [jekyll-archives](https://github.com/jekyll/jekyll-archives) 
This plugin is awesome and will automatically generate archives for you. It reads the `categories` and `tags` meta-data from posts and pages and creates page groups for each. 
Using the default theme did create a small problem wuth using this, the links at the top of the page. Those are automatically generated from the pages it sees. That will inculde the pages for the groups that jekyll-archives uses. Personally I wanted to keep the page links for when I create pages but I didn't want the archive pages, filtering ended up being fairly simple. 

In `_inculdes/header.html` there is a section under `{{"{% if page_paths"}} %}` that looks like this: 
```html
{{"{% for path in page_paths"}} %}
    {{'{% assign my_page = site.pages | where: "path", path | first '}} %}
    {{"{% if my_page.title"}} %}
    <a class="page-link" href="{{'{{ my_page.url | relative_url'}} }}">{{'{{ my_page.title | escape'}} }}</a>
    {{"{% endif"}} %}
{{"{% endfor"}} %}
```
You'll want to do something like this: 
```html
{{"{% for path in page_paths"}} %}
    {{'{% assign my_page = site.pages | where: "path", path | first '}} %}
    {{'{% if my_page.type != "category" and my_page.type != "tag"'}} %}
        {{"{% if my_page.title"}} %}
        <a class="page-link" href="{{'{{ my_page.url | relative_url'}} }}">{{'{{ my_page.title | escape'}} }}</a>
        {{"{% endif"}} %}
    {{"{% endif" }} %}
{{"{% endfor"}} %}
```
Then add categories and tags pages with the same `{{"{%for path in page_paths"}}%}` and the page type check replaced with `{{'{% if my_page.type == "category"'}} %}` and `{{'{% if my_page.type == "tag"'}} %}` respectively to list the archive pages for those. 

### [jekyll-compose](https://github.com/jekyll/jekyll-compose) 
This one is really self-explainatory. It allows you to create drafts, posts and pages. And to publish and unpublish posts from the command line without manually moving files around.  
Useful if you want to create drafts before you upload them as jekyll-admin doesn't currently support this. 

### [jekyll-admin](https://github.com/jekyll/jekyll-admin) 
This adds a CMS style gui for working with a jekyll site. I'm writing this blog post from the admin it creates right now.  
The admin only runs locally (or remotely if you work on your site from a server) and doesn't add anything extra in the site that is generated with `jekyll build`.  
Very useful if you want to easily work on the site from a tablet instead of a desktop or laptop.

## AWS 
I chose AWS s3 and CloudFront to host the static site instead of something like Heroku or a VPS because of the scaling options it provides.  
The cost is pretty low (less than a few dollars when things are slow) and it should be able to handle heavy load if a blog post catches a lot of attention from Reddit or HackerNews.  

While I don't expect either of these to happen (especially not anytime soon) more than one blogger (and company) has earned infamy from having a post get picked up by some other news feed or blog site only to have their servers fail under the load of so many people hitting it at once. By letting Amazon handle that part my infamy will come from my terrible css and horrible customized page layouts...

## Publishing
Publishing the site is pretty easy with [s3_website](https://github.com/laurilehmijoki/s3_website).  
It can set up s3 and CloudFront for you or use existing ones so I won't cover very much here except one or two things that might get you if you are doing the set-up manually.  

### s3 
If you do the set-up manually instead of using s3_website you'll want to make sure that the `index document` is set to `index.html` in `s3-bucket->Properties->Static Website Hosting`.  
That way the index.html for the main page and the various sub-folders will work as expected.  

### CloudFront  
Again s3_website will do the set-up for you but if you want to do it manually make sure you set the `Default Root Object` to `index.html` and when connecting to s3 copy and paste the url of the s3 bucket and don't select it from the list.  
If you select the s3 bucket you will get odd behavior from pages other than the main `index.html` and will have to directly link to the html.

## Closing 
I hope this can be of use for someone (or myself in the future) as a reference for setting up a Jekyll site with AWS s3 and CloudFront. 

Happy blogging