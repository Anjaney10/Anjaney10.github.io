---
layout: post
title: "Migrating from Conda to Pixi: A cleaner and faster alternative"
date: 2026-07-13 09:00:00 +0000
categories: [Tutorials]
tags: [bioinformatics, package management, rust]
author: Anjaney
---
If you have been working with data science tools, Python, and R for a while, you know the struggle. 

I recently found myself staring at a massive, cluttered `work/` directory full of Nextflow environments, old `r-miniconda` folders I forgot I even installed, and ghost kernels haunting my VS Code dropdown menus. 

It felt like my system was groaning under the weight of gigabytes of stale dependencies.

That is the exact moment I decided to completely nuke Miniconda from the system and switch to **Pixi**.

[Pixi](https://prefix.dev/blog/pixi_a_fast_conda_alternative) is a modern, blazing-fast package manager built in Rust. 

Instead of managing global environments that inevitably bleed into one another, Pixi manages dependencies purely on a *per-project* basis.

Before you take the plunge, here is a breakdown of what makes Pixi great, where it falls short, and exactly how to migrate your existing Conda setups.

---

## Pixi vs. Conda: The Pros and Cons

### The Pros of Pixi

1. **True Project Isolation:** Pixi stores your environment in a hidden `.pixi` folder directly inside your project directory. This means no more global environment pollution, and deleting a project safely deletes its environment, meaning no leftover cache eating up your hard drive.
2. **Lightning Fast:** Because it uses the `rattler` engine (built in Rust), resolving dependencies and downloading packages is significantly faster than standard Conda.
3. **Built-in Reproducibility:** Pixi automatically generates a `pixi.lock` file every time you change a dependency. If you share your repository with a colleague, they are guaranteed to get the exact same package versions, down to the system libraries.

### The Cons of Pixi

1. **No "Base" Environment:** If you are used to just opening a terminal and typing `python` using your base Conda environment, you will have to adjust. Pixi requires you to be inside a project or use global installs cautiously.
> _Not a big problem though, just some behavioural change required_
3. **Disk Space Duplication:** Because environments are local to the project, if you have ten projects using the same massive libraries (like PyTorch or Seurat), you will use more disk space unless you configure global package caching effectively.
> _Easy to overcome this using global packages_
4. **The Learning Curve:** You have to learn a new `pixi.toml` configuration syntax, and managing specific channel priorities (like `conda-forge` vs. `bioconda`) requires explicit setup rather than relying on a global `.condarc` file.
> _Again can be learnt very easily. In fact, most of the times, you'll not even need to open the `pixi.toml` file._

---

## Starting from Scratch: Installing Pixi and Initializing a Fresh Project

If you do not have a Conda environment to export, or if you simply want to start with a completely clean slate, getting Pixi up and running takes less than a minute. First, you need to install Pixi system-wide. Open your terminal and run the official standalone installation script. This will download the Pixi binary and automatically add it to your system's PATH.

> Refer to this for more details from the official developers: https://pixi.prefix.dev/latest/installation/

```bash
curl -fsSL https://pixi.sh/install.sh | bash

```

Once the installation finishes, either restart your terminal or refresh your shell configuration (for example, by running `source ~/.bashrc` or `source ~/.zshrc`) so your system recognizes the `pixi` command.

Next, let's create a brand-new project without relying on any old YAML files. Navigate to your desired workspace directory and initialize the project from scratch:

```bash
# Initialize a new Pixi project
pixi init my_fresh_project

# Move into the new directory
cd my_fresh_project

```

Unlike Conda, which builds a heavy environment immediately, this initialization command simply generates a lightweight `pixi.toml` configuration file. The actual `.pixi` environment folder is not created until you add your first package. To kickstart your data science setup, you can declare your core languages and libraries directly from the command line:

```bash
# For a Python-based project:
pixi add python jupyter pandas

# Or for an R-based project:
pixi add r-base r-irkernel r-ggplot2

```

As soon as you run the `add` command, Pixi reaches out to the repositories, resolves the dependencies using the lightning-fast `rattler` engine, locks the specific versions in a `pixi.lock` file, and builds your isolated environment on the fly. From there, you simply type `pixi shell` to activate the environment, and you are ready to start coding—no messy base environments required.

---

## Migrating from an existing Conda environment

### Step 1: Exporting Your Conda Environment and Initializing Pixi

If you have a working Conda environment, you can export it and use it to seed your new Pixi project.

First, export your existing environment to a YAML file:

```bash
# Activate your old conda environment
conda activate my_old_env

# Export it to an environment.yml file
conda env export > environment.yml

# For clean environment.yml export use this:

conda env export --from-history > environment.yml

# this ensures only explicitly installed libraries are listed unlike the system packages which can cause conflict

```

Now, let's create a brand new Pixi project using that file:

```bash
# Initialize a new Pixi project by importing the yaml file
pixi init --import environment.yml my_new_project

# Move into your new project directory
cd my_new_project

```

This command automatically translates your Conda dependencies into a `pixi.toml` file and creates the isolated `.pixi` environment directory.

---

### Step 2: Testing, Adding, and Removing Packages

To interact with your new environment, you need to enter the Pixi shell (which replaces `conda activate`).

```bash
# Activate the local project environment once inside the my_new_project directory
pixi shell

# Verify that Python or R is running from the local .pixi path
which python
which R

# the paths should be inside .pixi/...

```

Managing packages is incredibly straightforward. It updates your `pixi.toml` and lockfile automatically.

```bash
# Add a new package from conda-forge
pixi add pandas
# or use the shorthand:
# pixi a pandas

# Remove a package you no longer need
pixi remove pandas

```

---

## Troubleshooting the Bioconductor `org.Hs.eg.db` Issue

Bioconductor packages can sometimes be finicky when resolving dependencies from the `bioconda` channel. A common hurdle I ran into was installing the human genome annotation database, `bioconductor-org.hs.eg.db`.

If you just run `pixi add bioconductor-org.hs.eg.db` from your standard terminal, Pixi might struggle to resolve the R dependencies properly. The reliable fix is to ensure the `bioconda` channel is added to your project, and crucially, **you must enter the `pixi shell` before attempting to add the package**.

```bash
# First, ensure the bioconda channel is available in your workspace
pixi project channel add bioconda

# Drop into the active Pixi shell FIRST
pixi shell

# Now, add the package from inside the shell
pixi add bioconductor-org.hs.eg.db
# this is around 80 MB so should be downloaded within a minute on normal internet speed

```

By adding the package from *inside* the active shell, the resolver has full context of the existing R binaries, and the installation proceeds smoothly.

---

## Cleaning Up Your System (Optional step)

To fully embrace Pixi, you can purge the remnants of your old setup so they don't interfere. This means removing Miniconda and any global R installations that might confuse your IDE.

> _Important: Do this only once you are confident with Pixi and want to completely migrate to it. You can try keeping both for a week and then make a decision whether you think migrating completely makes sense for you. For me it did, so I removed the base-r and conda manager as I no longer needed them._

```bash
# Nuke Miniconda entirely
rm -rf ~/miniconda3
rm -rf ~/.conda ~/.condarc # you can keep the .condarc just in case you want to download it again in future so wouldn't have to reconfigure it again from scratch

# Remove any global R libraries in your user directory to prevent bleeding
rm -rf ~/R # optional step

```

> _Optional: If you are using Jupyter Notebook, there might be some dead paths still lingering around._

To remove the ghost kernels that still haunt your system (the ones pointing to dead Miniconda paths), delete the Jupyter config folders:

```bash
# Remove local user Jupyter kernels
rm -rf ~/.local/share/jupyter/kernels/ir

# If you are using a Remote-SSH server, clear the VS Code remote cache
rm -rf ~/.vscode-server/data/User/globalStorage/ms-toolsai.jupyter/
rm -rf ~/.vscode-server/data/User/workspaceStorage/

```

---

## Setting Up VS Code and `tmux`

Now that your system is clean, you want VS Code to recognize your Pixi-isolated R and Python kernels.

Instead of configuring global user settings, define the R path locally so it changes automatically when you switch projects. Inside your project, create or edit `.vscode/settings.json`:

```bash
mkdir -p .vscode
nano .vscode/settings.json

```

Add these lines to point explicitly to the Pixi binary:

```json
{
  "r.rterm.linux": "${workspaceFolder}/.pixi/envs/default/lib/R/bin/R",
  "r.rpath.linux": "${workspaceFolder}/.pixi/envs/default/lib/R/bin/R"
}

```

### Connecting to a `tmux` kernel

One of the best workflows for heavy data processing is starting your Jupyter server or R session inside a `tmux` session, so it survives network disconnects.

1. SSH into your server, start `tmux new -s project-name`, and navigate to your project.
2. Run your kernel (e.g., `pixi run jupyter notebook --no-browser` or simply start an R script). Copy the link with hash (localhost).
3. In VS Code, open your notebook (`script.ipynb`).
4. Click the **Kernel Selection** button in the top right.
5. Instead of picking a default environment, look for the **Existing Jupyter Server** and add the link that you copied at step 2.
6. From next time onwards, choose the specific active kernel (it will often be labeled `kernel(script.ipynb)` or match your project name like `pixi_r_proj1`).

By selecting the kernel tied to your `tmux` session, you can open multiple scripts in VS Code and connect them all to the exact same memory block. You define a variable in script A, and you can instantly read it in script B, all safely contained within your isolated Pixi environment.

Migration takes a bit of cleanup, but once you experience the speed and cleanliness of a purely project-based environment, you will never look back at global Conda environments again.
