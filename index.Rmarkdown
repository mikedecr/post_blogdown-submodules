---
author: Michael DeCrescenzo
categories: 
date: "2021-10-26"
draft: false
layout: single
tags:
- computing
- blogdown
- rstats
- git
- netlify
title: |
  Managing {Blogdown} content and dependencies with Git submodules
subtitle: Modular opportunities with workable solutions
excerpt: Modular opportunities with workable solutions
---

Most introductions to [`blogdown`](https://pkgs.rstudio.com/blogdown/) guide the reader to create a website managed by a single Git repository.
This repository manages everything in the website: site & theme configs, page content, theme customization, blog posts, build routines and more.
These guides are extremely helpful for users who are new either to blogdown or to Git.

But the single-repository workflow may present certain drawbacks for users who have more experience with these tools.
If you value **modularity** as a guiding principle in your software design (as I do), you may reject the idea that a website is just one project worthy of one repository.
You may instead view the website as a combination of modular components that are better managed independently with separate repositories.
These modular components _come together_ at the nexus of the website, but the components do not _belong to_ the website.

This post advances a modular view of blog sites and discusses how to use [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to manage the many separate components of the site.
I focus the discussion on [Blogdown](https://bookdown.org/yihui/blogdown/) websites (built by Hugo but driven by R/Rmarkdown), but the theoretical lessons are more broadly applicable.

Road-map for the post:

1. [What are submodules?](#what)
1. [Why would you use submodules for your website code?](#why)
1. [How does a submodule manage its content?](#how)
1. [How do you add submodules to your website workflow?](#how-to)
1. [How do you get it working on Netlify?](#netlify-setup)


## What are submodules? {#what}

Submodules are like repositories-within-repositories. 

Suppose you are working in one repository (your blog website), and there are external tools or resources that you want to import from another repository.
You have a strong project-based workflow, so you want all of the code that creates your website to be contained within the website directory on your computer.
At the same time, the external dependency is clearly its own entity, and there is no reason why its code should be owned by the website repository.
Git submodules allow you to clone this dependency repo into your website directory. 
Your website repository is _aware_ of the external dependency, so your project remains reproducible in its entirety, but the website directory does not redundantly track the _content_ of the dependency code.


### Why would you want to use submodules in your website repository? {#why}

There are several features of a blog website where the code or files are either agnostic to the content of this particular website or, in some cases, completely severable from the website. 
For my own workflow, I consider my blog posts, Hugo theme, and blogdown build settings (in my site-level `.Rprofile`) as modular or separable components from the website as a whole, and each are managed with submodules.
A quick note on each:

- **Blog posts**: The content of a blog post is completely separable from the website repo. 
    We can take a blog post and locate it in a different website, and it would still be meaningful.
    Many blogdown users remake their website and carry their old blog posts to the new site, which encapsulates the posts' relationship to the site as a whole.
    I discuss this modular approach to blog posts in more detail [here](/blog/post_submodule).

[^theme]: This isn't a perfect system; some themes define special fields whose values are specified in your content files, but the main idea is there.

- **Hugo theme**: Hugo is designed such that the `/content/` of a website (specified in markdown files) is more-or-less independent of its `/theme/`.[^theme]
    The same theme can be used for multiple websites, and a single website can swap out one theme for another.
    Because themes are managed with Git repositories already, you can pull theme updates without losing any extraneous customizations specified in your `/layouts/` folder.
    When you create a new website with blogdown, the package actually interrupts this workflow by deleting your chosen theme's `.git` directory, but if you install your theme as a submodule, you can use the theme _and_ maintain its connection to its remote repository.

- **The website `.Rprofile` file**: You may have a global .Rprofile file, but the purpose of the website-specific .Rprofile is to control [blogdown build behavior](https://bookdown.org/yihui/blogdown/global-options.html).
    Your blogdown build preferences probably are nto specific to this website repository.
    Instead, it is likely that those preferences reflect your workflow for blogging _in general_ and could be equally applicable to any other website repo you create or manage.
    If you change your blogdown workflow in a way that bears on this .Rprofile file, that change will likely affect all of your blogdown websites equally, so managing those .Rprofiles separately for each website is inefficient and error-prone. 



### How does a submodule manage its content separately? {#how}

If you have never heard of submodules before, the following details are helpful for understanding how they can fit into a blogdown workflow.
Disclaimer: this is not an exhaustive rundown of how to use submodule.

**When you add a submodule to a repository, the repository tracks the _presence_ of the submodule, but it does not track the content.** 
Your website repo tracks the presence of submodules to ensure that your repo can be cloned with all necessary dependencies in place.[^netlify-clone] 
However, your website repo is ignorant of the actual content of the submodule because the submodule code is versioned by its own separate repo.
There is no need to duplicate that effort.

[^netlify-clone]: This is also crucial for Netlify to build your site, in fact, because Netlify clones your repository in and rebuilds your website from the clone.

**Changes to the submodule repo can be pulled into your website repo.**
This is standard workflow for Git. 
If you want to pin your dependency to a particular commit of the submodule, simply check out that submodule.
If you want your dependency to stay dynamically up to date with the submodule's remote repo, checkout the desired remote branch and pull changes as they arise.

**Changes to the submodule content can be pushed to remote.**
If you have write access to the submodule repository (for example, because its source code is in another project on your computer), you can make changes to the submodule contents from within the submodule and push to remote.[^detached-head]
This is similar to a multi-user Git workflow, except you are one user editing the repo from potentially several endpoints.
This lets you keep the submodule content updated on all of its local copies and remotes with minimal effort.

[^detached-head]: Just be sure you have checked out a branch (not in detached `HEAD` state) before you make changes to the submodule files. (More [here](https://git-scm.com/book/en/v2/Git-Tools-Submodules#_working_on_a_submodule)).



## Using submodules: the absolute basics {#how-to}

In the spirit of modularity, there is actually nothing Blogdown-specific about including submodules within a project repository.
All the same, I will discuss the .Rprofile example mentioned above.
I keep my Blogdown .Rprofile in its own repository [here](git@github.com:mikedecr/dots_blogdown.git).

Add the submodule to your website repo (assuming the website repo is already initialized) with `git submodule add [url] [destinateion]`. 
You may want to be strategic about where you add the repo, since it will effectively behave like a cloned repository.
I prefer (lately) to give projects a `/submodules/` folder, and clone submodules there.

```sh
# from /path/to/site
mkdir submodules
cd submodules
git submodule add git@github.com:mikedecr/dots_blogdown.git
```

Adding the submodule _does not_ clone its contents.
It simply registers the submodule within the repository, creating an entry in your `.gitmodules` file (and creating the file altogether, if it didn't already exist).
You have to run a separate command to actually clone the submodule repo's contents:

```sh
git submodule update --init --recursive
```

The output will look like you did a `git clone`.

From there, your next step depends on how you want to use the contents of the submodule.
For me, I want to have this .Rprofile exist at the top of my project repository so it is sourced when I open an R session to control the website.
So I link this file to the site directory.

```sh
# exit /submodules/
cd ..
# -s = symlink, -f = force
ln -f ./submodules/dots_blogdown/.Rprofile ./.Rprofile
```

It is smart to automate any post-Git processes, such as linking files to other destinations, by specifying those operations in your website's `/R/build.R` and `/R/build2.R` files.
This ensures that your website builds in a robust and replicable way if your submodule content should ever change.
With that automation in place, if I ever change my .Rprofile repo, synchronizing that file in my website repo is as simple as pulling the submodule changes and rebuilding the website.


## Getting it working on Netlify {#netlify-setup}

Once you are done getting your site looking the way you want, commit the `.gitmodules` file and any other byproducts (such as the .Rprofile file copy).

At this point, however, your site may fail to build on Netlify. 
Why? 
Netlify works by cloning your website repository to their servers and building it with Hugo on their end.
This process fails if Netlify can't successfully reproduce your website repo with all of the submodules declared in your `.gitmodules` file.
This can happen if the submodule is a private repository or was added using the repo's `ssh` URL instead of the `https` URL.
Both of these causes are 100% fixable by specifying ssh-keys that grant Netlify permission to access those repositories. 
Netlify makes these keys easy to generate, and they describe it all [right here](https://docs.netlify.com/configure-builds/repo-permissions-linking/#git-submodules).

Once you add these dependencies as submodules and give Netlify permission to access them, Netlify does the rest.
