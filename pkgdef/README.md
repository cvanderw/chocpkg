
Package files are shell scripts which set variables and call well-defined
functions to describe how to build the package.

## Modules

`chocpkg` is divided into modules which provide specific functionality.
Modules are installed by calling certain dedicated functions.
Usually, your package should install a check module, a fetch module
and a build module.

### Check modules

Check modules are modules which determine whether the package is
installed on the system (may have been installed by `chocpkg` itself
or the system package manager).
If a check module is not installed, the default is to always
assume the package is not installed:

* `check_pkgconfig`: invokes `pkg-config` to determine if the package
  is installed. The name of the `pkg-config` .pc file must be provided
  as an argument to `check_pkgconfig`; for example:

```shell
check_pkgconfig SDL2
```

* `check_tool`: checks if a particular tool is installed in `$PATH`
  to determine if the package is installed. The name of the tool must
  be provided as an argument to `check_tool`; for example:

```shell
check_tool gnome-terminal
```

### Fetch modules

Fetch modules specify how to retrieve the package from the Internet:

* `fetch_download`: downloads the package from a URL specified as
  an argument.
  The file to
  be downloaded is assumed to be a well-formed tar.gz file with all its
  contents in a directory that matches the package name; if this is not
  the case, the variable `$IS_TAR_BOMB` should be set to `true`.
  Example use:

```shell
fetch_download http://example.com/example-pkg.tar.gz
IS_TAR_BOMB=true
```

* `fetch_git`: downloads the package from a Git repository at the URL
  given as an argument.
  The branch `master`
  will be checked out by default; this can be overriden by providing
  the branch name as a second argument to `fetch_git`.
  Example use:

```shell
fetch_git http://example.com/example.git my-neato-branch
```

### Build modules

Build modules specify how to build the package after it is fetched:

* `build_autotools`: builds the package assuming that it is laid out
  as a standard autotools package (ie. `./configure; make; make install`).
  Extra arguments passed to the function will be passed through to
  `configure`.
  Example use:

```shell
build_autotools --disable-broken-feature
```

### Other modules and functions

* `dependencies`: Arguments provided to the function are the names
  of other packages to install before trying to build this one.
  Example use:

```shell
dependencies other-package neato-lib
```

* `package_group`: Specifies that this is not really a package that
  should be built; rather, it just specifies a number of other packages
  to build.
  Example use:

```shell
# File contains no other lines
package_group neato-lib dumbo-lib
```

## Variables

* `PACKAGE_TYPE`: has a value of either `native` or `target`. The
  default is `target`. The variable controls whether the package is
  built for the target system when doing cross-compiles, or whether it
  is part of the build (native) system and just used to build tools as
  part of the build process. The package type determines whether the
  result is installed into `install` or `install.native`.

* `PACKAGE_VERSION`: version number of the package. If set, this is
  appended to the package name (ie. filename) to make the expected build
  directory name.

## Advanced builds

Complicated packages can require custom build steps. The following functions
can therefore be overridden in the package file in exceptional circumstances.
These are essentially the functions implemented by the modules described
above.

* `do_fetch`: the function which is invoked to fetch the package from the
  Internet. The fetched package is placed into `$PACKAGE_BUILD_DIR` for
  build.

* `prebuild_setup`: define a function with this name to execute special
  setup commands in the root of a package just before it is built.

* `do_build`: the function which is invoked to build the package.

* `do_install`: the function which is invoked to install the package after
  it has been built. This function should install built files from
  `$PACKAGE_BUILD_DIR` into `$PACKAGE_INSTALL_DIR`.
