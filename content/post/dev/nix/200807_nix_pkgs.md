+++
type = "post"
status = "in_progress"
date = "2020-08-07"

slug = "nix-pckgs"
title = "Using nix-shell as package manager for any language"
foot = "There are more dead people than living, and their numbers are increasing - Eugene Ionesco"
categories = ["DEV"]
series = ["NIX-SHELL"]
part="1"
tags = ["direnv", "nix-shell", "package", "language" ]


description = "use NIX tools such as: NixOS itself, nix-shell and nix-pkgs in a development environment."
punchline = "Getting a working development environment is often far from an easy task. Actually, setting up projects can take days before getting everything working together on a development machine."
tldr = "THING ARE ABOUT TO GET MESSY"

credits = [
    "https://ocharles.org.uk/blog/posts/2014-02-04-how-i-develop-with-nixos.html",
]

[style]
    accent = "#ffd7d7"
    theme = "dark"
    width="60%"

+++


# Enter NIX

It’s difficult to categorize Nix. On the website, it is presented as a “purely functional package manager”. However, it could also be thought of as a programming language, a configuration tool, and a build tool. In a nutshell, Nix expressions are programs written in a purely functional programming language and can return a single derivation or a set of derivations.

This difficulty to categorize comes due to a few different projects sharing similar names. So, lets just touch them breifly before we dig in:

- Nix: the package manager that installs software, and the associated programming language. This project can be installed on any Linux distribution, along with OS X.
- Nixpkgs: the common repository of software that defines how to install GHC, Emacs, etc.
- NixOS: an operating system that uses all the power of Nix and Nixpkgs, but also provides configuration management.


# Enter NIX
