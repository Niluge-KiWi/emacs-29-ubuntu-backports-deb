VERSION 0.7
FROM ubuntu:22.04
WORKDIR /workdir

# goal: build ubuntu 24.04 emacs package from ubuntu 22.04
# following: https://www.cmiss.org/cmgui/wiki/BuildingUbuntuPackagesFromSource
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential fakeroot dpkg-dev devscripts wget

build-tree-sitter:
    # get deb source files from ubuntu 24.04: https://packages.ubuntu.com/noble/libtree-sitter-dev
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/t/tree-sitter/tree-sitter_0.20.8.orig.tar.xz
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/t/tree-sitter/tree-sitter_0.20.8-2.dsc
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/t/tree-sitter/tree-sitter_0.20.8-2.debian.tar.xz

    # extract the source and patch it using the diff file
    RUN dpkg-source -x ./tree-sitter_0.20.8-2.dsc ./tree-sitter/

    WORKDIR tree-sitter/

    # patch build dependencies for ubuntu 22.04
    COPY ./tree-sitter.patch .
    RUN patch -p1 < ./tree-sitter.patch

    # get build dependencies
    RUN DEBIAN_FRONTEND=noninteractive apt-get build-dep -y ./


    # bump version and add changelog entry: https://wiki.debian.org/BuildingFormalBackports
    RUN DEBFULLNAME="Thomas Riccardi <riccardi.thomas@gmail.com>" dch --distribution jammy --force-distribution --nmu "Build-Dependencies adjustments for backport on ubuntu jammy"

    # build the source file
    RUN dpkg-buildpackage --build=binary

    WORKDIR ..

    # outputs
    SAVE ARTIFACT ./libtree-sitter*deb /dist/ AS LOCAL dist/

    # uncomment for debug, with earthly -i
    # RUN false


build-emacs:
    # get deb source files from ubuntu 24.04: https://packages.ubuntu.com/noble/emacs
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/e/emacs/emacs_29.1+1.orig.tar.xz
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/e/emacs/emacs_29.1+1-5ubuntu1.dsc
    RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/e/emacs/emacs_29.1+1-5ubuntu1.debian.tar.xz

    # extract the source and patch it using the diff file
    RUN dpkg-source -x ./emacs_29.1+1-5ubuntu1.dsc ./emacs/

    WORKDIR emacs/

    # patch build dependencies for ubuntu 22.04
    COPY ./emacs.patch .
    RUN patch -p1 < ./emacs.patch

    # get build dependencies
    # manually for libtree-sitter-dev
    COPY +build-tree-sitter/dist/ tree-sitter/
    RUN dpkg -i tree-sitter/*.deb
    # also save them as output
    SAVE ARTIFACT ./tree-sitter/*deb /dist/ AS LOCAL dist/
    # get build dependencies
    RUN DEBIAN_FRONTEND=noninteractive apt-get build-dep -y ./


    # bump version and add changelog entry: https://wiki.debian.org/BuildingFormalBackports
    RUN DEBFULLNAME="Thomas Riccardi <riccardi.thomas@gmail.com>" dch --distribution jammy --force-distribution --nmu "Build-Dependencies adjustments for backport on ubuntu jammy"

    # build the source file
    RUN dpkg-buildpackage --build=binary

    WORKDIR ..

    # outputs
    SAVE ARTIFACT ./emacs*deb /dist/ AS LOCAL dist/

    # uncomment for debug, with earthly -i
    # RUN false


test:
    FROM ubuntu:22.04
    WORKDIR /workdir
    RUN apt-get update
    COPY +build-emacs/dist/*.deb .
    RUN DEBIAN_FRONTEND=noninteractive apt-get install -y ./emacs-bin-common_*.deb ./emacs-common_*.deb ./emacs-el_*.deb ./emacs-gtk_*.deb ./libtree-sitter0_*.deb
    RUN emacs -batch --eval '(message system-configuration-features)' 2> system-configuration-features.txt
    RUN cat ./system-configuration-features.txt
    SAVE ARTIFACT ./system-configuration-features.txt

    # RUN false
