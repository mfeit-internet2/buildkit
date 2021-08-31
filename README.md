# Unibuild

Unibuild is a kit for building repositories of software packaged in
RPM or Debian format.  It consists of two parts:

 * Unibuild-Package, which automatically builds packaged software
   based on suppled OS-specfic packaging information (RPM
   specifications or Debian packaging data).

 * Unibuild, which oversees construction of multiple packages using
   Unibuild-Package and combines the finished products into a
   repository suitable for the system where it was invoked.



# Installation

## Prerequisites

 * A POSIX-compliant shell
 * POSIX-compliant command-line utilities
 * GNU Make
 * GNU M4
 * Sudo
 * On RPM-based Systems:
   * The Bourne Again Shell (BASH)
   * RPM
   * RPMBuild
 * On Debian-based Systems:
   * Apt
   * DPkg


Builds done by a user other than `root` (recommended) require that the
user can acquire superuser privileges using `sudo`.  For unattended
builds, this access must require no human interaction.


# Build and Install

In the directory containing this file, run `make`.  Unibuild will
install any dependencies, self-build and install.  A repository
containing all packages will be placed in the `unibuild-repo`
directory.

Short summary:

```
$ git clone https://github.com/perfsonar/unibuild.git
$ cd unibuild
$ make
```



# Using Unibuild

Building a repository with Unibuild requires establishing a directory
where all of the sources live and a file the determines what packages
get built and in what order.  For example:

```
some-unibuild-project/
    package1/
    package2/
    ...
    packageN/
    unibuild-order
```

## The `unibuild-order` File

The order in which the packages are built is determined by the
contents of `unibuild-order`.  This is a flat file containing a list
of package names to be built.  Each package is found in a same-named
directory (e.g., the package `foo` would be located in the directory
`./foo`).

To allow intelligent decision making about what packages to build on
what platforms, `unibuild-order` is processed with the [GNU M4 Macro
Processor](https://www.gnu.org/software/m4).  To assist in this
process, Unibuild makes the following macros available:

| Macro | Description | Example |
|-------|-------------|---------|
| `OS` | The operating system as reported by `uname(1)` | `Linux` |
| `DISTRO` | The name of the operating system distribution | `CentOS` |
| `FAMILY` | The family to which the operating system belongs.  For systems where this does not apply (e.g., `Darwin`), this will be empty. | RedHat |
| `RELEASE` | The release of the operating system. | `7.9.2009` |
| `MAJOR` | The major version of `RELEASE`. | `7` |
| `MINOR` | The minor version of `RELEASE`. | `9` |
| `PACKAGING` | The type of packaging used by this system.  Currently-provided values are `deb` and `rpm`. | `rpm` |
| `OS` | The system architecture as reported by `uname(1)` | `x86_64` |

The list of provided macros and their values can be displayed by
running `unibuild macros`.

While all of M4's processing features are available, most decisions
can be made using the [if-else or multibranch
construct](https://www.gnu.org/software/m4/manual/html_node/Ifelse.html#Ifelse)
and the [integer expression
evaluator](https://www.gnu.org/software/m4/manual/html_node/Eval.html#Eval).
For example:

```
# Built everywhere, unconditionally.
foo

# Built only on RPM systems:
ifelse(PACKAGING,rpm,bar)

# Built everywhere other than Debian-derived systems on Intel:
ifelse(FAMILY/ARCH,deb/x86_64,,baz-o-matic)

# Built only on versions of Ubuntu prior to 20:
ifelse(DISTRO/eval(MAJOR < 20),Ubuntu/1,quux)

# Built everywhere except on Debian 9 or anything Debian on ARM 64 or
# PowerPC 64.
ifelse(DISTRO/MAJOR,Debian/9,,
       FAMILY/ARCH,Debian/arm64,,
       FAMILY/ARCH,Debian/ppc64el,,
       xyzzy)
```


## Unibuild Commands

Unibuild is invoked by running `unibuild` on the command line with a
command and optional arguments.  IIf no command is provided, the
default command will be `build`.

These are the available commands:


| Command | Description |
|---------|:-----------:|
| `build` | Builds all packages (equivalent to `unibuild make clean build install`) and gathers the results into a repository (equivalent to `unibuild gather`). ||
| `make` | Runs `make` against targets in each package directory. |
| `gather` | Gathers the products of building each package into a repository. |
| `macros` | Displays the macros available for use in `unibuild-order` files and their values on this system. |
| `order` | Processes the `unibuild-order` file and displays the results. |


The `--help` switch may be used with all commands for further
information on invoking them.




## Preparing Individual Packages

As noted above, each package to be built as part of a repository lives
in a directory.

### Tarball

```
foomatic/
    Makefile
    foomatic-1.23.tar.gz
    unibuild-packaging/
        deb
        rpm
```

The Makefile

```
include unibuild/unibuild.make
```



### Raw Sources

```
foomatic/
    Makefile
    foomatic/
        ...foomatic sources...
        unibuild-packaging/
            deb
            rpm
```
Makefile:
```
AUTO_TARBALL := 1
include unibuild/unibuild.make
```




## Building Repositories with Unibuild

```
TOP
 |
 +-- unibuild-order
 |
 +-- package1
 +-- package2
 +-- ...
 +-- packageN
```



TODO: See ...Unibuild... for details
TODO: See ...Unibuild-Package... for details
TODO: Hello world example
