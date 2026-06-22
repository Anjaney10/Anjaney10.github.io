---
layout: post
title: "Version Control with Git/GitHub"
date: 2026-06-21 09:00:00 +0000
categories: [Tutorials]
tags: [bioinformatics, git, github, version control]
author: Anjaney
---

If you have ever tried to set up Git and GitHub for the first time, you have probably run into a wall of cryptic errors. I certainly have. You tell your IDE to track your files, you click "Publish," and suddenly you are bombarded with upstream tracking errors, non-fast-forward rejections, and email privacy blocks.

As bioinformaticians, we deal with enough complexity in our data pipelines; saving our code shouldn't require a Ph.D. in computer science.

In this guide, I will break down exactly how Git and GitHub communicate, share my strict rules for version-controlling bioinformatics projects, and provide step-by-step setups tailored to your specific coding environment—whether you use VS Code, RStudio, a pure Bash terminal, or Jupyter Notebooks.

## The Mental Model

The biggest hurdle in learning version control is understanding that tracking a file locally does not automatically upload it to GitHub. Git operates in distinct phases:

1. **The Local Time Machine (`.git`):** When you initialize Git in a folder, it creates a hidden `.git` database. It does not auto-save every keystroke. Instead, you have to tell it to take a "snapshot" of your project at a specific moment. This is called a **Commit**. Your local Git works 100% offline.
2. **The Cloud Backup (GitHub):** GitHub is just a server running the same Git software. To get your local snapshots onto GitHub, you have to explicitly **Push** them.

---

## Golden Rules for Bioinformatics

Before we touch any commands, here are the non-negotiable rules I follow to keep my repositories clean and reproducible:

* **Never commit raw data:** GitHub has strict file size limits and is meant for code, not data. Create a `.gitignore` file in your main folder and add extensions like `*.bam`, `*.vcf`, `*.fastq`, and `*.csv`.
* **Track your environment, not just your code:** Code rot is real. A script that runs today might break next year due to package updates. I use Conda, so I always export my environment and commit it alongside my scripts:

```bash
conda env export --no-builds > environment.yml
```

* **Commit logically:** Commit when you finish a specific task (e.g., `Add data parsing function`), not just at the end of the day.

---

## The Setup Guides

How you implement Git depends entirely on where you write your code. Find your primary environment below.

### Scenario A: Visual Studio Code

VS Code has a fantastic visual Git interface, but linking an existing GitHub repository for the first time usually triggers a cascade of errors. Here is how I set it up flawlessly.

**1. The Initial Link & First Push**
If you created a repository on GitHub first and want to push your local VS Code files to it, open your integrated terminal and run:

```bash
git remote add origin https://github.com/yourusername/your-repo-name.git
git branch -M main
git push -u origin main
```

**2. Handling the "Non-Fast-Forward" Error**
If GitHub rejects your push because the repository already has a `README.md` or `License` file that your computer doesn't have, force the local code to overwrite the remote:

```bash
git push -u origin main --force
```

**3. Handling the "GH007: Private Email" Error**
If GitHub blocks your push to protect your email privacy, get your anonymous GitHub email from your settings (`12345678+username@users.noreply.github.com`) and update your Git config:

```bash
git config --global user.email "your-anonymous-email@users.noreply.github.com"
git commit --amend --reset-author --no-edit
git push origin main --force  
```

**4. The Daily VS Code Routine**
Once the setup is done, I don't have to touch the terminal for Git again. At the end of the day:

1. Open the **Source Control** panel.
2. Click the **`+`** to stage changed files.
3. Type a message and click **Commit**.
4. Click the blue **Sync Changes** button.

*(Pro tip: Allow VS Code to periodically run `git fetch`. It safely checks GitHub for updates in the background without altering your local files.)*

### Scenario B: The Bash Terminal

If you are working via SSH on an HPC cluster, you won't have a graphical interface. You have to rely purely on Bash commands.

**1. The Initial Setup**
Navigate to your project directory and run:

```bash
git init
git add .
git commit -m "Initial commit of bioinformatics pipeline"
git branch -M main
git remote add origin https://github.com/yourusername/your-repo-name.git
git push -u origin main  
```

**2. The Daily Terminal Routine**
After a day of coding, my sync routine looks like this:

```bash
git add script_name.py environment.yml
git commit -m "Update variant calling parameters"
git push origin main  
```

### Scenario C: RStudio

RStudio has Git integration baked directly into its GUI, which is perfect for `ggplot2` and `dplyr` workflows.

**1. The Initial Setup**

* Go to **Tools** > **Global Options** > **Git/SVN**. Ensure the path to your Git executable is set.
* To start a new project, go to **File** > **New Project** > **Version Control** > **Git**.
* Paste your GitHub repository URL. RStudio will clone it and set up the working directory automatically.

**2. The Daily RStudio Routine**

* Look at the top right pane in RStudio; you will see a **Git** tab.
* Check the boxes next to the `.R` scripts you modified (this is Staging).
* Click **Commit**, type your message in the pop-up window, and save.
* Click the **Push** (Up Arrow) button in the Git pane to send it to GitHub.

### Scenario D: Jupyter Notebooks

Jupyter Notebooks (`.ipynb` files) are notorious in the Git world. Under the hood, they are messy JSON files that store not just your Python/R code, but also the visual outputs, plots, and execution counts. If you commit a notebook with its outputs, your GitHub history becomes an unreadable mess.

**1. The Best Practice Setup**
I treat Jupyter Notebooks as temporary scratchpads. If I absolutely must version control a notebook, I always clear the outputs before committing.

* In Jupyter, click **Cell** > **All Output** > **Clear**.
* Save the notebook.

**2. The Sync Routine**
Jupyter itself does not have native, robust Git tools built into the default interface. I always run a Bash terminal window alongside my Jupyter server, or I open the folder in VS Code, and use the terminal/VS Code routines mentioned in Scenarios A and B to stage and push the cleaned `.ipynb` files.

---

Version control has a steep learning curve, but once that initial pipeline is connected, it becomes invisible. Set up your `.gitignore`, lock your Conda environments, and stick to the daily staging and syncing routine. Your future self will thank you.
