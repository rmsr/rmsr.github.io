---
layout: post
title: "pip Versus Apt for Software Deployment"
date: 2014-03-13 14:43:00
---

I was asked via email via an old colleague whether pip or apt (in the form of Debian packages and a private repository) is better for Python server software deployment. I thought I'd put my thoughts into a blog post.

For some background, at an old company of mine, we ran our server software on a variety of physical and virtual Debian GNU/Linux machines. For software deployment, rather than roll our own solution or use one of the many automated deployment frameworks (which were, admittedly, somewhat less mature at that time), we packaged our software and various updated dependencies into .deb packages and deployed to the machines via apt installation.

In hindsight, this process was fragile, limited, overcomplex, and overall kind of silly, for a variety of reasons:

* Especially for physical servers, your core OS update process needs to be completely separate from your software deployment process. You will hopefully be updating your software regularly, and you don't want a flaw in this process to take the whole machine down.

* Virtual machines are ephemeral. Rather than upgrade-in-place, you need to have an architecture which tolerates rolling restarts, where you update the source image and spin up fresh VMs to replace obsolete instances, which are then terminated.

* Your coders' development environment needs to match your test and deployment environment as closely as possible, and needs to be capable of supporting multiple virtual setups on the same laptop. Requiring them to set up chroots or whole VMs just to fix a bug is too much hurdle.

* Automated testing and deployment requires that everything be as scriptable as possible. OS-level packages are targeted towards stability and flexibility *for the package maintainers*, not for your devs or sysadmins.

* Third-party packages are often fast-moving, and you need a system which can respond flexibly and quickly to upstream fixes, or the need to pin at a specific version, especially if these are different for trunk and your deployment branch. Trying to cram fast-moving dependencies into OS packages can be an extreme headache. You have to maintain your own build and repo infrastructure. You have to keep local branches of every package you update. Often, the upstream package update is incompatible with the packaging metadata, so you have to be a packaging expert along with the rest of your responsibilities.

Basically, the only time I consider it appropriate to wrap your software in an OS-level package is if you expect it to be deployed on a large number of end-user systems which are not under your control. Even then, it's entirely permissible to include your versioned dependencies in the package.

For more complex software, you might even consider wrapping your entire setup in a lightweight appliance container, such as [https://www.docker.io/](Docker), even if you're deploying to physical machines.
