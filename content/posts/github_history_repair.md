+++
title = "Github commits' emails change"
author = ["Stanislav Arnaudov"]
description = "A short description of a solution to a Github related problem with the email addresses associated with a commit history"
date = 2018-06-22T00:00:00+02:00
keywords = ["github", "git", "commit", "change", "history"]
lastmod = 2019-10-12T00:52:35+02:00
categories = ["github"]
draft = false
weight = 100
+++

## Abstract {#abstract}

You know that _Contributions_ panel on your [Github](http://github.com/) profile page. Let's face it, developers these days take much pride when they can show their consistent work and progress of their own/forked repositories... even when some of their commits are nothing more than an implementation of a single function. Or of course, it may be just me with that pride 😒. Said panel is a clear indication to your future employee or members of your "coding community" that you are pretty much a big deal with a good work ethic. Imagine in what of a situation you would be if for some random reason your commits didn't show up there. This post addresses the issue of not having a proper _git_ configuration (`~/.gitconfig`) file on your system and commits to Github with the wrong email address.


## My problem with GitHub and some contributions in limbo {#my-problem-with-github-and-some-contributions-in-limbo}

So, yeah, I had the problem myself. I was committing things regularly to my repositories on Github and but only a very small portion of my commits were showing in the _Contributions_ panel. It took me a while to notice that and I was kinda freaked about it as I thought that my "hard work" will be forgotten and the future generations would not be able to see it. <br /> <br /> A little research (I literally just clicked on [Learn how we count contributions](https://help.github.com/articles/why-are-my-contributions-not-showing-up-on-my-profile/)) on how Github counts contributions and I found out that the email address of the commits must match the main email of your Github account. Some other hints are taken from the Github page about how contributions are counted:

> Commits will appear on your contributions graph if they meet all of the following conditions:
>
> -The email address used for the commits is associated with your GitHub account. -The commits were made in a standalone repository, not a fork. -The commits were made:
>
> -   In the repository's default branch (usually master)
> -   In the _gh-pages_ branch (for repositories with [Project Pages](https://help.github.com/articles/user-organization-and-project-pages/#project-pages-sites) sites)

The thing that I found out was happening - I would cloned one of my repositories, do some work on it and then commit the work it and push it on Github. The email address for the commit, however, would be set to `arnaud@localhost.localdomain` which is some weird default configuration of Debian and most cetrantly not the address I want to use on Github. This makes it think that the commits aren't coming from me.


## Proper github configuration {#proper-github-configuration}

In order to avoid the problem of the future, you have two options. You can set your name and email address manually for each repository you are working on. Just go in the repository's folder and set the `user.name` and `user.email` fields like:

```sh
git config user.name 'Stanislav Arnaudov' && \
git config user.emial 'email_adress@somewhere.org'
```

This sets the name and the email on a repository "local" level. After that, each commit will have this name and email address. If you want to set a global name and email address for your system and not worry about each individual repository, you would have to edit (create if you don't have it) your `~/.gitconfig` file. Somewhere in there, you should have something like

```conf
[user]
        name = Stanislav Arnaudov
        email = email_adress@somewhere.org
```

This sets the name and email globally and every commit made on your system will have the proper fields. <br /> <br /> On Github you can see the raw path of a commit with `https://github.com/<user_name>/<repository>/commit/<commit_id>.patch`. On this page, you could inspect the commit and see exactly which email address is associated with it.


## Fixing the your repositories {#fixing-the-your-repositories}

But of course, so far we've only prevented the problem of occurring in the future. You probably want to know how do we fix the previous "damage". This section describes that and my approach to tackling it.

<br />

First, some of my experiments and findings. The thing about git repositories - everything is stored in that nifty `.git` folder. Every delta, every branch, every everything. So, I supposed that if I go there and start changing things, I could probably change the email address of past commits. If you run _grep_ in your repository like:

```sh
grep -r "arnaud@localhost.localdomain" ./.git
```

you get all the occurrences of your "wrong" email in your commits... or everywhere else for that matter. OK, so if you change those Github probably will notice and everything will be OK, right? So, I run _[sed](https://en.wikipedia.org/wiki/Sed)_ in conjunction with _find_ like:

```sh
find ./.git/ -type f -exec sed 's/arnaud@scc-wkit-clxa-23-139.scc.kit.edu/email_adress@somewhere.org/' -i {} +;
```

This finds every regular file in the _.git_ folder and runs `sed 's/arnaud@localhost.localdomain/email_adress@somewhere.org/' -i` on it. With the expression in the quotation marks _sed_ replaces everything that matches `arnaud@localhost.localdomain` with `email_adress@somewhere.org`. The `-i` means "in place" so be careful when running _sed_ like that because you may break something. So I do that and then push onto Github, And nothing happened. The thing is `git push` the things "in" your repository and kinda is disregarding of changes in `.git`. In some cases, git may refuse to make the push, crying about "histories don't match" (or something). In those cases, one would have to call push as

```sh
git push --force
```

which overwrites 'everything' in the Github repository. So I do that and....again, this didn't work. When I expected the commits in the repository on Github my profile picture still wasn't there. I am not sure what the reason for that is although I was pretty sure I would change at least _something_ while messing with the _.git_ folder.

<br />

Here I will stop rambling and boring you with my "personal experience" and just give you the solution.


### The right solution {#the-right-solution}

After some more research, I found [this help page](https://help.github.com/articles/changing-author-info/) on Github. I guess the problem is not uncommon. The page describes the whole process of "re-authoring" your commit history. They even provide a script that could be run in a bare repository (one only for pushing and pulling from) that changes everything. Something to note - the provided script must be given old (wrong) and new (the right one) email to re-placed. This is done because while preforming the "hack" on a shared repo where not one but a few developers have commits. Of course, you don't really want to overwrite the email of someone else. <br /> <br /> BUT... in my repositories, I had commits also made from my university's network and the emails of those were completely other beasts. Therefore I change the script a little bit and wrote an extra supplementary script that does absolutely everything. If you want to automate the whole process, copy this:

```sh
#!/bin/bash

echo "Cloning repo..."
git clone --bare $1 new_repo
cd ./new_repo
echo "Running reindex script..."
git filter-branch --env-filter '
CORRECT_NAME="<user name>" \
CORRECT_EMAIL="<user email>" \
export GIT_COMMITTER_NAME="$CORRECT_NAME" \
export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL" \
export GIT_AUTHOR_NAME="$CORRECT_NAME" \
export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL" \
' --tag-name-filter cat -- --branches --tags
echo "Hard-pushing to origin..."
git config --global credential.helper cache
git push --force --tags origin 'refs/heads/*'
cd ..
echo "Deleting folder..."
rm -rf new_repo
```

in a `.sh` file, adjust the `<user_name>` and `<user email>` fields with your name and email and run the script like `./file.sh https://github/<user>/<repo>.git` This will:

1.  Clone the repo in a new folder with name new\_repo
2.  Run a slight modification of Github's script
3.  Push the change repo on Github (eventually, you will be asked to give your username and password)
4.  Deletes the newly created folder.

In the end, everything is as it was but all of the commits in the repo will have the email that was in the script. <br /> <br /> <span class="underline">Note:</span> This will create a file in your home directory with name .git-credentials and it will contain your Github username and password. I do this so I can use `git config --global credential.helper cache` which removes the necessity of typing out your name and password on every commit. (Yes, I know, I should set up the [ssh thing](https://help.github.com/en/articles/connecting-to-github-with-ssh), shut up) <br /> <br /> **Caution:** My script changes the commiter's emails and names of <span class="underline">**ALL**</span> commits. If you run the script on a repository with several developers, you could potentially and most likely will cause some people to be angry with you... and rightfully so! I mean, you just ruined someone's _Contributions_ panel... how would you like that if done to you. <br /> <br /> And there you have it. With your newly acquired knowledge you could transform your _Contributions_ panel form this: ![](/ox-hugo/panel_bad.png) to this. ![](/ox-hugo/pane_good.png) And you know, the ultimate bragging rights are yours!
