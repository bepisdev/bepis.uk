---
author: Josh Burns
pubDatetime: 2024-05-11
modDatetime: 2024-05-11
title: Rewriting a Python project in Go
slug: rewriting-python-project-in-go
featured: true
draft: false
tags:
  - git
  - python
  - go
  - software-development
description: A rundown on how and why I Rewrote my gitignore generator in Go from its origin python.
---

# Rewriting Software from Scratch: A Journey in a New Language

## Introduction

A few years ago I wrote a Git plugin called `git-ignores`. It was a dead simple automation tool
to generate a `.gitignore` file using Github's Gitignore templates. I chose to write the
plugin in Python as it was an off-the-cuff idea that was more about automating a
process for myself when starting a new project than anything else. Once I'd finished it,
I never really went back to maintain anything and the script _"just worked"_.

Cut to a few years later, and I have no current projects to work on, and I am creatively
burnt out, on 2 weeks' leave from work, sitting in front of my PC bored out of my mind. Scrolling
through my GitHub page and I see it sitting there. I thought to myself that could be better, and
at that moment I decided I would translate that script into a new language. Both as a
well-needed update to an otherwise stale piece of software, and a challenge to see how well I
not only understood both the original and new language choice, but also my skills of interpretation
between the two and as a result my understanding of certain software development concepts in general.

## Understanding the Original Software

As stated above, the original `git-ignores` plugin was simply a Python script, that would create a `.gitignore` file for a particular
language or framework based on User input. To accomplish this, the user would supply a string using `-t [STRING]` this string would match
the name of a file in [this repo](https://github.com/github/gitignore/). For example, if a user runs `git ignores -t Python` it would (internally)
generate the URL of `https://github.com/github/gitignore/blob/main/Python.gitignore`. A HTTP GET is then used to fetch the contents of this file
and in turn, is then written to a `.gitignore` in the current working directory. There was also a safety built in that it would not overwrite the existing
`.gitignore` unless `--force` flag was present.

## Choosing the New Language

When beginning the process of choosing the language to translate my plugin into, I had to fill in 2 specific criteria.

1. A compiled, statically typed and self-contained binary as a final product.
2. First-party testing, no external dependencies for writing tests.

It came down to 2 options, Rust or Go. Rust is a language that I'm heavily interested in, and have played around with. I wasn't too confident in
my abilities with Rust to confidently translate a Python program into a Rust project with any degree of decency. While it would make a great opportunity
to learn Rust properly and at a deeper level, I felt like it would be more appropriate to choose something that I already understood and something I
could debug and maintain more consistently as this is a piece of software I do use on a more or less consistent basis.

So for both my sanity and ease of maintainability, I chose Go.

## Planning the Rewrite

I started like I would any other project. Identifying the requirements of the project and things I would need to pay attention to, to maximize my ability
to side-step any obstacles before it was too late and I would have to build my way over them. As this was a very simple project with a very simple implementation
it would be easy to avoid any pitfalls.

I knew that it would be a CLI program (it's a git plugin after all). It takes some user input, runs a HTTP request, and performs a file write.
When writing CLI programs in Go, the go-to solution for the longest time for most people has been [cobra](https://github.com/spf13/cobra). Cobra is
a fully-fledged framework for building CLI apps and handles everything from flags and options to subcommands and configs, I felt like Cobra was too
much for what I needed, What I opted for was a library called [pflag](https://github.com/spf13/pflag) which is the underlying library (written by the same developer) powering Cobra. `pflag` extends Go's inbuilt `flag` package and is written as a _drop-in_ replacement, the main difference is
The inbuilt package only allows you to bind a single flag definition to a single value or option it also defaults to a single dash style, and doesn't allow for
shortcuts (`-flag` instead of POSIX style `--flag` with `-f` as a shortcut).

`pflag` was the only external dependency I would need. Everything else Go provides in the standard library would suffice.

## Implementation Phase

I broke down the development into 3 identifiable phases so I could track my progress and overall success in the mission.

- **Phase 1 (Business logic)** - Successfully translate the process of generating the HTTP GET request.
- **Phase 2 (File Handing)** - Successfully check if a file exists, and if not, write the HTTP response to that file path.
- **Phase 3 (Behavior Modification)** - Use flags to modify how the program deals with the result of an if statement.

I started with a blank `main.go` file and the original Python source code open on my second monitor. Line by Line identifying
how a given block of code would work in the new context and translating it over bit by bit. The overall translation was honestly
seamless and I didn't run into any real roadblocks. Go's focus on explicit error handling made the file handling process straightforward
and implementing the `--force` flag to allow for overwrites was boiled down to 2 lines of code.

## Testing and Quality Assurance

Go has extremely powerful built-in tooling for testing as well as HTTP stuff. The combination of the two enabled
a very simple test suit to be written against the program's core functionality. Simply writing a list of test cases that included
a target which simulated the user input (i.e `Python` which targets `Python.gitignore` or `DoesNotExist` which would simulate bad input)
and an expectation for the HTTP status of the request. we iterate over this list and boom, the test suit is complete.

## Conclusion

I now have a static binary called `git-ignores` sitting in a `/usr/share/bin/` that has no dependency on an installed runtime or package
environment, unlike its Python predecessor. It's simple and it works. The project is available on my GitHub [here](https://github.com/joshburnsxyz/git-ignores).
