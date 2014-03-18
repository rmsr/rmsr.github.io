---
layout: post
title: "pip Versus Apt for Software Deployment"
mongo: foo
categories: python
updated: 2014/03/18
---

I was asked[^1] by an old colleague whether pip or apt (in the form of Debian packages and a private repository) is better for Python server software deployment. I thought I'd put my thoughts into a blog post.

For some background, at an old company of mine, we ran our server software on a variety of physical and virtual Debian GNU/Linux machines. For software deployment, rather than roll our own solution or use one of the many automated deployment frameworks (which were, admittedly, somewhat less mature at that time), we packaged our software and various updated dependencies into .deb packages and deployed to the machines via apt installation.[^2]

In hindsight, this process was fragile, limited, overcomplex, and overall kind of silly, for a variety of reasons:

* Particularly for physical servers, your core OS update process needs to be completely separate from your software deployment process. You will be deploying your software regularly, and you don't want a flaw in this system to take down the whole machine.

* Virtual machines are ephemeral. Rather than upgrade-in-place, you should be using an architecture which tolerates rolling restarts, where you update the source disk image and clone fresh VMs to replace obsolete instances, which are then terminated.

* Your coders' development environment needs to match your test and deployment environment as closely as possible, and needs to be capable of supporting multiple virtual setups on the same laptop. Requiring them to set up chroots or whole VMs just to fix a bug is too much hurdle.

* Automated testing and deployment requires that everything be as scriptable as possible. OS-level package formats are designed for usability *by package maintainers*, not for developers or sysadmins.

* Third-party packages are often fast-moving, and you need a system which can respond simply and quickly to upstream fixes and pin things at specific versions, especially if your trunk dependencies can differ from your deployment branch. Trying to maintain fast-moving dependencies as OS packages is an extreme headache. You must run your own build system and repository. You must maintain local branches of every package you update. You must merge from both upstream and your distributions repositories. Often, an upstream update is incompatible with the existing build scripts, so you have to be a packaging expert along with all your other responsibilities.

* Rollback of system-level packages is not automatic and can be quite painful, especially for a failed upgrade. It's possible to work around this using [filesystem snapshots](https://fedoraproject.org/wiki/Features/SystemRollbackWithBtrfs).

In summary, the only time I consider it appropriate to wrap your software in an OS-level package is if you expect it to be deployed on a large number of end-user systems which are not under your control. Even then, it's not unreasonable to include private copies of some dependencies in your package.[^3]

For more complex software, you might even consider wrapping your entire setup in a lightweight appliance container, such as [Docker](https://www.docker.io), especially if you're deploying to physical machines.

[^1]: It turns out the aforementioned colleage was not asking about this, he was just asking about keeping local copies of dependencies in the form of deb packages or tarballs. I recommended to him that he use pip's [offline mode](http://pip.readthedocs.org/en/latest/user_guide.html#fast-local-installs "pip: fast and local installs"), rather than manually downloading and archiving deb dependencies as they emerged during his repo updates.

[^2]: My old company's OS deployment infrastructure was actually even sillier and more paranoid. We kept local copies of EVERY Debian package we deployed, and our machines did not use the public repositories at all. A total headache and waste of time, as we did not have anywhere near the manpower to make any meaningful use of this setup.

[^3]: This isn't always a great idea. The classic example is when a buffer overflow was found in zlib. For portability reasons, many opensource packages shipped their own private copy, and all of these had to be identified and then rebuilt to link against the system zlib. Please only include packages for which you genuinely require updated versions.
