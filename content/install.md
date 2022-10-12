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

If your distribution is not in the list above, but it does ship Rizin/Cutter,
please let us know and we will fix it! If you cannot find Rizin/Cutter in the
official repositories, we provide install instructions for some other
distributions through [OBS](https://openbuildservice.org/). Follow the
instructions
[here](https://software.opensuse.org/download/package?package=rizin&project=home%3ARizinOrg).

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

Rizin can be installed through [Homebrew](https://brew.sh/)
```
$ brew install rizin
```

Cutter can be installed through the .dmg file provided in the [latest release](https://github.com/rizinorg/cutter/releases/latest).

## OpenBSD

Rizin and Cutter are available in stable releases starting with OpenBSD-7.3.

```
# pkg_add rizin cutter
```

## Building from source

Source code for Rizin and Cutter can be downloaded from Github:

 - [Rizin repository](https://github.com/rizinorg/rizin/)
 - [Cutter repository](https://github.com/rizinorg/cutter/)

Build instructions can be found in the `README.md` files.

---
