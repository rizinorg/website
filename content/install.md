+++
"author" = "Rizin"
"title" = "Install"
"date" = "2022-08-17"
"summary" = "How to install Rizin and Cutter"
"tags" = []

"ShowToc" = false
"TocOpen" = false
"weight" = 2
+++

## Linux

If your distribution ships Rizin from the official repositories, use that. We
are currently aware of the following Linux distributions shipping an up-to-date Rizin:

- Arch Linux
- Fedora
- Gentoo

If your distribution is not in the list above, but it does ship Rizin/Cutter,
please let us know and we will fix it! If you cannot find Rizin/Cutter in the
official repositories, we provide install instructions for some other
distributions through OBS. Follow the instructions
[here](https://software.opensuse.org/download/package?package=rizin&project=home%3ARizinOrg)
(select the *"Add repository and install manually"* option).

## Windows

You can install Rizin through the installer for your architecture provided in
the [latest release](https://github.com/rizinorg/rizin/releases/latest) (e.g.
`rizin_installer-vX.Y.Z-x86_64.exe`).

Otherwise, you can download the portable builds that can be run without any
installation on your system, by just extracting the archives in the path you
want and executing Rizin from there (e.g. `rizin-windows-share64-vX.Y.Z.zip`).

You find Cutter for Windows in the [latest Cutter
release](https://github.com/rizinorg/cutter/releases/latest). The archive can be
extracted anywhere on your system and Cutter can be executed from there.

## MacOS

You can install both Rizin and Cutter through [Homebrew](https://brew.sh/)
```
$ brew install rizin
$ brew install --cask cutter
```

Alternatively, you can find Pkg/DMG files for both
[Rizin](https://github.com/rizinorg/rizin/releases/latest) and
[Cutter](https://github.com/rizinorg/cutter/releases/latest).

## OpenBSD

Rizin and Cutter are available in stable releases starting with OpenBSD-7.3.

```
# pkg_add rizin cutter
```

## Android

Statically compiled binaries for some common architectures where Android runs
are compiled and attached to all releases. We currently support aarch64, armv7,
and x86_64. You can find the artifacts for Android on the [latest Rizin
release](https://github.com/rizinorg/rizin/releases/latest).

Those files are named as `rizin-<version>-android-<architecture>.tar.gz`. Files
within the archive can be extracted anywhere on your Android device because
Rizin is compiled in a "portable" way, allowing moving the whole directory
anywhere.

We do not currently have a package on Termux, but if you would like to add one
we will be happy to help you with any issue you may encounter.

## Building from source

Source code for Rizin and Cutter can be downloaded from Github:

 - [Rizin repository](https://github.com/rizinorg/rizin/)
 - [Cutter repository](https://github.com/rizinorg/cutter/)

Build instructions can be found in the `README.md` files.

---

# Install Rizin plugins

To install Rizin plugins you can use our package manager,
[rz-pm](https://github.com/rizinorg/rz-pm), that will compile and install
packages for you in the right locations.

Get the latest version for your system
[here](https://github.com/rizinorg/rz-pm/releases/latest), make the file
executable and you are good to go!

The list of currently supported plugins is available in the [rz-pm-db
repository](https://github.com/rizinorg/rz-pm-db).
