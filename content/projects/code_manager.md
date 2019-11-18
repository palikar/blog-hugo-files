+++
title = "Code Manager [PART 1]"
author = ["Stanislav Arnaudov"]
description = "C++ implementation of a graph that is fully usable at compile time"
date = 2019-11-16T00:00:00+01:00
keywords = ["c++", "programming", "compile-time", "constexpr"]
lastmod = 2019-11-18T13:32:41+01:00
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

This is my personal tool now for managing my GitHub repositories, some system software that I use and pretty much everything that can be downloaded, compiled locally and then installed on a Debian based Linux system. Through this utility one can quickly download and install random things from all over the internet. I've always wanted some small program that would allow me to quickly bring my GitHub repositories on my local machine so I end it up writing this in my spare time. The program is focused on automation but also flexibility in the installation process. A lot of software is compiled and installed in some standard way but some things are a little bit trickier. The utility -- named appropriately `code_manager` -- aims to provide a unified interface for the installation process of all types of software, the trickier kind included.

<br />

In a way, I see `code_manager` as a substitute for an [AUR Helper](https://wiki.archlinux.org/index.php/AUR%5Fhelpers) on Arch-based Linux systems. My idea is, however, not to have a dedicated repository with packages like the [AUR](https://wiki.archlinux.org/index.php/Arch%5FUser%5FRepository) but rather have the "entire internet" as a source for packages. If I install something once, I can quickly update some of the configuration files for the `code_manger` and have that thing automatically installable for the future. My final goal for the project is to be able to download the utility on a fresh Debian System, execute a couple of commands on the console and then let `code_manger` do the magic of pulling everything, compiling the necessary packages and then installing what is needed to be installed. The product should then be ready to use Linux environment.

<br />

For more information as well as the source code, check out the [GitHub page](https://github.com/palikar/code%5Fmanager) of the project.


## Usage {#usage}

When running `code_manger` for the first time, several configuration files should be copied at the right place inside of the `${HOME}/.config` folder. This should preferably be done by executing:

```sh
./code_manager.py --setup-only
```

Upon running this, `code_manager` will setup everything needed on the system. The user will also be prompted for several things. Those are:

-   CODE directory - this is the location where the source code of the packages will be fetched. Each package will have its own directory within the code directory.
-   USR directory - this is meant to be a user local equivalent of a directory like `/usr`. Every package that requires some sort of installation will be installed there. `USR/bin` will be automatically added to the `${PATH}` and `USR/lib` to `$LD_LIBRARY_PATH`.
-   Virtual environment directory - `code_manager` will create its own virtual environment in this directory. All python packages that will be installed through `code_manger` will in the newly created environment.
-   Python executable - it will be used to create the virtual environment. This will define which version of python will be used.

`code_manager` will also create the shell file `CODE/setenv.sh`. This file must be sourced before using `code_manager`. It will activate the virtual environment and will export the needed environment variables.

<br />

Once the `code_manager` is set up, you can start using it immediately. The file that defines what kind of packages can be installed is `${HOME}/.config/package.json`. For more information on what exactly is defined in the file and how, see the [guide](https://github.com/palikar/code%5Fmanager#packagesjson) on GitHub or my personal [packages.json](https://github.com/palikar/dotfiles/blob/master/.config/code%5Fmanager/packages.json).

<br />

To install my CTGraph project, for example, all I have to execute is:

```sh
code_manager install ctgraph
```

In my packages.json, ctgraph is defined as:

```json
"ctgraph": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/ctgraph"
    },
    "install": ["cmake", "make"],
    "cmake_args": "",
    "make_extra_targets" : ["install"],
    "make_args": "-j3"
}
```

This means that the package will be fetched with git from <https://github.com/palikar/ctgraph/>. `@arnaud_base` is a variable and its value is my github account. When fetched, the project will be built first with [cmake](https://cmake.org/) and then with [make](https://www.gnu.org/software/make/). Make will be run with `-j3` and also the `make install` target will be executed.


## Installers {#installers}

`code_manger` has a decent collection of installers that can be run automatically. With "installer" I mean a separate utility or system that can be executed and it will compile, build and/ or install the given package. In the long term, I plan to add a lot more installers and make `code_manger` as versatile as possible. Every installer is used a little bit differently. This means that the package node in the packages.json file has to have certain properties. I'll briefly go over the current installers here. You can find an example `packages.json` file [here](https://raw.githubusercontent.com/palikar/dotfiles/master/.config/code%5Fmanager/packages.json) where most of the installers are used and it should be self-explanatory.


### autoreconf {#autoreconf}

For configuring projects that use [autoreconf tools](https://wiki.debian.org/Autoreconf).

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["autoreconf"],
    "autoreconf_args": ""
}
```


### bashrc {#bashrc}

For adding things in the current user's `${HOME}/.bashrc` file. Alternatively, shell executable code can be inserted in the `CODE/setenv.sh` file. Only valid lines will be added

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["bashrc"],
    "in_bashrc" : true,
    "bash_lines": [
        "this is invalid line",
        "export VALID_VAR=123"
    ]
}
```


### cmake {#cmake}

For running cmake.

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["cmake"],
    "cmake_args": "DBUILD_TESTS=1"
}
```


### command {#command}

For executing a custom shell command inside of the package's directory

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["command"],
    "command" : "echo this is command"
}
```


### cp {#cp}

For copying arbitrary files to arbitrary locations.

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["cp"],
    "cp" : [
        {
            "source" : "example.txt",
            "dest" : "${HOME}/.config/example"
        }
    ]
}

```


### emacs {#emacs}

For adding files to he [Emacs](https://www.gnu.org/software/emacs/) [init file](https://www.gnu.org/software/emacs/manual/html%5Fnode/emacs/Init-File.html).

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": "emacs",
    "el_files" : ["example.el"]
}
```


### make {#make}

For running targets of a Makefile.

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["make"],
    "make_extra_targets" : ["install"],
    "make_args": "-j3"
}
```


### pip {#pip}

For installing packages through [pipy](https://www.w3schools.com/python/python%5Fpip.asp). It will also install the packages in `requirements.txt` if the file is present inside the package's directory.

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["pip"],
    "pip_packages" : ["python-package-example"]
}
```


### script {#script}

For running custom script to build and install a package. For more information see [here](https://github.com/palikar/code%5Fmanager#installation-scripts).

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": ["script"],
    "script": "install_example.sh",
    "script_args": "--example",
}
```


### setup\_py {#setup-py}

For installing python projects with `setup.py`.

<br />

Example:

```json
"example": {
    "fetch": "git",
    "git":{
        "url" :  "@arnaud_base/example"
    },
    "install": {"setup.py"},
    "setup_args" : ["--optimize=1", "--record=install_log.txt"],
}
```
