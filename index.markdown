---
author: Michael DeCrescenzo
layout: single
categories: 
tags:
- computing
- blogdown
- rstats
- git
- netlify
- modularity
title: |
    Highly modular blogging with Blogdown
subtitle: |
    That's it. Every blog post gets its own repository.
excerpt: |
    A blogdown website is more like a combination of modular components that are better managed independently, each component with its own repository.
    These modular components _come together_ at the nexus of the website, but the components do not _belong to_ the website.
date: "2021-11-06"
draft: false
---


When I finished graduate school, I tore down my website. 

For a handful of reasons.
I no longer needed a website that cried out, "Help me, I'm finishing my PhD and I need to escape."
I didn't need to showcase unpublished papers, teaching resources, or old blog posts that I had grown detached from or embarrassed by.
It was time for a clean reset.

But if you work with Blogdown, you know that starting over is laborious.
Not that Blogdown isn't great, because it is.
It's that, when you're a finicky person like me, setting up a website with the right balance of capable features, pleasant aesthetics, and a clean, principled codebase is legitimately challenging.

**For example, the site's Hugo theme.**
I would take a lot of Hugo themes for test-drives. 
Hugo advertizes themes as if they were completely modular with respect to your `/content/` folder.
For most themes, this is a lie.
Themes usually want too many bespoke variables or file structures in your `content` files.
Some amount of this is okay, but it comes at a cost.
If you really want to take a theme for a spin, it's almost always more expedient to create an entirely new `blogdown::new_site()` than it is to specify a different theme in your `config.toml`.

**Except now you're dragging the same files around your computer all over again.**
Ugh, this new website directory needs your blog source files, your website-level `.Rprofile` that controls Blogdown's build behaviors, the jpg/png image files that you use to brand yourself online.
And maybe these files need to go in different folders or be given different file names from the previous theme.
After a while, these files no longer have a single authoritative "home" on your computer, and you may have multiple conflicting(!) versions of these files across your various experimental website folders.

