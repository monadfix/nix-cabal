# nix-cabal

This is a small wrapper script to run `cabal` inside `nix-shell`. It assumes
`shell.nix` to be present in the current directory.

## Installation

Download the script to a location of your choice, for instance
`$HOME/.local/bin/`, and make it executable:

```bash
curl https://raw.githubusercontent.com/monadfix/nix-cabal/master/nix-cabal -o $HOME/.local/bin/nix-cabal
chmod u+x $HOME/.local/bin/nix-cabal
```

Make sure this location is in your `$PATH`.

## Usage

```bash
nix-cabal new-build all
```

Here's an example of a `shell.nix` you might want to use with this script:

```nix
{
  pkgs ? import <nixpkgs> {},
  hc ? "ghc844"
}:

pkgs.stdenv.mkDerivation rec {
  name = "example";
  buildInputs = [
    pkgs.haskell.compiler.${hc}
    pkgs.git
    pkgs.zlib
    pkgs.cabal-install
    pkgs.pkgconfig
    pkgs.which
  ];
  shellHook = ''
    export LD_LIBRARY_PATH=${pkgs.lib.makeLibraryPath buildInputs}:$LD_LIBRARY_PATH
    export LANG=en_US.UTF-8
  '';
  LOCALE_ARCHIVE =
    if pkgs.stdenv.isLinux
    then "${pkgs.glibcLocales}/lib/locale/locale-archive"
    else "";
}
```

## Travis CI

Here's an example of a `.travis.yml`:

```yaml
language: nix

os:
  - linux
  - osx

env:
  - HC=ghc822
  - HC=ghc844
  - HC=ghc861

before_install:
  - curl https://raw.githubusercontent.com/monadfix/nix-cabal/master/nix-cabal -o nix-cabal && chmod u+x nix-cabal
  - travis_retry ./nix-cabal new-update

script:
  - ./nix-cabal new-test
  - ./nix-cabal new-bench

cache:
  directories:
    - ~/.cabal
    - ~/.ghc
```
