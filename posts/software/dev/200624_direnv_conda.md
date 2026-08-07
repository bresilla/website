+++
type = "post"
status = "published"
date = "2020-06-24"
readingtime = 6

slug = "direnv-anaconda"
title = "Using direnv with Conda"
thumbnail = "thumbnail.png"
foot = "Artificial Intelligence is no match for natural stupidity - AI itself"
description = "Activate a project-local Conda environment when entering a directory and cleanly unload it when leaving."
categories = ["DEV"]
series = ["DIRENV"]
part = "1"
tags = ["direnv", "anaconda", "conda", "python"]
credits = [
    "https://direnv.net/",
    "https://direnv.net/man/direnv-stdlib.1.html",
    "https://docs.conda.io/projects/conda/en/stable/user-guide/tasks/manage-environments.html",
    "https://docs.conda.io/projects/conda/en/stable/dev-guide/deep-dives/activation.html",
]

[style]
    accent = "#ee82ee"
    theme = "dark"
    width = "60%"
+++

# Why combine them?

Conda manages Python interpreters, native libraries, and packages. Direnv watches the current directory and adjusts the shell environment when I enter or leave it. Put them together and each project gets its own Conda environment without the daily ritual of typing `conda activate` and later forgetting which environment is still active.

There is no magic daemon hiding behind this. Direnv finds an approved `.envrc`, evaluates it in a Bash subprocess, and exports the resulting environment difference into the current shell. When I leave the directory, it reverses that difference. Conda activation also works by emitting shell code that changes `PATH` and a few `CONDA_*` variables. The two tools fit together rather neatly.

# Install the shell hook

Install Conda and direnv using the method appropriate for your system. Then add the direnv hook to the shell.

For Bash:

```bash
eval "$(direnv hook bash)"
```

For Zsh:

```zsh
eval "$(direnv hook zsh)"
```

Put the matching line near the end of `~/.bashrc` or `~/.zshrc`, then start a new shell. Check both commands before touching the project:

```bash
direnv version
conda --version
```

I prefer not to auto-activate Conda's base environment. It keeps the normal shell boring and makes project activation obvious:

```bash
conda config --set auto_activate_base false
```

# Create a local environment

From the project root, create the environment inside the project instead of giving it a global name:

```bash
mkdir sensor-model
cd sensor-model
conda create --prefix ./.conda python=3.12 pip
```

The exact Python version is a project decision, not a universal recommendation. The useful part is `--prefix ./.conda`: the environment now lives beside the project that owns it.

Do not commit the environment directory. It contains platform-specific binaries and can be rebuilt from a manifest.

```gitignore
.conda/
.direnv/
```

# Add the `.envrc`

Create `.envrc` in the project root:

```bash
if ! has conda; then
  echo "conda is not available in PATH" >&2
  return 1
fi

eval "$(conda shell.bash hook)"
conda activate "$PWD/.conda"
```

Even if the interactive shell is Zsh or Fish, direnv evaluates `.envrc` through Bash. That is why `conda shell.bash hook` is intentional.

Direnv refuses to execute a new or modified `.envrc` until it is approved:

```bash
direnv allow
```

The prompt should now show the Conda environment, depending on the prompt theme. These checks are less pretty but more reliable:

```bash
echo "$CONDA_PREFIX"
python --version
command -v python
```

`CONDA_PREFIX` should end in `/sensor-model/.conda`, and `python` should resolve inside that directory. Move one directory up and direnv unloads it. Come back and it loads again.

# Make the environment reproducible

A local prefix gives isolation, but isolation is not reproducibility. Record the dependencies in `environment.yml`:

```yaml
name: sensor-model
channels:
  - conda-forge
dependencies:
  - python=3.12
  - numpy
  - pandas
  - pip
  - pip:
      - pyserial
```

Create the local prefix from that file:

```bash
conda env create --prefix ./.conda --file environment.yml
direnv reload
```

After changing the manifest, update the existing environment and remove dependencies that disappeared from the file:

```bash
conda env update --prefix ./.conda --file environment.yml --prune
```

Conda's solver may still select newer package builds over time unless the inputs are locked. For a research project that must be reconstructed exactly, generate platform-specific lock files with a dedicated lock tool or export an explicit package list alongside the readable `environment.yml`. I keep the readable file as the source of intent and use a lock only where exact binary reproduction matters.

# Keep secrets separate

`.envrc` is shell code. Read it before running `direnv allow`, especially after pulling somebody else's repository. Approval is a safety boundary, not an annoying confirmation dialog.

Do not put tokens in the committed file. Load a private companion file instead:

```bash
source_env_if_exists .envrc.private
```

Then ignore it:

```gitignore
.envrc.private
```

Direnv now owns directory-based activation, Conda owns the Python environment, and Git owns the small files needed to rebuild it. That is the whole setup. More importantly, leaving the project puts the shell back the way it was.
