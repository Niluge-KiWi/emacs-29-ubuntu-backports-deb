Got a need for a recent emacs 29.1?
But not emacs-snapshot?
Still using old ubuntu 22.04 (jammy)?
Don't trust random PPA without source for rebuild?
Don't like snap or flatpak (hard to add more tools such as ripgrep)?

You're maybe at the right location!

I wanted to explore [Earthly](https://docs.earthly.dev/) for the build framework instead of raw docker, and here is the result:

**Info**: check the other git branches for other Ubuntu versions.

# Usage
## Build
([Get Earthly](https://docs.earthly.dev/), (which requires docker and git), then:)

Build and get `.deb` files in `dist/`:
```shell
earthly +build-emacs
```

## Install
```shell
cd dist/
sudo apt-get install -y ./emacs-bin-common_*.deb ./emacs-common_*.deb ./emacs-el_*.deb ./emacs-gtk_*.deb ./libtree-sitter0_*.deb
```
(adapt if you want another variant than `gtk`, maybe `nox`?)

# Development
See comments in `Earthfile`.

Use `earthly -i` on any earthly target to get a root shell on error.

## How to create the patches
The backport build required some adjustments (gcc-11, debhelper 10 instead of 13), and libtree-sitter:

### `tree-sitter` patch
Get `tree-sitter_*.debian.tar.xz` files, decompress them in subdirs, then:

```shell
cp -r tree-sitter-ubuntu-noble{,-on-jammy}
# edit files
ec tree-sitter-ubuntu-noble-on-jammy/debian/control
# fix 'dh' error
echo 10 > tree-sitter-ubuntu-noble-on-jammy/debian/compat

# create patch: -N for new files (treated as empty)
diff -urN tree-sitter-ubuntu-noble{,-on-jammy} > tree-sitter.patch
```

### `emacs` patch
Get `emacs_*.debian.tar.xz` files, decompress them in subdirs, then:

```shell
cp -r emacs-ubuntu-noble{,-on-jammy}
# edit files
ec emacs-ubuntu-noble-on-jammy/debian/control
# fix 'dh' error
echo 10 > emacs-ubuntu-noble-on-jammy/debian/compat

# create patch: -N for new files (treated as empty)
find emacs-ubuntu-noble-on-jammy/ -name '*~' -delete
diff -urN emacs-ubuntu-noble{,-on-jammy} > emacs.patch
```

## Test
```shell
earthly -i +test
```
