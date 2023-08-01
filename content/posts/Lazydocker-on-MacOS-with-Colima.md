---
title: "Lazydocker on MacOS With Colima"
date: 2023-07-31T20:10:46-06:00
lastmod: 
draft: true
tags: ["macos", "docker", "colima"]
categories: ["tech"]
---

I recently discovered the very helpful Docker TUI [lazydocker](https://github.com/jesseduffield/lazydocker). Installation on MacOS is, thanks to Homebrew, pretty straightforward. Taken from the official lazydocker github repo:
- Tap into formula with recent updates:
`brew install jesseduffield/lazydocker/lazydocker`
- or install Core:
`brew install lazydocker`

That's it. You can now open lazydocker by calling `lazydocker` in your terminal.

Well, almost. When you're using [Colima](https://github.com/abiosoft/colima), lazydocker is not able to find your Docker daemon, your `.sock` file. You're greeted by this friendly red text:

![Warning when running lazydocker on MacOS with Colima for the first time.](/img/lazydockerColima.png)
> Docker event stream returned error: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

Fortunately the fix is very quick:
Add this line to your `.zhsrc` file:

`export DOCKER_HOST="unix://$HOME/.colima/default/docker.sock"`

followed by

`source .zshrc`

and restart lazydocker. 