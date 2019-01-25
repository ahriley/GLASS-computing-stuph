# Snakes on a Plane: `conda` explained

As you ascend the ranks of coding, eventually (usually pretty quickly) you will come across the need for **package managers** and **virtual environments**. The difference is subtle, but essentially:
* **Package managers** assist with installation of packages. You tell them "hey, go download this package" and they download that package *and any of its dependencies* (for example, `scipy` requires `numpy` to run). Examples include `pip` (Python-specific), MacPorts (Macs), and Rstudio (for...R...).
* **Virtual environments** help keep dependencies required by different projects separated by creating isolated "environments." For me, these let me use the latest version of `matplotlib` for my research plots and v2.1.1 for when I need Fermi ScienceTools. Examples include `pipenv` and `virtualenv`.

Enter `conda`, which ***does both***. With one tool, I can just say "install these packages into this environment" and `conda` just ***does the thing***. It's the dream for anyone who just wants their software to work out of the box, and it's also extremely useful for when you move over laptops.

**NOTE**: often, package managers offer full-scale *distributions* that have many commonly-used packages in a nice, pre-wrapped parcel that installs all the packages when you install the manager (e.g. Anaconda for `conda`). Personally, I prefer to just have the bare-bones installation (Miniconda for `conda`) and then install whatever packages I need as the need comes up.

## Why `conda`?

Everyone is different, and might have different needs. For example, in the Astropy developer Slack there is a channel named #ihateconda, so obviously there are better programmers than me who disagree with me. However, the big advantages to `conda` over something like `pip+pipenv` or other solutions are:
* **OS agnostic**: there are distributions available for Windows, Mac, and Linux
* **Language agnostic**: there are software packages in Python, C++, R, and most everything else that are delivered by `conda`
* **Used by a lot of astronomers already**: there is an `astropy` channel for Astropy-affiliated packages, `astroconda` for STScI, `fermi` for Fermi, etc.
* **Binary packages**: if you've ever tried to install something from source code and gotten a clang error, you know where I'm coming from
* **Open source**: BSD licensed, you can see the guts on Github. Even though Anaconda was created and is owned by Continuum Analytics, that for-real [isn't really a big deal](https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/#Myth-#7:-conda-is-not-open-source;-it-is-tied-to-a-for-profit-company-who-could-start-charging-for-the-service-whenever-they-want)

## `conda` vs. `pip`?

For a very nice breakdown on this, see ["Conda: Myths and Misconceptions" by Jake VanderPlas](https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/). The basic one-liner is:
* `pip` installs _Python_ packages in _any_ environment
* `conda` installs _any_ package in _conda_ environments

You can take this to mean whatever you want. Basically my workflow is
1. If it can be installed by `conda`, use `conda`
2. If it can be installed by `pip`, use `pip` ***WITHIN THE APPROPRIATE CONDA ENVIRONMENT***
3. Install from source (e.g. Github or tarball) within the appropriate conda environment

## Installation

Ok, so you're sold and now you want to install `conda`.  As alluded to earlier, you can either install the full Anaconda distribution (~3 GB) or the smaller Miniconda distribution (400 MB). My preference is Miniconda, but to each their own.
* [Installation (`conda` documentation)](https://conda.io/projects/conda/en/latest/user-guide/install/index.html)

Once you've followed all the steps, try something like `conda info` to verify that everything worked.

## Quick terminology
Just some vocab specific to conda to keep you on your toes
* **Channel**: where conda is allowed to download packages from. There are lots of channels, including `default`, `conda-forge`, and even `astropy` for Astropy-affiliated packages
* **Downloaded**: exists on your computer
* **Installed**: is set up to work with your current environment
* **Upgraded**: you have an older version in your environment, about to be a new version
* **Downgraded**: you have a newer version in your environment, about to be an older version

## Basic usage

Commands I use regularly:
* `conda search [package]`: comb through your default channels looking for the package (priority in the order of your `.condarc` file)
* `conda install [package]`: install package (and any not-already-fulfilled dependencies)
* `conda update --all`: update everything it can within the environment

Commands I use occasionally:
* `conda update [package]`: update package to most recent version
* `conda create -n [name]`: create a blank environment (still uses whatever is in your `base` until overridden)
* `conda create -n [name] --file [file]`: create environment based on requirements listed in the (text) file
* `conda clean`: clear away any unused packages/downloads (useful if you haven't done it in awhile and need to free up some storage space)

Commands I have used before:
* `conda list --revisions`: list all changes that have been made to an environment
* `conda install --revision [num]`: rewinds environment back to revision [num]

## Useful tips
Just some things I've learned over time
* You don't want to put things in your `base` environment. The only two exceptions I make to this are `git` and `pip` (and their few dependencies), since both are so basic and essential and have versions that come shipped with Macs that I'd rather not touch at all
* Run `conda update conda` before installing/updating anything. It's the recommended flow of things
* You can run `conda clean --all --yes` to remove everything without having to type 'y' a few times
* Most functions (`clean`, `install`, etc.) have a `-d` or `--dry-run` flag to do a "dry-run" (shows you everything that would happen if you ran the command, but doesn't actually do the thing)

## How to learn more?
There is excellent documentation on conda's website, so the places to start would be
* [Getting started on managing](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html#managing-conda)
* [Concepts (describes environment, package, etc.)](https://conda.io/projects/conda/en/latest/user-guide/concepts.html)
* [Cheat sheet for commonly used conda commands](https://conda.io/projects/conda/en/latest/user-guide/cheatsheet.html)

There is also an extensive Stack Overflow and other messaging forums, as well as tutorials online.  Brother Google has your back.

## External resources
* `pip` vs. `conda`: ["Conda: Myths and Misconceptions" by Jake VanderPlas](https://jakevdp.github.io/blog/2016/08/25/conda-myths-and-misconceptions/)
* [`conda` documentation](https://docs.conda.io/projects/conda/en/latest/index.html)
