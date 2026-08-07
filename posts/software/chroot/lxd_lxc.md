+++
type = "post"
status = "published"
date = "2020-08-01"
readingtime = 11

slug = "lxd-lxc:"
title = "Next-generation system container management"
thumbnail = "thumbnail.png"
foot = "God is a comic playing to an audience that's afraid to laugh - Voltaire"
description = "A practical introduction to LXD system containers and virtual machines, including profiles, storage, snapshots, backups, and the security traps worth knowing."
categories = ["LINUX"]
series = ["CONTAINERS"]
part = "1"
tags = ["container", "lxd", "lxc", "namespace", "cgroups", "wm", "cgrpups"]
credits = [
    "https://documentation.ubuntu.com/lxd/latest/installing/",
    "https://documentation.ubuntu.com/lxd/latest/howto/initialize/",
    "https://documentation.ubuntu.com/lxd/latest/tutorial/first_steps/",
    "https://documentation.ubuntu.com/lxd/latest/profiles/",
    "https://documentation.ubuntu.com/lxd/latest/howto/instances_backup/",
    "https://documentation.ubuntu.com/lxd/latest/explanation/security/",
]

punchline = "LXD gives Linux containers the manners of small virtual machines: images, networks, storage, profiles, snapshots, and one consistent command-line interface."

[style]
    accent = "#d78700"
    theme = "dark"
+++

# Why I reached for LXD

