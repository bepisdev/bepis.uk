---
author: Josh Burns
pubDatetime: 2022-11-25
modDatetime: 2022-11-25
title: Build your own Git plugin
slug: build-your-own-git-plugin
featured: true
draft: false
tags:
  - tutorial
  - git
  - python
description:
  A simple tutorial on building your own git plugin using Python
---

## Introduction

In this article I will run you the reader, through building your very own plugin for
everyones favourite, already bloated, VCS solution, the one and only Git.

Plugins are a way to extend / automate git related tasks and run them as a part of the git
tool suite.

We will start with an overview of the git plugin system and what a git plugin looks like. Before
building out a quick example that you can install and use on your machine immediately. 

### How Git Plugins Works?

Git plugins are simply executables that exist on your `$PATH` that follow the naming convention
`git-PLUGIN`. The git commands you're fammiliar with are themselves technically plugins as the `git`
command itself is just picking up the `git-push` and `git-clone` binaries and loading them as plugins.

### What We Will Build

We will build out a script that will scan a given directory for git repos and then for each repo run
`git pull`, `git push`, and `git push --tags` all in sequence. We will also build an ad-hoc installation
script so the script is correctly permissioned and named on the system.

I will use Python as my language of choice to write the plugin. I will also use the [click](https://click.palletsprojects.com) library to
handle CLI argument parsing.

## The Implementation

Firstly lets start by setting up our work environment. Create a directory to act as a workspace, then we will
go ahead and create a virtualenv inside of that directory.

```
~/code $ mkdir my-git-plugin
~/code $ cd my-git-plugin
~/my-git-plugin $ python -m virtualenv ./venv 
~/my-git-plugin $ source ./venv/bin/activate
~/my-git-plugin (venv) $ 
```

Now we have our virtualenv setup we can start writing some code. Create a new file named `git-myplugin.py` (You can name it whatever you want as long as it starts with `git-` and ends with `.py` for the sake of simplicity i am going with `myplugin`) The `myplugin` part of the filename is what will become our sub command when we call it through git.

```
~/my-git-plugin (venv) $ touch ./git-myplugin.py
```

We will need to install the `click` package, after that we will use `pip freeze`
to create our `requirements.txt` file.

```
~/my-git-plugin (venv) $ pip install click
~/my-git-plugin (venv) $ pip freeze > requirements.txt
```

Open `git-myplugin.py` and add the following code.

```python
#!/usr/bin/env python3

import click

@click.command()
def run():
  print("Hello")

if __name__ == '__main__':
  run()
```

Here we have initialized a very basic click CLI app. Click is a framework that provides some very nice features to build out CLIs using decorators.
We have simply defined a function that prints _"hello"_ to the console. This function is called when the script is executed from the console, which in-turn
exposes our CLI to the user. This is the foundation of our app, let's get cracking...

Next thing is to write a function that can scan a given directory for git repos. This is a pretty straight forward task, simply take a path, walk through it, check each step if
a `.git` exists, and then return a boolean value. For the sake of this demonstration I will call this function `_walk_dir()`. It takes one param which is the dir path itself. The `_`
prefixing the function name is simply an idiom I use to identify helpers and utility functions, it isn't important.

```python
#... Rest of file omitted

import os

def _walk_dir(dir):
  """Walk through dir and return true if 
  we find a directory labelled .git, otherwis
  return false"""
  for filename in os.listdir(dir):
    f = os.path.join(dir, filename)
    if os.path.isdir(dir) == True:
      if filename.basename == ".git"
        return true
  return false

#...
```

Now we need a way to supply `_walk_dir()` with the `dir` parameter
it requires to run. To do this we will get the user to supply a `-d`
or `--dir` switch to the CLI with that argument. To accomplish this,
we will add a click option decorator to the `run()` function and then a
parameter to the function matching the switch.

```python
#... Rest of file omitted

@click.command()
@click.option('-d', '--dir', default=".", type=str, help="Target path to scan")
def run(dir):
  print("Hello")

#...
```

Now when we run our script we can use the `-d` option to tell the script where to scan, if we dont it will use
the path we defined in the `default=` param, we have set it to the current directory for the sake of keeping it
simple.

The script doesn't do anything yet, but here is what it would look like to call the script as it stands.

```
~/my-git-plugin (venv) $ python3 ./git-myplugin.py -d ~/Code
```

Now we will wire up our `_walk_dir()` helper to our `run()` function like so. We will create
a variable named `is_git` and set it equal to the result of `_walk_dir(dir)`. This will be used
in a conditional statement to determine if the script should then fire off another helper (`_run_git_commands()`)
that we will create later on.

```python
#... Rest of file omitted

@click.command()
@click.option('-d', '--dir', default=".", type=str, help="Target path to scan")
def run(dir):
  is_git = _walk_dir(dir)
  if is_git is True:
    _run_git_commands(dir)
#...
```

Now we will quickly build out the `_run_git_commands()` helper as well.

```python
#... Rest of file omitted

import subprocess

def _run_git_commands(repo_dir):
  """Runs the following commands in order
  on the repo given"""
  subprocess.run(["git", "pull"])
  subprocess.run(["git", "push"])
  subprocess.run(["git", "push", "--tags"])

#...
```

At this current point we have a fully functional script that does the following:

- Takes a directory path via a CLI option.
- Scans that directory to determine if its a Git repo or not
- If it is a Git repo then run git pull, and then push all changes and 
tags up to the remote.

The only thing left to do is make sure our installed Git will pick up our script
and register it as a command so instead of running the script the old fashioned way
we can call it as a git subcommand like illustrated earlier.

To accomplish this, the script will have to be executeable, exist on `$PATH`, and follow
the naming convention without a file-extension. Here are the commands step-by-step to 
get the script installed.

> 1. Create a copy with correct name
```
~/my-git-plugin (venv) $ cp git-myplugin.py git-myplugin
```

> 2. Set execution permissions correctly
```
~/my-git-plugin (venv) $ chmod a+x ./git-myplugin
```

> 3. Move the new executable to a directory on `$PATH` for this example I will use `/usr/bin`
```
~/my-git-plugin (venv) $ sudo mv ./git-myplugin /usr/bin/git-myplugin
```

We can now run our script with the command `git myplugin` or `git myplugin -d ~/Code`.

Here is the full code if you're interested...

```python
#!/usr/bin/env python3

import click
import os
import subprocess

def _run_git_commands(repo_dir):
  """Runs the following commands in order
  on the repo given"""
  subprocess.run(["git", "pull"])
  subprocess.run(["git", "push"])
  subprocess.run(["git", "push", "--tags"])

def _walk_dir(dir):
  """Walk through dir and return true if 
  we find a directory labelled .git, otherwis
  return false"""
  for filename in os.listdir(dir):
    f = os.path.join(dir, filename)
    if os.path.isdir(dir) == True:
      if filename.basename == ".git"
        return true
  return false

@click.command()
@click.option('-d', '--dir', default=".", type=str, help="Target path to scan")
def run(dir):
  is_git = _walk_dir(dir)
  if is_git is True:
    _run_git_commands(dir)

if __name__ == '__main__':
  run()
```