
+++
title = "Monorepo vs polyrepo"
date = "2021-05-24" 
+++

# Structuring code base 

In general, there are two ways to lay out your code: mono- or polyrepo. The problem is well described [here](https://earthly.dev/blog/monorepo-vs-polyrepo/)
by Earthly, but it is inconclusive. Well put by reddit user *inhumantsar*
> I feel like I can tldr all articles like this without reading them *There are pros and cons of each so do whichever your team prefers*
>Edit: lol yep. "We weren't dogmatic about it and built a hybrid that works for us"
>This stuff is so hugely dependent on tooling and workflow and personal preference that in the end, it simply does not matter which you pick, only that you make a decision and be consistent right up until it stops being the right decision.

# Deciding on the layout 

I used to think it does not matter. When joining the project it is always structured and it is easier to keep it the way it is. The challenge is when you have to decide how to structure your own project. 

I found three main aspects:
* contributing
* releasing 
* running continuous integration

And there is no layout suiting all, see the table below

| | mono | poly |
--- | --- | ---
| contribute | + | - |
| release | - | + |
| test | + | - |

To decide you can ask the following questions:
* Will you release sub-projects of your project to the outside world?
* How big is your code base?

If you want to release a sub-project (library, a module) and you don't want to make your whole repo public, you have to go polyrepo. 
If your code base is huge (>1GB), monorepo without extra tooling will be slow.

# Tooling

In general monorepo is comfy when working. At /dev we really enjoyed working with monorepos but at some point we wanted to open the code which was a part of a bigger project. The change from mono- to poly- was inevitable. We were  looking for a tool which will help us manage repos.
I took the [list from StackOverflow](https://stackoverflow.com/questions/6500524/alternatives-to-git-submodules) and started reading and trying. When reading on the tools, I also came across the [git-subrepo](https://github.com/ingydotnet/git-subrepo), which turned out to be the simplest tool to get s**it done. Kudos Ingy döt Net!

# Go hybrid!

With git-subrepo you can decide to extract a part of your project to a subrepo at any stage of your project. This is IMHO a killer feature. You can work in a comfy monorepo, and when it's time, you cast two spells and *woosh* part of your work lands on GitHub! At the same time, nothing changes for your project, you keep working in monorepo. The only change is that you need a maintainer, a person who will decide if to push the latest changes outside or if to pull contributions from the subrepo.

# Links

* [Monorepo vs Polyrepo, Vlad A. Ionescu](https://earthly.dev/blog/monorepo-vs-polyrepo/)
* [Reddit discussion](https://www.reddit.com/r/programming/comments/l5i8sv/monorepo_vs_polyrepo/)
* [You too can love the MonoRepo, Alex Eagle](https://slashdev.team/)
* [4 Git Submodules Alternatives You Should Know, Jonathan Saring](https://codeburst.io/4-git-submodules-alternatives-you-should-know-592095859b0)
* [git-subrepo](https://github.com/ingydotnet/git-subrepo)