**And then there's reproducibility.**
Even after lugging around all the same files to the new site, good luck getting your blog posts to render if your package library has changed since they were written, which it probably has.
Danielle Navarro [wrote about reproducibility in Rmarkdown blogging](https://blog.djnavarro.net/posts/2021-09-30_on-blogging-reproducibly/), and argues convincingly that the best way to protect your `.Rmarkdown` posts against package deprecation is to create a _dedicated package library for each separate blog post_ using `renv`.
This sounds intense at first, but the underlying principle is simple, which makes it a good solution to a difficult problem.

This post will continue the pattern of intense-sounding-but-principled solutions for difficult problems.

## What this post is about

What we want is a principled and robust approach for managing the many interlocking components of your website.
Specifically, we explore the **_modularity_ of the elements in your website.**
I take the view that your website is a collection of modular components that are better managed independently, **each component with its own source repository.**
Yes, running your website from multiple repositories. 
Stick with me.

The modules that compose your website include your theme, your blog posts, your Blogdown build preferences (implemented in your `.Rprofile`), and maybe more.
These modular components _come together_ at the nexus of the website, but as I will argue, these components should not _belong to_ the website.
This is because they can be re-used across different websites or substituted with other similar components.
Hugo already flirts with this idea in its core design by separating the `/theme/` directory from the `/content/` directory, as if to say, "these components can be combined but do not depend on one another."
This post takes an opinionated stance that **modularity is a good idea** and should be assertively extended to other components of your Blogdown site.
I make no assertion that this stance is objectively correct---only that it has been useful enough for me that I wanted to share it.

Modularity as a software philosophy is one thing, but implementing it in code requires technical solutions.
This post will discuss how to achieve this using **Git submodules,** an intermediate-level Git construct that, if you're like me, is somewhat familiar but somewhat intimidating.
In short, a Git submodule is a repository-within-a-repository.
It has its own version history that is distinct from the "parent" repository.
In this post, I provide a simple tour of submodules and how they can be used to structure your website workflow.
We will set up a website as our "primary" repository, and we import other modular site components (like our blog posts) as Git submodules.
In case you host your blog on Netlify, I will also discuss how to ensure that Netlify can build your site successfully.

### Before we begin, some terminology

This discussion will involve plenty of concepts that sound similar but should be read as distinct.
I want to flag these concepts so that we understand each other better.

**Directory vs. repository.** 
A directory is a folder on your computer that holds files.
A (Git) repository tracks changes to files.
In many projects, a directory is entirely managed by one repository, so the distinction between the two may be papered over.
When Git submodules are involved, this is no longer be true.
Your website directory will be managed by one repository, and sub-directories below your website will be managed by other repositories.

**Website vs. module.**
The _website_ is the entire project that puts your website online.
Your website will contain various _modules_ that combine to build the entire project.
Your blog posts will be considered a module (or several modules, depending on your implementation).
Your theme is another module.
Think of modules as building blocks for your website that can be stacked, swapped out, and so on.

**Parent repository (for the website) vs. child repository (for the module), aka "submodule".**
The website and the module will be versioned by separate repositories.
We can refer to the over-arching project (the website) as the "parent" repo and the module as the "child" repo.
A "Git submodule" is a Git construct that is overlaid onto this relationship.
A repo, in isolation, is just a repo.
But if you import a repo into another project as a dependency (like importing an R package into an analysis), this dependency is considered a "submodule" to the parent repository, and this affects our Git workflow as a result.
I explain all of that below.


## Websites are a collection of modules

Modules are like little building blocks, and your website has plenty of them.
Setting aside any formal mathematics of what constitutes a "module", we can crudely recognize them as structures in your website that are agnostic to the content of other structures.
We may even be able to replace modules with other modules or remove them entirely without affecting the core function of other modules.

Some examples.
For my own workflow, I consider my blog posts, Hugo theme, and blogdown build settings (in my site-level `.Rprofile`) as modular or separable components from the website as a whole, and I version each component with its own separate repository.
A quick note on each:

- **Blog posts**: The content of a blog post is completely separable from the website repo.
    We can take a blog post and locate it in a different website, and the blog post should still be meaningful (and reproducible) unto itself.
    Many blogdown users remake their websites and carry their old blog posts to the new sites, which shows that the blog content doesn't functionally depend on the website.

    It turns out that, for blog posts, modularity and reproducibility are pretty closely related.
    In her discussion of blog reproducibility, Danielle Navarro touched on the principle that a blog should be ["encapsulated" or "isolated"](https://blog.djnavarro.net/posts/2021-09-30_on-blogging-reproducibly/) to robustify the blog against website dependencies.
    By insisting that blogs also be modular, not only is the blog protected from the website's computational environment, we can control each post independently of one another, move posts around across contexts, and remove posts entirely without side-effects.
    
    This also affects how we treat the blog post's dependencies.
    Suppose that your post includes an analysis on a data file that you read from disk.
    This file should belong to your blog post (and be versioned by that blog post's Git repository), not your website.
    This means you should keep all of these files in the blog post directory, and forget about the website's `/static/` folder except for files that rightfully belong _to the website_.


- **Hugo theme**: Hugo is designed such that the `/content/` of a website (specified in markdown files) is more-or-less[^theme] independent of its `/theme/`.
    The same theme can be used for multiple websites, and a single website can (in theory) swap out one theme for another.
    Because themes are managed with Git repositories already, you can pull theme updates from their remote repositories without overwriting any bespoke theme customizations specified in your `/layouts/` folder.

    Blogdown complicates this somewhat.
    When you install a theme with `blogdown::new_site()`, Blogdown actually deletes your theme's `.git` directory.
    (At least, this was my experience.)
    This is probably for ease-of-use among users who will not find it desirable to manage the site theme as a submodule.
    But we are enthusiastic seekers of modularity, so we want to keep that upstream remote connection alive.

[^theme]: The system isn't perfect. 
Some themes define special fields whose values are specified in your content files, but the main idea is there.

- **The website `.Rprofile` file**: You may have a global .Rprofile file, but it is an increasingly common Blogdown workflow staple to set up a website-specific .Rprofile to control [Blogdown's build behavior](https://bookdown.org/yihui/blogdown/global-options.html).
    How is this a module?
    Your blogdown build preferences are probably not specific to this website repository.
    Instead, it is likely that your preferences reflect your workflow for blogging _in general_ and could be equally applicable to any other website repo you create or manage.
    If you change your blogdown workflow in a way that bears on this .Rprofile file, that change may affect all of your blogdown websites equally!
    Managing these .Rprofiles separately for each website would be inefficient and error-prone, so instead we manage the .Rprofile in one repository that we import to our website as a submodule.


## Git submodules

Git submodules are repositories-within-repositories.
Suppose you are working on a project repository (like your website), and there are external tools or resources that you want to import from another project.
You have a strong project-based workflow, so you want all of the code that creates your website to be _contained within the website directory_ on your computer.
At the same time, the external dependency is clearly its own entity, and there is no reason why its code should be owned by the website repository.
Git submodules allow you to clone this dependency repo into your website directory so you can use this code without versioning it redundantly.

### Submodule basics {#basics}

If you have never worked with submodules before, here is how they work in broad strokes (read: **not exhaustive**).

**When you add a submodule to a repository, the repository tracks the _presence_ of the submodule, but it does not track the content.** 
Your website repo tracks the presence of submodules to ensure that your repo can be cloned with all necessary dependencies in place.[^netlify-clone] 
However, your website repo is ignorant of the actual content of the submodule because the submodule code is versioned by its own separate repo.
There is no need to duplicate that effort.

[^netlify-clone]: This is how Netlify builds your site, in fact. 
Netlify clones your website's Git repository and rebuilds on their servers with their own copy of Hugo.

**Upstream changes to the submodule repo can be pulled into your website repo.**
This is standard workflow for Git. 
If you want to pin your dependency to a particular commit of the submodule, simply `git checkout` that commit.
If you want your dependency to stay dynamically up to date with the submodule's remote repo, checkout the desired branch and pull changes as they arise.

**Local changes to the submodule content can be pushed to remote.**
If you have write access to the submodule remote (for example, because its source code is in another project on your computer), you can make changes to the submodule contents _from within the submodule_ and push to remote.[^detached-head]
This is just like Git workflow where multiple users are pushing to the same remote repository, except instead of multiple users, it's only you, editing the repo and committing/pushing changes from different endpoints.
This allows you to keep the submodule content updated on all of its local and remote copies without duplicating any effort.

[^detached-head]: Just be sure you have checked out a branch (not in detached `HEAD` state) before you commit changes to the submodule files. More [here](https://git-scm.com/book/en/v2/Git-Tools-Submodules#_working_on_a_submodule).


### Adding website components as submodules

In the spirit of modularity, there is actually nothing Blogdown-specific about including submodules within a project repository.
All the same, I will discuss the .Rprofile example mentioned above.[^blog-example]
I keep my Blogdown .Rprofile in its own repository [here](git@github.com:mikedecr/dots_blogdown.git).

[^blog-example]: I discuss how I manage _blog posts_ with submodules in a later section.

Add a submodule to your website repo (assuming the website repo is already initialized) with `git submodule add [url] [destination-folder]`. 
You may want to be strategic about where you add the repo, since it will effectively behave like a cloned repository.
I often give projects a `/submodules/` folder, and clone submodules there.

```sh
# from /path/to/site
mkdir submodules
cd submodules
git submodule add git@github.com:mikedecr/dots_blogdown.git
```

Adding the submodule _does not_ clone its contents.
It simply registers the submodule with the repository, creating an entry in your `.gitmodules` file (and creating the file altogether, if it didn't already exist).
You have to run a separate command to actually clone the submodule repo's contents:

```sh
git submodule update --init --recursive
```

The output will look like you did a `git clone`.

From there, your next step depends on how you want to use the contents of the submodule.
For a module like this, 
For me, I want to have this .Rprofile exist at the top of my project repository so it is sourced when I open an R session to control the website.
So I link this file to the site directory (and remove write permissions[^write]).

[^write]: I do this in order to prevent myself from editing a copy of a file instead of the "source" file.
Because I forcefully link the .Rprofile file from the submodule to the website root, any changes I make to the copy at the root will be overwritten if I ever re-link the file.
If I try to edit the file after removing write permissions, my computer will tell me that I am opening a read-only file, which will remind me to edit the .Rprofile in the submodule repo instead!
Just a little trick to guard against bugs :)

```sh
# exit /submodules/
cd ..
# -s = symlink, -f = force
ln -f ./submodules/dots_blogdown/.Rprofile ./.Rprofile
# bonus: remove write-permissions (make read-only)
chmod -w ./.Rprofile
```

It is smart to automate any post-Git processes, such as linking files to other destinations, by putting these commands and other pre-build operations in your website's `/R/build.R` file.
This ensures that these operations are done each time your website is built, ensuring that your website can be safely reproduced if your submodule content should ever change.
With that automation in place, if I ever changed my .Rprofile repo, I never have to worry about re-linking my updates to the right place.
I just pull the submodule changes and rebuild the website---copying the file from the submodule to the project root happens automatically.


### Developing within the submodule repo

The above instructions describe how to simply employ submodule files in your website.
But suppose you wanted to change the content of the submodule files and push those changes back upstream.
What would you do?
This isn't too hard.

**Before making any changes to the submodule files, make sure the submodule isn't in detached HEAD state.**
A detached HEAD state is basically what happens when you have checked out a commit in isolation of the branch on which that commit is located.
As a result, any files you change cannot be committed to a persistent branch.
To make permanent changes, you should checkout the branch that you want the submodule to track, which is probably `main`.

**Make your changes.**
Even though you are editing a file within a submodule repository, Blogdown doesn't know or care, so it shouldn't behave any differently.
It will knit/render blog posts and serve your website locally like nothing is wrong.
That's because nothing _is_ wrong.

**Commit changes to submdodule files to the submodule repository.**
From the command line, this means you probably should `cd` into the submodule repo before adding any files to the index.
If you use a Git GUI, you should be able to make the submodule appear as its own repo that you can do add/commit/push actions to.
After committing to the local submodule repo, you should notice that your parent repository detects an updated commit in the submodule!
You should commit that change to the parent repository as well.
This simply tells the parent repo that it depends on this new submodule commit, not the older one.
This is important because anyone else who clones your website repository (like Netlify!) will need to import the submodule at the correct commit.

**Both submodule and parent repos can be pushed.**
Although if this is your first time pushing submodule commits to Netlify, you may want to read the [section about Netlify below](#netlify-setup).

As you get more familiar with Git, you won't need to follow a checklist.
You will simply be familiar enough with how Git works to know exactly what to do!


## What about the blog?

Should your blog be one submodule repository, or several?
My current setup is to treat every blog post as its own, separate module with its own, separate repository.
This keeps each post and all of its dependencies isolated from other posts, which is cleanest for me from a reproducibility and modularity standpoint.

However, you may find many blog post repositories to be overkill, and would instead want a single repository containing all of your blog posts.
Would that be fine?

In short, the single-blog-module setup may be possible, but you will have to do some extra Git magic to make it work within your typical website structure.
If you really want to know the technical details, you can read about the [problem](#problem) and one [potential solution](#fix).
If you trust me that this is pretty complicated except for people who want to push their Git chops, you can skip ahead to read about [separate repositories for each post](#every-post). 


### One submodule for all posts: the problem {#problem}

To explain, consider the [submodule workflow](#basics) mentioned earlier.
If we wanted to use a "single submodule" approach to blogging, we would

- move our blog posts to another repository
- add this repository as a submodule located in `content/blog` or whatever you call your blog folder.
- The `content/blog` folder is now effectively owned by the submodule repository. The parent repo won't know what's going on in those files---only if you have make new commits.

Unfortunately, this may be a critical problem for your website repo.
Check your `content/blog` folder right now. 
Are there other files in there aside from the posts?
Perhaps an `_index.md` file that manages the content of your blog's "listings" page?
Perhaps that index file also links to a sidebar image in the same directory?
If we let our blog submodule own this folder, those files can no longer be tracked by the parent repository (the website repo), even if add the files to the submodule `.gitignore`.
Or, at least, _I_ was not successful in making the submodule "disown" those files for the parent repo to rediscover them.

### One submodule for all posts: there might be a way {#fix}

I haven't tested this, but there might be a way to version all of your blog posts in a single submodule repository: you could make your blog post repository a **bare repository**.
A bare repository is a repository with no root directory.
If you have only used Git on a per-project basis, the idea of a repo with no root directory sounds unthinkable, but it is actually a common way to [version your user profile dotfiles](https://dev.to/bowmanjd/store-home-directory-config-files-dotfiles-in-git-using-bash-zsh-or-powershell-the-bare-repo-approach-35l3).
Here's why: your dotfiles usually live at your `/home/username/` or `~/` directory.
You may want to version these files across computers to keep certain preferences synchronized on different machines, but making a Git repository out of your entire `~/` folder is a horrible idea.
Instead, people create _bare_ repositories that only track the contents that are explicitly added to the repository, regardless of their relation to the root location of the repo's `.git` folder.

If we want one submodule repo to track all of the posts in `/content/blog`, but we don't want that repo to own the other files in that same directory, you might be able to make this happen with a bare repo.
The repo shouldn't be aware of the other files in the `content/blog`, because the repo _doesn't know that it is the same folder_ as those files.


### Instead: every post gets its own repository {#every-post}

In lieu of the bare repository solution, we opt for peak modularity: every blog post gets its own repository.

This workflow actually easier than you would think, and most of the steps are identical to what I have already covered above.
Here's what I do.


1. **Start on Github** or whichever remote service you prefer.
   Make a remote-first repository for a new post (give it a meaningful title) and copy its cloning link.
1. Add the new repo as a submodule to a new folder for that post inside of `/content/blog`.
   We assume you use a post organization model where the post's source code lives in a named folder, and the post content is written in an `index.[R]markdown` file.
   You can read more about this _page bundle_ model from Allison Hill [here](https://www.apreshill.com/blog/2020-12-new-year-new-blogdown/#step-4-create-content).
   It's advisable to use this page bundle setup to organize your blog even if you don't want to version-control all of your posts individually.
1. Your `.gitmodules` file will automatically update to reflect the new submodule.
   You will eventually want to commit that change, but it doesn't have to be now.
   If necessary, initialize/update the submodule to clone its contents into the new post directory.
1. Checkout your desired submodule branch (e.g. `main`) so you can commit changes to your blog repo.
1. Create an `index.Rmarkdown` file and edit as you normally would.
   This is also where you would take a snapshot of your R package library using `renv` which I recommend.
   Blogdown trips over the files/folders created by `renv`, however, so if you want to use it (again, you should), add `"renv"` to the `ignoreFiles` field in your `config.toml`.
   You only have to do this once per site.
1. Commit changes to the blog module repository and push to remote.
1. You should notice that your parent repository detects an updated commit in the submodule.
   Commit that change to the parent repository as well.



## Building locally and pushing to Netlify {#netlify-setup}

Once you are done getting your site looking the way you want, and all of your files are committed to the parent and child repositories, you can push your website repo to the location that Netlify is tracking.

Except, whoops, your site may fail to build on Netlify. 
Why? 
Netlify works by cloning your website repository to their servers and building it with Hugo on their end.
This process fails if Netlify can't successfully reproduce your website repo with all of the submodules declared in your `.gitmodules` file.
This happens for two benign and fixable reasons: if the submodule is a private repository or was added using the repo's `ssh` URL instead of the `https` URL.
In either case, all you have to do is add ssh-keys to these repositories so Netlify can read their contents. 
Netlify makes these keys easy to generate, and they describe it all [right here](https://docs.netlify.com/configure-builds/repo-permissions-linking/#git-submodules).

Once you add these dependencies as submodules and give Netlify permission to access them, Netlify takes care of the rest.


## No right way

This post has described a hyper-modular approach to blogging with Blogdown and Git submodules.
It presents an opinionated interpretation of a Blogdown website as a collection of modules, but you should always do what works for you.
It happened to be the case that I had particular problems that I think will be smoothed by a permanent + principled solution.


