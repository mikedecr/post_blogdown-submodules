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
title: Managing {Blogdown} site content with Git submodules
subtitle: Modular opportunities with workable solutions
excerpt: Modular opportunities with workable solutions
---

This post discusses the use of Git submodules within a personal website/blog repository.
I focus the discussion on [Blogdown](https://bookdown.org/yihui/blogdown/) websites (built by Hugo but driven by R/Rmarkdown), but the theoretical lessons are more broadly applicable.

Roadmap for the post:

1. [What are submodules?](#what)
1. [Why would you want submodules in your website repository?](#why)
1. [How does a submodule manage its content separately from the parent repo?](#how)
1. [How do you add submodules to your website repo?](#how-to)
1. [How you build it on Netlify?](#netlify-setup)


## What are submodules? {#what}

Suppose you are working in one repository, such as your blog website, and there are tools or resources that you want to import from another repository.
You have a strong project-based workflow, so you want your project code to be contained within the project, but your desired tool is clearly as external dependency and shouldn't be managed by the same repository.
Git submodules allow you to clone this external repo into your website repo while keeping its versioning entirely separate.

### Why would you want to use submodules in your website repository? {#why}

There are a few areas of a blog website where some code or files are either severable from or modular-with-respect-to the content of the website.

One clear example is the website's **Hugo theme**.
Hugo is designed for the `/content/` of your website (specified in markdown files) to be more-or-less independent of the `/theme/`.
This isn't a perfect system as some themes have some special fields and capabilities that are intertwined with the content, but the idea is there.
At the very least, you want to be able to pull changes from your theme's Git repository in order to keep it up to date, and those updates should be easy to distinguish from any theme modifications you make in your site `/layouts/` folder.
At least in my experience, one interesting quirk of Blogdown is that when you initialize a new site and install a theme from its Git repository, it doesn't actually keep the theme repo's `.git` folder around, making it less straightforward to incorporate changes from the theme's remote repo.
You, the user, can install a theme _and_ keep the connection to its remote alive by installing the theme as a submodule yourself, instead of relying on convenience functions in Github.

Another example is the **site-level `.Rprofile` file** that controls [Blogdown package behavior](https://bookdown.org/yihui/blogdown/global-options.html).
Again, the Blogdown site sets this up for you, but if you think about it, the site repo doesn't need to own this file.
The file reflects your personal preferences that may be _equally applicable_ to any other website repo you create or manage---there is nothing special about any particular site repo that should give it ownership of this .Rprofile file.
Speaking to my own experience, I often experiment with new Blogdown themes by creating a new site folder altogether.
There's no reason why I should keep track of separate .Rprofiles across all these experimental repositories if my Blogdown preferences are identical for all of them.
I can instead import this .Rprofile using a Git submodule.

Or consider the **photos** you or I use to represent ourselves online.
Chances are you have a few photos that you recycle for your various online profiles: headshots or otherwise.
Again, why cart all of these files around on your computer and keep track of them independently when you can, instead, keep the master files in one place and import them elsewhere as deterministic submodules?

You may also regard **your blog posts themselves** as severable from any website repo you are working out of.
If you have ever torn your website down and started over, but had to cart your actual blog posts from one project folder to the next, you have already felt the pain that might have been avoided by saving your posts in their own, separate repository.


### How does a submodule manage its content separately? {#how}

There are a few important details to note about how this workflow comes together:

- **When you add a submodule to a repository, the repository tracks the _presence_ of the submodule, but it does not track the content.** 
    It is important for your website repo to detect the presence of the submodule; this ensures that your website repo can be cloned and recreated elsewhere with all of its dependencies in place.[^netlify-clone] 
    It is also important that your website repo be ignorant of the actual content of the submodule.
    The submodule is already being versioned by its own repo, and there's no need to duplicate that effort by tracking it in the website repo as well.
- **Changes to the submodule content can be pulled into your website repo.**
    This is standard workflow for Git. 
    If the remote repository changes, you can incorporate those changes in your local (in this case, submodule) copy of the repository by Git-pulling.
- **Changes to the submodule content can be pushed to remote.**
    If you have write access to the submodule repository (for example, because its source code is in another project on your computer), you can make changes to the submodule contents from within the submodule and push to remote.
    It's similar a multi-user Git workflow, except you're one user editing the repo from multiple endpoints.
    This lets you keep the submodule content updates on all of its local and remote copies with minimal effort.


## How to use submodules {#how-to}

In the spirit of modularity, there is actually nothing Blogdown-specific about including submodules within a project repository.
All the same, I will discuss the .Rprofile example mentioned above.
I keep my Blogdown .Rprofile in its own repository [here](git@github.com:mikedecr/dots_blogdown.git).

Add the submodule to your website repo (assuming the website repo is already initialized) with `git submodule add [...]`. 
You may want to be strategic about where you add the repo, since it will effectively behave like a cloned repository.
I prefer (lately) to give projects a `/submodules/` folder, and clone submodules there.

```sh
# from /path/to/site
mkdir submodules
cd submodules
git submodule add git@github.com:mikedecr/dots_blogdown.git
```

Adding the submodule _does not_ clone the repository contents.
It simply registers the submodule within the repository, creating an entry in your `.gitmodules` file (and creating the file altogether, if it didn't already exist).
You have to run a separate command to actually clone the submodule repo's contents:

```sh
git submodule update --init --recursive
```

The output will look like you did a `git clone`.

From there, your next step depends on how you want to use the contents of the submodule.
For me, I want to have this .Rprofile exist at the top of my project repository so it is sourced when I open an R session to control the website.
So I should link this file to the site directory.

```sh
# exit /submodules/
cd ..
# -s = symlink, -f = force
ln -f ./submodules/dots_blogdown/.Rprofile ./.Rprofile
```

You may use hard links or soft (symbolic) links---  have been hard linking on macOS because I can't get the symlink to work the way I want it to, but that's probably my own mistake.
For my workflow for my job (which uses Linux machines & Python modules), symbolic linking is easier and sufficient.

From here, if I ever change my .Rprofile repo, updating it on my website is as simply as pulling the repo (and updating your links, depending on the way you set any links).


## Getting it working on Netlify {#netlify-setup}

Once you are done getting your site looking the way you want, commit the `.gitmodules` file and any other byproducts (such as the .Rprofile file copy).

At this point, however, my site failed to build on Netlify. 
Netlify works by essentially cloning your website repository to their servers and building Hugo on their end.
This process fails if Netlify can't successfully reproduce your website repo.
Submodules can cause this failure for two reasons.
First, if you added a submodule using `ssh` instead of `https`, you need to give Netlify extra permission.
Second, if the repo is private, it won't matter if you used ssh or https, you need to give Netlify extra permission.
Both of these are fixable (and you should be using ssh anyway).

Because Netlify is a dream for ease-of-use, they post their own guidance for adding [dependencies as submodules](https://docs.netlify.com/configure-builds/repo-permissions-linking/#git-submodules).
All you need to do is add ssh keys to your repositories so Netlify can access them when building.
And Netlify makes that key generation very easy to do.

## That's really it.

Once you have added those dependencies as submodules, and given Netlify permission to access those repositories, Netlify does the rest.
You can see for yourself on this website repo that my [theme](https://github.com/mikedecr/mikedecr-site/tree/main/themes) is managed by a submodule, and my [submodules folder](https://github.com/mikedecr/mikedecr-site/tree/main/submodules) contains everything else I'm currently using.
It's all working fine.

