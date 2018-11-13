# nix-cabal

This is a small wrapper script to run `cabal` inside `nix-shell`. It assumes
`shell.nix` to be present in the current directory. Example usage:

```bash
nix-cabal new-build all
```

Here's an example of a `shell.nix` you might want to use with this script:

```
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
  '';
  LOCALE_ARCHIVES =
    if pkgs.stdenv.isLinux
    then "${pkgs.glibcLocales}/lib/locale/locale-archive"
    else "";
}
```
