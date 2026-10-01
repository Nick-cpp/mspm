# mspm

A simple, lightweight package manager.

---


## Installation

Installation into current system:

```
# make install
```

Uninstallation:

```
# make uninstall
```

Installation into a rootfs ( e. g. /mnt ):

```
# make install DESTDIR=/mnt
```

Uninstallation:

```
# make uninstall DESTDIR=/mnt
```

---

## Usage

### 1. Synchronize Repositories

Fetch and update local repository trees defined in `/etc/mspm/repos.conf`:

```
# mspm sync
```

### 2. Installing Packages

Install one or multiple packages:

```
# mspm install <package1> <package2> ...
```

Explicitly target a specific repository using `pkg::repo` syntax:

```
# mspm install cmake-bin::master fastfetch
```

### 3. Removing Packages

Remove installed packages from the system:

```
# mspm remove <package1> <package2> ...
```

Target a specific repository entry for removal:

```
# mspm remove fastfetch::master
```
### 4. Upgrading installed packages: 

```
# mspm update
```

---

## Configuration

* `/etc/mspm/repos.conf` — Defines repositories and sync commands.
* `/etc/mspm/make.conf` — Configures environment variables (e.g., `MAKEFLAGS="${MAKEFLAGS} -j16"`).
* `/etc/mspm/installed` — Plain-text database tracking installed package specifications.

### make.conf example

```
CFLAGS="-march=alderlake -O2 -pipe"
CXXFLAGS="-march=alderlake -O2 -pipe"
MAKEFLAGS="${MAKEFLAGS} -j12"
```

---

## Adding custom repositories to `mspm`

To make repositories available to `mspm`, add an entry to `/etc/mspm/repos.conf` using the format `<repo_name> = <sync_command>`

Adding `testing` - **unstable** repository to mspm:

```
testing=wget -q "https://github.com/Nick-cpp/mspm-test-repo/archive/refs/tags/testing.tar.gz" -O - | tar -xz --strip-components=1
```

**Testing repository is an unstable mspm repository, packages from it may not be built, some dependencies for packages in the testing repository may be missing, if you use the testing repository and notice errors - report it to my email - mikola@atomicmail.io**

# Creating a Repository for mspm

A guide on how to structure, create, and maintain your own package repository for **mspm**.

---

## Repository Structure

An `mspm` repository is a simple directory hierarchy. Each package lives in its own subdirectory and must contain a build script named `mspm-build`.

```
my-repo/
├── fastfetch/
│   └── mspm-build
├── htop/
│   └── mspm-build
└── cmake-bin/
    └── mspm-build
```

---

## Requirements for `mspm-build`

By default, mspm-build executes inside /etc/mspm/cache/<repository_name>/<package_name>. Do not change this working directory; perform all build operations directly within it.

Every `mspm-build` script is sourced as a Bash script and must adhere to the following rules:

### 1. File Naming
* The recipe file **must** be named exactly `mspm-build`.

### 2. Dependencies (`depends`)
* Dependencies are defined using a standard Bash array: `depends=("pkg1" "pkg2::repo")`. Use `depends=("pkg1||pkg2")` to require one of pkgs
* If there are no dependencies, leave the array empty or omit it entirely.

### 3. Conflicts
* If your package may conflict with other packages use `conflicts=("pkg1" "pkg2")` to mark packages as conflict

### 4. Version
* The version is used for checking for the updates you should define the version in your mspm-build. Use `version="1.2.3`"

### 5. The `install()` Function
* Must be defined in the script.
* Handles downloading, compiling, and copying binaries/files into the system (`/usr`, `/etc`, etc.).

### 6. The `remove()` Function
* Must be defined in the script.
* Cleanly removes all files installed by the package.

---

## Example `mspm-build` Script

```
# Package recipe for fastfetch

#!/bin/bash

version="2.68.1"
conflicts=("fastfetch-bin")
depends=("cmake-bin||cmake")

install() {
    rm -f $version.tar.gz
    wget https://github.com/fastfetch-cli/fastfetch/archive/refs/tags/$version.tar.gz
    rm -rf fastfetch/
    mkdir -p fastfetch/
    tar -xf $version.tar.gz -C fastfetch --strip-components=1
    cd fastfetch/
    ./run.sh
    cp build/fastfetch /usr/bin/fastfetch
}

remove() {
    rm -f /usr/bin/fastfetch
}

```

Thanks to [LINUXHUNTERREDHAT](https://github.com/LINUXHUNTERREDHAT) for providing the Makefile.
