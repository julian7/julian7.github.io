+++
date = "2016-06-28T09:29:41+02:00"
draft = true
title = "A Good Plan for building containers"
tags = ["docker", "continuous delivery"]

+++
I'm building docker containers. I've built a couple already: GitLab CE, PowerDNS, TeamSpeak3, Youtrack, Perforce (it's called Helix Versioning Engine nowadays), and Goiardi.

I'm not totally happy with building images though.

Building a docker image creates a lot of layers, for, hmm, *nothing in particular*. However, It also creates a lot of hidden dependencies, like keeping build dependencies in runtime. It can be worse though: when a new layer removes build dependencies. It's not there, yet you have to download it.

A lot of Dockerfiles start with downloading `curl` or `wget` to fetch dependencies. They don't check downloaded files' integrity. Sometimes they get rid of the downloader utilities though, usually as a new layer.

Dockerfile has no notion for tagging, uploading, versioning, even though it keeps track of all input versions.

I even went on writing my not so difficult dockerfile builder to handle artifacts, files, and templating separately.

However, what I really wanted is a separate build environment.

## Enter Habitat

[Adam Jacob](https://twitter.com/adamhjk) felt the same pain, and he took a more drastic approach: he started building Habitat, which takes the care of building, distributing, configuring, managing, and clustering completely independent applications.

### Building by the plan

Habitat does all the build in a separated environment called Studio. It allows you to download, check integrity, unpack, compile, and install artifacts. It handles dependencies for both build and run time, and allows you to template configurations. Output name and version is also well defined.

This all goes to a plan. A well-defined, versionable, easy to understand plan.

### Create, Invent, Make a mess

The fact Studio runs in a docker instance has other benefits. Your plan directory is mounted as `/src` in Studio, so you can use your favorite editor, while you can ~~rise entropy level~~ explore through the build process. If you feel you've blown everything up, just throw the old Studio away, and create a new one.

### Slim results

In Docker, it doesn't really matter which OS you're using on the desktop, so you can use which you're the most comfortable with (think of Docker for Windows / Docker for Mac). Then, you can choose another OS at server side. I mean a linux distro's tooling.

Habitat breaks these rules: why do you need an OS if you don't use it? Habitat takes care of all dependencies, starting from GLIBC to unix commands, interpreters. Just what's needed, and nothing else. It also edits executables, to use exact library versions.

When you build a docker image of a Habitat package, it adds a very slim layer of linux tools though, a busybox image. Results are very small. My goiardi docker instance is 174MB (as a comparison, my ubuntu-based docker instance is 354MB, executable is 16MB, the habitat package file is 3.2MB). This is not that bad, given glibc, gcc-libs, and openssl take 135MB.

### Manage configurations

Templates allow us to fine-tune configurations. It allows modifying configs at start time, at run time, when clustering changes, when bound services change. Everything is handled by the Supervisor, which monitors our applications.

Nothing is hidden though: clustering and bindings have to be added to the templates. Hooks can handle events like files uploaded, reconfiguration, even inquiring health checks.

### Gives way to orchestration systems

While Habitat has tight control on configuration and service discovery, it is also polite enough not to mess with scheduling, volume management, and other tasks. There are very good products which solve these issues very well. It is also not descriptive: you can run a habitat package directly, or using Mesos, or using a Docker / ACI scheduler.

I'm excited. It fixes problems I feel myself, with a level of purity and elegance I admire. I'm looking forward publishing my plans.
