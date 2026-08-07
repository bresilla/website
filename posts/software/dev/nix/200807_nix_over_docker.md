+++
type = "post"
status = "published"
date = "2021-02-09"
readingtime = 7

slug = "docker"
title = "Running the Nix package manager inside a Docker container"
foot = "The more I want to get something done, the less I call it work - Richard Bach"
categories = ["DEV"]
series = ["NIX"]
part = "4"
tags = ["docker", "linux", "nix"]
thumbnail = "docker-thumbnail.jpg"
description = "Try Nix without installing it on the host, mount a project into the official image, and keep the Nix store between runs."
punchline = "Nix inside Docker is useful for experiments and CI. Docker supplies the disposable machine; Nix supplies the package graph and development environment."
tldr = "Use the official nixos/nix image, mount only the project you need, and persist /nix in a named volume if repeated downloads become annoying."
credits = [
    "https://nixos.org/download/",
    "https://releases.nixos.org/nix/nix-2.34.0/manual/installation/installing-docker.html",
    "https://nix.dev/tutorials/first-steps/ad-hoc-shell-environments.html",
]

[style]
    accent = "#f72c4a"
    theme = "light"
    width = "60%"
+++

# Why put a package manager in a container?

Running Nix inside Docker sounds like putting a toolbox inside another toolbox. It is. Sometimes that is exactly what I want.

The container gives me a disposable Linux userspace and keeps the experiment away from the host. Nix lets me fetch exact tools or enter a development environment declared by a repository. This is handy on a CI runner, on a machine where I cannot keep Nix installed, or when I simply want to poke it and throw everything away afterward.

It is not the best way to use Nix every day. Starting a container adds friction, filesystem permissions can become irritating, and macOS still runs the Linux container through Docker's virtual machine. If Nix is part of the normal workflow, install Nix on the host. Use the container when disposability is the point.

# Start with the official image

The Nix project publishes `nixos/nix` on Docker Hub. Start an interactive shell:

```bash
docker run --rm -it docker.io/nixos/nix:latest
```

Inside the container:

```bash
nix --version
nix-shell -p hello --run hello
```

`--rm` deletes the container after exit. The downloaded store paths disappear with it, which is clean but slow if the same command is run repeatedly.

For a reproducible CI job, do not rely on `latest`. Pin a version tag or, better, the image digest recorded by `docker image inspect` or the registry. Tags are names; digests identify image content.

# Work on a host project

Mount the current directory at `/work` and make it the working directory:

```bash
docker run --rm -it \
  --mount type=bind,src="$PWD",dst=/work \
  --workdir /work \
  docker.io/nixos/nix:latest
```

The project files are now visible inside the container. A repository with `shell.nix` can be entered with:

```bash
nix-shell
```

For a flake development shell:

```bash
nix --extra-experimental-features 'nix-command flakes' develop
```

The command-line flag avoids quietly depending on a host configuration file. A custom image can enable those features permanently, but an explicit flag is useful in examples because it shows the requirement.

The container runs as root by default. Any files it creates on the bind mount may therefore become root-owned on a Linux host. Most Nix development commands only create `result` symlinks or write to the Nix store, but build tools may write into the project. Check ownership after running an unfamiliar build. Trying to solve this with `--user` immediately can break write access to `/nix`; a deliberately built non-root image is cleaner than a pile of runtime permission flags.

# Keep the Nix store

Create a named volume:

```bash
docker volume create nix-store
```

Mount it at `/nix`:

```bash
docker run --rm -it \
  --mount type=volume,src=nix-store,dst=/nix \
  --mount type=bind,src="$PWD",dst=/work \
  --workdir /work \
  docker.io/nixos/nix:latest
```

The first container populates the volume. Later containers reuse downloaded store paths, profiles, and database state. Remove it when the cache is no longer useful:

```bash
docker volume rm nix-store
```

This volume is a cache, not a backup. Anything important should live in the project repository or another intentional storage location.

# Run one command and leave

An interactive shell is unnecessary in CI. This runs a command in a temporary Nix environment and exits:

```bash
docker run --rm \
  --mount type=volume,src=nix-store,dst=/nix \
  docker.io/nixos/nix:latest \
  nix-shell -p git jq --run 'git --version && jq --version'
```

For a mounted flake project:

```bash
docker run --rm \
  --mount type=volume,src=nix-store,dst=/nix \
  --mount type=bind,src="$PWD",dst=/work \
  --workdir /work \
  docker.io/nixos/nix:latest \
  nix --extra-experimental-features 'nix-command flakes' develop --command make test
```

Replace `make test` with the repository's own verification command. The important bit is that the flake lock and source tree come from the project, while `/nix` is disposable cache state.

# A small custom image

If every invocation repeats the same options, put them in an image:

```dockerfile
FROM docker.io/nixos/nix:latest

RUN printf '%s\n' \
    'experimental-features = nix-command flakes' \
    'sandbox = false' \
    >> /etc/nix/nix.conf

WORKDIR /work
```

Build it:

```bash
docker build -t local/nix-dev .
```

Then use it like the official image:

```bash
docker run --rm -it \
  --mount type=volume,src=nix-store,dst=/nix \
  --mount type=bind,src="$PWD",dst=/work \
  local/nix-dev
```

Nix's own build sandbox is disabled in this example because the process is already inside a restricted container and nested sandbox support varies with the Docker runtime. Docker is still not a security boundary for hostile builds by default. Do not mount the Docker socket, SSH directory, cloud credentials, or the whole home directory into a build you do not trust.

# Docker image or Nix image?

This setup uses Nix *inside* an ordinary container. Nix can also *build* OCI images with `dockerTools`, which solves a different problem: producing an image from Nix derivations. I use the official image when I need a quick Nix-capable runner. I use `dockerTools` when the image itself is a build output that must be described by Nix.

The nested-toolbox arrangement is not elegant, but it is honest. Docker controls the disposable runtime. Nix controls the tools inside it. When the experiment is over, remove the container and, if desired, the store volume. Nothing else on the host has to know it happened.
