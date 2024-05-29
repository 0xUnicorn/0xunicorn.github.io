---
layout: post
title:  "How to setup GitHub Pages"
date:   2021-08-29 01:11:49 +0200
author: 0xUnicorn
tags: Jekyll GitHub
---

![GitHub Pages](../assets/img/github-pages.png){:.post-cover-image}

When you need a place to express your thoughts, feelings or technical doings a blog is a nice solution.

If you want a "drag/drop" solution, there's a lot of solutions out there now a days, but if you want a quick site with some simple markdown language, [GitHub Pages](https://pages.github.com) is a really good solution.

Setting up GitHub Pages is very fast and easy. If you want a more customized layout, it can take some time to get everything to fit nicely though.

<!--excerpt-->

# Simple page

If you just want a simple page with no or very little customization to the layout. Follow the few very simple steps on the [GitHub Pages site](https://pages.github.com)

# Customized page

For getting a more customized page, the easiest solution is to create a local development environment with Jekyll. 

## Prepare GitHub

On GitHub you need to create a public repository with the name `<username>.github.io`. After you created a new empty repository, clone it to your local machine.
Go to your new repository and select `Settings > Pages`. Enable GitHub Pages for your repository.

## Install Jekyll

Before we can start the devlopment environment, we need to install the needed prerequisites for Jekyll to work.

```bash
# Install prerequisites
sudo apt-get install ruby-full build-essential zlib1g-dev

# RubyGems needs to be installed under the user account instead of root.
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Install Jekyll and Bundler with gem:
gem install jekyll bundler
```

## Create site

Go into the newly created repository which you cloned to your machine when you prepared GitHub.

```bash
# Make sure to get the period added for initializing your site in your current folder
jekyll new .
```

For getting your dev environment up and running, you need to change the content of your `Gemfile` with the following:

```ruby
source "https://rubygems.org"

# To update to the latest github dependencies run: `bundle update`
# To list current versions: `bundle exec github-pages versions`
# Check github versions: https://pages.github.com/versions/
gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem 'jekyll-feed'
  gem 'jekyll-paginate'
  gem 'jekyll-seo-tag'
  gem 'jekyll-sitemap'
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?
```

Before you can start the local server for serving you new site, you need to install all of the `gem` needed by running:

```bash
bundle update
```

## Run site

For starting up your new site, you need to run:

```bash
bundle exec jekyll serve
```

Now you will be able to access your page on http://localhost:4000

## Upload site to GitHub

When you have a site you like, you can simple do a commit and push to GitHub and your site will go through a pipeline before deploying.

```bash
git add .
git commit -m "New site"
git push
```


