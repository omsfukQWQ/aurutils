# aurutils

Collection of helper tools for use with the Arch User Repository.

## aurbuild

```aurbuild [options] [file]```

The input file must include names of directories containing a PKGBUILD file. The ```-d``` (database), ```-r``` (root) and ```-p``` (pool) arguments are relayed to repose. ```-c``` builds a package in an nspawn-container (requires _devtools_).

To get started, create a local repository:

```
 $ sudo vim /etc/pacman.conf # uncomment [custom], change Server to suit
 $ sudo install -d /home/custompkgs -o $USER
 $ repo-add /home/custompkgs/custom.db.tar
 $ sudo pacman -Syu
```

See also "Migrating foreign packages".

## aurchain

```aurchain pkgname ...```

_pkgname_ must be the name of an AUR package. Dependencies are retrieved recursively, sorted topologically, and printed to stdout.

### Examples

Run actions on AUR targets in total order:

```
 $ while read -r pkg; do ... done < <(aurchain _foobar_)
```

## aurqueue

```aurqueue pkgbase depends ...```

_pkgbase_ must be a directory containing a .SRCINFO file. Dependencies are not retrieved recursively, and must be specified on the command line for a complete graph.

### Examples

Build all packages in the _pkgbuilds_ github repository:

```
 $ git clone https://www.github.com/Earnestly/pkgbuilds
 $ cd pkgbuilds
 $ find -maxdepth 2 -name PKGBUILD -execdir mksrcinfo \;
 $ aurbuild -d custom.db -r /home/custompkgs -p /home/custompkgs <(aurqueue *)
```

## aursearch

```aursearch [options] pattern```

Search AUR packages from a PCRE pattern. Due to aurweb limitations, results are searched by name only.

## aursift

```command ... | aursift | ...```

Filter input for packages in the official Arch Linux repositories. Virtual packages (provides/replaces) are solved.

### Examples

Search for perl modules that are both in the AUR and official repositories:

```
 $ aursearch -p '^perl-.+' > pkgs
 $ grep -Fxvf <(aursift < pkgs) pkgs
```

## aursync

```aursync [options] package```

Wrapper for aurchain, aurbuild and repofind. Build files are:

+ downloaded with `git` (`-t`: `.tar.gz` snapshots)
+ inspected with PAGER or, when installed, `vifm` (`-k`: disable)
+ updated in case of VCS packages (`-n`: disable)
+ marked for building if newer (`-n`: disable)

### Examples

Build plasma-desktop-git and its dependencies (add `-c` to use an nspawn container):

```
 $ aursync plasma-desktop-git
```

Query the AUR for updates, and build the results:

```
 $ aursync -n $(repofind -u | awk '{print $1}')
```

Rebuild all packages in the _custom_ repository:

```
 $ aursync -fn $(pacman -Slq custom)
```

## repofind

```repofind [options]```

Print (`-i`) or select (`-s`) `file://` repositories. `-u` checks packages for updates in the AUR (implies `-s`).

# Migrating foreign packages (optional)

This is straightforward if the built packages are still available, for example in `/home/packages`:

```
 $ cd /home/packages
 $ repose -fv custom.db
 $ sudo pacman -Syu
```

To reverse this operation, repeat the procedure with `--drop`:

```
 $ repose -dfv custom.db
```

Without packages, check the installed files first. If needed, rebuild packages with md5sum mismatches.

```
 $ pacman -Qqm | paccheck --md5sum --quiet
```

Recreate the packages, and save them to PKGDEST, or PWD if not set:

```
 $ for b in $(pacman -Qqm); do bacman "$b"; done
```

To check for AUR updates, use `repofind -u` or pass the repository name to a compatible helper. For example: `pacaur --ignorerepo=custom -Syu`, `cower -u --ignorerepo=custom`.

To keep the repository updated when building with other AUR helpers, set PKGDEST and create a repose alias:

```
 $ sudo vim /etc/makepkg.conf
 $ alias custom='repose -vf custom.db -p /home/packages -r /home/packages'
```
