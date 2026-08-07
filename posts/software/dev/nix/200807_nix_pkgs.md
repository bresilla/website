+++
type = "post"
status = "published"
date = "2020-08-07"
readingtime = 8

slug = "nix-pckgs"
title = "Using Nix as a package manager for any language"
thumbnail = "thumbnail.png"
foot = "There are more dead people than living, and their numbers are increasing - Eugene Ionesco"
categories = ["DEV"]
series = ["NIX-SHELL"]
part = "1"
tags = ["direnv", "nix-shell", "package", "language"]
description = "Use Nix for project tools while leaving Cargo, pip, npm, and other language package managers to manage source dependencies."
punchline = "A project depends on more than its language libraries. Nix can supply the compiler, database client, code generator, and native libraries without installing them globally."
tldr = "Start with nix-shell -p for temporary tools, commit a pinned shell.nix or flake for projects, and keep language lock files for language dependencies."
credits = [
    "https://nix.dev/tutorials/first-steps/ad-hoc-shell-environments.html",
    "https://nix.dev/tutorials/first-steps/declarative-shell.html",
    "https://nix.dev/guides/recipes/direnv.html",
    "https://search.nixos.org/packages",
]

[style]
    accent = "#5fd7ff"
    theme = "dark"
    width = "60%"
+++

# The dependency nobody wrote down

A Python project may have a perfect `requirements.txt` and still fail because `libpq` is missing. A Rust project can pin every crate and still need Clang, CMake, Protobuf, or OpenSSL from the host. JavaScript packages occasionally assume a C compiler exists. Even a shell script quietly depends on the exact behavior of `sed`, `curl`, and whatever else happens to be in `PATH`.

Language package managers handle language packages well. I do not want Nix to replace Cargo or pip merely because it can. I want Nix to describe the layer they usually ignore: interpreters, compilers, command-line tools, native libraries, and environment variables.

That split keeps the setup understandable:

- Nix provides system-level project tools.
- Cargo, pip, npm, Go modules, and similar tools provide source dependencies.
- Each side commits its own lock file.

# Borrow a package for one command

Suppose `jq` is not installed. There is no need to add it to the whole system:

```bash
nix-shell -p jq
```

Inside that shell:

```bash
jq --version
```

Exit and it disappears from `PATH`. Run a command without entering an interactive shell:

```bash
nix-shell -p jq curl --run 'curl -s https://api.github.com | jq .current_user_url'
```

The newer command interface expresses the same idea like this:

```bash
nix shell nixpkgs#jq nixpkgs#curl
```

These commands are convenient, not reproducible by themselves. If `nixpkgs` is not pinned, running the command next year may select different package versions.

Use [search.nixos.org](https://search.nixos.org/packages) to find package attribute names. The executable and package names are not always identical.

# Commit a development shell

For a project, write the environment down. This `shell.nix` gives a C project Meson, Ninja, pkg-config, and OpenSSL headers:

```nix
let
  nixpkgs = fetchTarball "https://github.com/NixOS/nixpkgs/tarball/nixos-26.05";
  pkgs = import nixpkgs { config = {}; overlays = []; };
in
pkgs.mkShell {
  packages = with pkgs; [
    meson
    ninja
    pkg-config
  ];

  buildInputs = with pkgs; [
    openssl
  ];
}
```

Run:

```bash
nix-shell
meson setup build
ninja -C build
```

The pinned Nixpkgs release is much better than importing `<nixpkgs>`, which depends on the user's channel. For stronger reproducibility, replace the release branch with an exact Git commit and a fixed hash, or use a flake lock.

`packages` is the right home for tools executed during development. Libraries used for compilation often belong in `buildInputs`. Nix's compiler wrappers then supply the necessary search flags. The exact distinction matters more for packaging than for a simple shell, but using it correctly avoids confusion later.

# The same shell as a flake

Flakes put input locking into the normal workflow:

```nix
{
  description = "C development environment";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in {
      devShells.${system}.default = pkgs.mkShell {
        packages = with pkgs; [
          meson
          ninja
          pkg-config
        ];
        buildInputs = with pkgs; [ openssl ];
      };
    };
}
```

Create and commit the lock file:

```bash
nix flake lock
git add flake.nix flake.lock
```

Enter with:

```bash
nix develop
```

The example names `x86_64-linux` directly to keep the Nix code readable. A real multi-platform repository should generate shells for every supported system rather than pretending one platform string is portable.

# Examples by language

The pattern stays the same even when the tools change.

Python with native numerical libraries:

```nix
pkgs.mkShell {
  packages = with pkgs; [ python312 uv ];
  buildInputs = with pkgs; [ stdenv.cc.cc.lib ];
}
```

Rust with bindgen:

```nix
pkgs.mkShell {
  packages = with pkgs; [ cargo rustc rustfmt clang pkg-config ];
  LIBCLANG_PATH = "${pkgs.libclang.lib}/lib";
}
```

Node with a database client used by migrations:

```nix
pkgs.mkShellNoCC {
  packages = with pkgs; [ nodejs_22 pnpm postgresql_16 ];
}
```

These snippets do not replace `uv.lock`, `Cargo.lock`, or `pnpm-lock.yaml`. They make sure the expected interpreter and external tools exist before the language package manager starts.

# Load it automatically

Direnv can load the shell when entering the repository. With a `shell.nix`, create `.envrc` containing:

```bash
use nix
```

With nix-direnv and a flake:

```bash
use flake
```

Then approve it:

```bash
direnv allow
```

Read `.envrc` and the Nix expression before approving them. Nix builds are designed for isolation, but development shells can execute `shellHook` code and expose whatever credentials already exist in the environment.

# What reproducible means here

A pinned Nix input fixes the package definitions. Nix store paths include hashes derived from build inputs, so conflicting versions can coexist instead of fighting over `/usr/lib`. This removes a large class of "works on my machine" failures.

It does not freeze the universe. A development command can still download unpinned data, contact a mutable service, read files outside the repository, or behave differently across operating systems. Reproducibility comes from closing those gaps one by one, not from adding the word Nix to a README.

I usually start with `nix-shell -p` when exploring. Once the tool list stops changing every five minutes, I move it into `shell.nix` or a flake and pin the input. That is enough for most projects: no global compiler pile, no mystery `PATH`, and no attempt to force every language dependency through one package manager.