I first dug into namespaces and cgroups while building [tent](https://github.com/bresilla/tent), an experiment somewhere between `chroot` and a tiny container manager. That led me down through `liblxc` and back up to LXD, which is a much nicer place to get work done.

LXD is useful when I want a complete Linux userspace without the weight and ceremony of a conventional virtual machine. It feels like managing machines: I can boot one, get a shell, attach a disk, snapshot it, and move it elsewhere. Underneath, a container is still sharing the host's Linux kernel.

{{< image url="containers.png" border="1" width="30" padding="15" >}} LXD's container and virtual-machine model. {{< /image >}}

# LXC, LXD, and application containers

The names overlap enough to cause unnecessary confusion:

- **LXC** is the lower-level Linux container runtime and toolset.
- **LXD** manages system containers and virtual machines through a daemon and API. Its command-line client is named `lxc`.
- **Docker and OCI tools** usually package one application and its dependencies. LXD usually runs a small operating system with several processes.

A system container gets its own process tree, network interfaces, mounts, users, and resource limits through kernel namespaces and cgroups. It starts quickly and uses little memory, but it is not a separate kernel. If I need a different kernel, stronger hardware isolation, or a non-Linux guest, I use an LXD virtual machine instead.

# Install and initialize it

On Ubuntu, the supported path is the snap package:

```console
$ sudo snap install lxd
```

Add the day-to-day user to the `lxd` group, then start a fresh login or group shell:

```console
$ sudo usermod --append --groups lxd "$USER"
$ newgrp lxd
```

Membership in `lxd` is not a harmless convenience. It grants access to a daemon that can attach host paths and create privileged instances, so it should be treated as root-equivalent access. I do not add accounts I would not trust with the host.

Initialize the daemon:

```console
$ lxd init
```

The defaults create a local storage pool and a managed bridge with NAT. They are good for a laptop or a first server. On a machine that already has carefully planned storage or networking, I read every prompt instead of accepting the defaults and hoping for the best.

Check the result:

```console
$ lxc storage list
$ lxc network list
$ lxc profile show default
```

# Launch a first container

Start an Ubuntu container called `demo`:

```console
$ lxc launch ubuntu:24.04 demo
$ lxc list
```

Open a shell and inspect it:

```console
$ lxc exec demo -- bash
```

The double dash matters when the command has its own flags. For a single command there is no need to start an interactive shell:

```console
$ lxc exec demo -- cat /etc/os-release
```

The basic lifecycle is pleasantly boring:

```console
$ lxc stop demo
$ lxc start demo
$ lxc restart demo
$ lxc delete demo
```

Use `lxc delete --force demo` only when deliberately discarding a running instance.

# Move files without opening SSH

The daemon can copy files directly:

```console
$ lxc file push ./config.toml demo/etc/myapp/config.toml
$ lxc file pull demo/var/log/myapp.log ./myapp.log
```

For a directory, add `--recursive`. Ownership is interpreted inside the instance, so I check it after copying service files rather than assuming the host's numeric user IDs mean the same thing.

# Put limits and devices in profiles

Profiles are reusable bundles of configuration and devices. Instead of remembering how I configured five similar development containers, I write it once:

```console
$ lxc profile create dev-small
$ lxc profile set dev-small limits.cpu 2
$ lxc profile set dev-small limits.memory 2GiB
$ lxc profile device add dev-small work disk source="$HOME/work" path=/work
$ lxc profile add demo dev-small
```

That last disk device exposes a host directory inside the container. It is useful, but it also weakens the boundary: a compromised process can modify whatever the mapped path permits. I mount the narrowest directory possible and think carefully about UID mapping and write access.

See the effective configuration with:

```console
$ lxc config show demo --expanded
```

Instance-specific settings are fine for one-offs. If a setting expresses a class of machine, it belongs in a profile.

# Networks and storage

The default managed bridge gives containers private addresses and routes their traffic through the host. `lxc network list` and `lxc network show lxdbr0` reveal what LXD created. Public services can be exposed with a proxy device, routed NIC, or an external load balancer; the right choice depends on the network rather than on a magic container command.

Storage pools separate instance management from the underlying filesystem or volume manager. A pool may use a directory, ZFS, Btrfs, LVM, or another supported backend. I choose the backend before filling the server because changing it later means moving instances and their data.

Add separate persistent data as a custom volume:

```console
$ lxc storage volume create default app-data
$ lxc config device add demo data disk pool=default source=app-data path=/srv/data
```

Now the application's data has a lifecycle independent of the container's root filesystem.

# Snapshots are not backups

A snapshot is excellent before an upgrade:

```console
$ lxc snapshot create demo before-upgrade
$ lxc snapshot list demo
$ lxc snapshot restore demo before-upgrade
```

It is usually stored in the same pool as the instance. If that disk or pool dies, the snapshot dies with it. A backup leaves the failure domain:

```console
$ lxc stop demo
$ lxc export demo demo-$(date +%F).tar.gz --instance-only
$ lxc start demo
```

Omit `--instance-only` when the snapshots belong in the export too. Copy the archive to separate storage, verify it can be read, and periodically test an import:

```console
$ lxc import demo-2026-08-07.tar.gz demo-restored
```

An exported instance does not automatically include every external custom volume attached to it. Those volumes need their own backup plan.

# Use a virtual machine when sharing the kernel is wrong

The same client can launch a VM:

```console
$ lxc launch ubuntu:24.04 lab-vm --vm
$ lxc exec lab-vm -- bash
```

VMs cost more memory and take longer to boot, but they have their own kernel. That is the correct trade when testing kernel modules, running a workload that cannot share the host kernel, or asking for a harder isolation boundary.

# Security notes I do not skip

LXD containers are unprivileged by default, which maps container users away from the same host user IDs. I leave them that way unless a specific device or workload proves it cannot work otherwise. Setting `security.privileged=true`, passing broad host directories, or exposing sensitive devices can turn a neat boundary into decoration.

I also keep these rules:

- Do not expose the remote API casually. Authenticate trusted clients and restrict the listening network.
- Keep the host, LXD, and instance images updated.
- Limit CPU, memory, processes, and disk where noisy workloads share a host.
- Treat the `lxd` group as administrative access.
- Back up data outside the storage pool and test the restore path.

LXD makes system containers feel easy, but it cannot decide the trust boundary for me. Once that boundary is explicit, the rest of the tool is refreshingly coherent.
