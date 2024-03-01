#golang #package 

### References
https://go.dev/doc/go1.4
### Internal packages

Go’s package system makes it easy to structure programs into components with clean boundaries, but there are only two forms of access: local (unexported) and global (exported). Sometimes one wishes to have components that are not exported, for instance to avoid acquiring clients of interfaces to code that is part of a public repository but not intended for use outside the program to which it belongs.

The Go language does not have the power to enforce this distinction, but as of Go 1.4 the [`go`](https://go.dev/cmd/go/) command introduces a mechanism to define “internal” packages that may not be imported by packages outside the source subtree in which they reside.

To create such a package, place it in a directory named `internal` or in a subdirectory of a directory named internal. When the `go` command sees an import of a package with `internal` in its path, it verifies that the package doing the import is within the tree rooted at the parent of the `internal` directory. For example, a package `.../a/b/c/internal/d/e/f` can be imported only by code in the directory tree rooted at `.../a/b/c`. It cannot be imported by code in `.../a/b/g` or in any other repository.

For Go 1.4, the internal package mechanism is enforced for the main Go repository; from 1.5 and onward it will be enforced for any repository.

Full details of the mechanism are in [the design document](https://go.dev/s/go14internal).

### Canonical import paths

Code often lives in repositories hosted by public services such as `github.com`, meaning that the import paths for packages begin with the name of the hosting service, `github.com/rsc/pdf` for example. One can use [an existing mechanism](https://go.dev/cmd/go/#hdr-Remote_import_paths) to provide a “custom” or “vanity” import path such as `rsc.io/pdf`, but that creates two valid import paths for the package. That is a problem: one may inadvertently import the package through the two distinct paths in a single program, which is wasteful; miss an update to a package because the path being used is not recognized to be out of date; or break clients using the old path by moving the package to a different hosting service.

Go 1.4 introduces an annotation for package clauses in Go source that identify a canonical import path for the package. If an import is attempted using a path that is not canonical, the [`go`](https://go.dev/cmd/go/) command will refuse to compile the importing package.

The syntax is simple: put an identifying comment on the package line. For our example, the package clause would read:

```
package pdf // import "rsc.io/pdf"
```

With this in place, the `go` command will refuse to compile a package that imports `github.com/rsc/pdf`, ensuring that the code can be moved without breaking users.

The check is at build time, not download time, so if `go` `get` fails because of this check, the mis-imported package has been copied to the local machine and should be removed manually.

To complement this new feature, a check has been added at update time to verify that the local package’s remote repository matches that of its custom import. The `go` `get` `-u` command will fail to update a package if its remote repository has changed since it was first downloaded. The new `-f` flag overrides this check.

Further information is in [the design document](https://go.dev/s/go14customimport).