---
layout: post
title:  How I improved my Jekyll Build time using jekyll-include-cache
tags: jekyll, cache
---
My recipe blog [Food I Made](https://foodimade.com) is kind of my playground
for all my random static site generation ideas as well as a place for me to
  share my recipes and food photography.

One thing that bothered me recently is that as I was adding more content to my
site builder and adding more interdependent links, the build time on my jekyll
pages was creeping up to about 30 seconds.

```sh
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 35.816 seconds.
```

One method I read about was to use a plugin called
[jekyll-include-cache](https://github.com/benbalter/jekyll-include-cache).
These are the steps that I took to use this plugin on my blog. 

# 1. Verify that this plugin is usable on github pages

The first thing I wanted to check before I invested any resources in this is if
my hosting provider for jekyll, Github Pages, would support this plugin. You
don't want to go down too far of a rabbit hole investigating this locally if it
won't even work on the hosting provider. It takes a little digging, but you can
find all the supported gems for github pages
[here](https://github.com/github/pages-gem/blob/master/lib/github-pages/plugins.rb).
They keep this list pretty trim for security reasons, but they are whitelisting
new plugins all the time as they become popular. As you can see from this
history, this is an older plugin and they actually whitelisted it about [2
years
ago](https://github.com/github/pages-gem/commit/7a2e6db9b27a97807cde94ce2db8b9b8b6113569),
which is a great sign that this is a widely used plugin that isn't going
anywhere.

# 2. Update my build process

To get started then, I update my local repository in the following ways.

To my site's `Gemfile`, I add
```gem 'jekyll-include-cache'```

To my site's `_config.yml` file I update the `plugins:` section to include ```
plugins:
  - jekyll-include-cache
```

# 3. Identify and update all includes

I then tried to find all files in my jekyll directory that use the `include`
directive. Just to make sure I'm not overmatching I'm excluding directories I
know I don't need to modify, such as my generated directory `_site`. I'm also
making sure to include the space after `include` in my grep pattern so when I
do replace these with `include_cached` I can keep rerunning the exact same
  command and not picking up the lines that I've already fixed.

```sh
{% raw  %}
   find . -type f -not -path "_site/" | xargs grep '{% include '
{% endraw %}
```

# 4. Make sure that all includes don't reference page or local variables

One note you should make sure you do when you switch to `include_cached` is
that you have to make sure your include templates don't reference any page or
local variables. You can pass variables to your templates and reference them
within the template as `include.name`.

# 5. Verify improvements!

By making this switch, and updating a few files, I immediately saw huge
performance improvements generating my static jekyll site. My build time is
down from over 30 seconds to just about 3 seconds. Even better, if I only
modify one file that doesn't have too many dependencies, the generation time to
update can be under 1 second if I have the incremental flag on.

```sh
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 3.039 seconds.
```
