# How To Break Build With Go Vendor Experiment

The `GO15VENDOREXPERIMENT` provides a powerful way to place packages your project needs within the project. Yet, there is an easy way for this to break things.

Take a project with this structure:
```
- $GOPATH/src/github.com/mattfarina/golang-broken-vendor
  - foo.go
  - vendor/
    - a/
    - b/
        - vendor/a/
```

In this example both `a` packages are exactly the same. The package `b` stores the `a` package within its codebase. The top level application also includes the `a` package.

The file `foo.go` is fairly simple in that it does...
```go
func main() {
	var it a.A
	it = "foo"

	b.Do(it)
}
```

The problem is this doesn't build. Trying to build this returns the error:
```sh
$ GO15VENDOREXPERIMENT=1 go build
./foo.go:12: cannot use it (type "github.com/mattfarina/golang-broken-vendor/vendor/a".A) as type "github.com/mattfarina/golang-broken-vendor/vendor/b/vendor/a".A in argument to b.Do
```

You can clone this repo and try this for yourself.

With [Glide](https://github.com/Masterminds/glide) (a Go package manager) we've found the best solution is to flatten the included dependencies to the top level `vendor/` directory. If a package is meant to be included by another application it may be better to not store your external dependencies. If you use Glide you can specify them in a `glide.yaml` file and Glide will take care to install the right versions.

In any case, be careful how and where you include dependencies.
