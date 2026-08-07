+++
type = "post"
status = "published"
date = "2020-07-25"
readingtime = 8

slug = "direnv-nixshel-proji"
title = "How to manage projects, the *NIX way!"
thumbnail = "thumbnail.png"
foot = "Things will not calm down, as a matter of fact they will just calm up - Teal'c (Stargate)"
description = "A small project registry, automatic Nix development shells, and a Rofi launcher without a giant IDE workspace."
categories = ["DEV"]
series = ["DIRENV"]
part = "2"
tags = ["direnv", "nix-shell", "nix-direnv", "rofi", "projects", "proji"]
credits = [
    "https://direnv.net/",
    "https://nix.dev/guides/recipes/direnv.html",
    "https://github.com/nix-community/nix-direnv",
    "https://davatorium.github.io/rofi/",
]

[style]
    accent = "#fef460"
    theme = "dark"
+++

{{< image url="projects-menu.png" border="1" width="70" >}} A Rofi menu listing active projects {{< /image >}}

# The itch

My projects never lived in one language, one editor, or even one sensible directory. A robotics experiment might need Python, a compiler project wants Rust and LLVM, and an old blog tool still expects something I installed three years ago and immediately forgot about. The code is usually easy to find. Reconstructing the exact environment is the annoying part.

I originally called my little project manager **Proji**. That repository is retired now, but the useful idea was much smaller than the program: keep projects in predictable places, store the environment beside the code, and generate the launcher menu from data instead of maintaining it by hand.

The current version of that workflow uses four boring pieces:

- Git owns the project.
- Nix describes the tools.
- direnv activates them when I enter the directory.
- A tiny index feeds Rofi, `fzf`, or any other menu.

No database is required. This is project management in the literal sense of "where did I put the thing?"

# Give projects a home

I use a shallow directory tree grouped by broad purpose:

```text
~/code/
├── experiments/
├── research/
├── tools/
└── websites/
```

The categories are not taxonomy. They are there so `find ~/code -maxdepth 3` remains useful. A project should not move every time my opinion about it changes.

For a small setup, the filesystem itself can be the registry:

```bash
find "$HOME/code" -mindepth 2 -maxdepth 2 -type d -name .git -printf '%h\n'
```

I prefer an explicit file because it lets me hide archived repositories and attach a short label. Save this as `~/.config/projects.tsv`:

```text
faqe	/home/me/code/tools/faqe
website	/home/me/code/websites/website
plant-model	/home/me/code/research/plant-model
```

Tabs separate the label and path. That is enough structure for a menu and still easy to edit without a special client.

# Put the environment in the repository

Here is a minimal `flake.nix` for a Rust project:

```nix
{
  description = "project development shell";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in {
      devShells.${system}.default = pkgs.mkShell {
        packages = with pkgs; [
          cargo
          rustc
          rustfmt
          pkg-config
        ];
      };
    };
}
```

Run `nix flake lock` once and commit both files. The lock file pins the inputs; without it, "use nixpkgs unstable" means a different package set later.

Enter manually with:

```bash
nix develop
```

For an older project that uses `shell.nix`, the equivalent is:

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-26.05";
  pkgs = import nixpkgs {};
in
pkgs.mkShell {
  packages = with pkgs; [ git jq ];
}
```

Then run `nix-shell`. Flakes and `shell.nix` solve the same immediate problem here: the repository states which tools belong in its development shell.

# Let direnv do the repetitive bit

Install direnv and hook it into the interactive shell. For Bash:

```bash
eval "$(direnv hook bash)"
```

For Zsh:

```zsh
eval "$(direnv hook zsh)"
```

With nix-direnv installed, a flake-backed `.envrc` is one line:

```bash
use flake
```

For `shell.nix`:

```bash
use nix
```

Approve it after reading it:

```bash
direnv allow
```

Now `cd` into the project. The tools appear. Leave it and they disappear from `PATH`. When `flake.nix`, `flake.lock`, or `shell.nix` changes, direnv reloads the environment.

The approval step matters because `.envrc` is executable shell code. Never auto-approve every repository you clone. That would turn a useful safety check into theatre.

# Build the launcher

This script reads the registry, asks Rofi to choose a label, and opens the selected project in a terminal. Save it as `~/.local/bin/project-menu`:

```bash
#!/usr/bin/env bash
set -euo pipefail

registry="${XDG_CONFIG_HOME:-$HOME/.config}/projects.tsv"
choice="$(cut -f1 "$registry" | rofi -dmenu -i -p 'project')"
test -n "$choice" || exit 0

path="$(awk -F '\t' -v name="$choice" '$1 == name { print $2; exit }' "$registry")"
test -d "$path" || {
  printf 'project path does not exist: %s\n' "$path" >&2
  exit 1
}

exec foot --working-directory="$path"
```

Replace `foot` with the terminal you actually use. For example, Kitty accepts `kitty --directory "$path"`, while Alacritty accepts `alacritty --working-directory "$path"`.

Make it executable:

```bash
chmod +x ~/.local/bin/project-menu
```

Bind that script to a desktop shortcut. I use the menu for old projects more than active ones; active projects are already sitting in a terminal somewhere.

If Rofi is not available, the same registry works with `fzf`:

```bash
choice="$(cut -f1 "$registry" | fzf --prompt='project> ')"
```

# Add projects without editing TSV by hand

A helper keeps the registry free of duplicate labels:

```bash
#!/usr/bin/env bash
set -euo pipefail

name="${1:?usage: project-add NAME PATH}"
path="$(realpath "${2:?usage: project-add NAME PATH}")"
registry="${XDG_CONFIG_HOME:-$HOME/.config}/projects.tsv"

mkdir -p "$(dirname "$registry")"
if test -f "$registry" && cut -f1 "$registry" | grep -Fxq "$name"; then
  printf 'project already exists: %s\n' "$name" >&2
  exit 1
fi

printf '%s\t%s\n' "$name" "$path" >> "$registry"
sort -o "$registry" "$registry"
```

There is deliberately no clone command in it. Cloning a repository, choosing its directory, and deciding whether to trust its `.envrc` should remain visible operations.

# What this setup does not do

It does not track tasks, issues, time, or project status. GitHub and GitLab already do some of that, and plain text does the rest. It also does not make a Nix shell perfectly reproducible by itself. Pin the package source, commit the lock file, and remember that external services and mutable data still exist.

What it does give me is a repeatable entrance. Pick a project, land in its directory, and get the tools it declared. Proji was the first version of that idea. The tiny version has survived longer because there is almost nothing to break.
