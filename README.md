# This branch applies our resource pool modifications on top of the OCP 4.9.5 release

## To compile openshift-installer:

For linux:

```sh
hack/build.sh
tar -cvf openshift-install-linux-4.9.5-resource-pool.tar.gz bin/openshift-install README.md
```

For mac:

```sh
go generate ./data
SKIP_GENERATION=y GOOS=darwin GOARCH=amd64 hack/build.sh
tar -cvf openshift-install-mac-4.9.5-resource-pool.tar.gz bin/openshift-install README.md
```

Generate sha256sums:

```sh
sha256sum *.tar.gz > sha256sum.txt
```

## To bump version:

Download desired installer from the [cloud console](https://console.redhat.com/openshift/downloads#tool-x86_64-openshift-install). Extract it and run `./openshift-install version`.

Fetch correct `release-4.X` branch from [openshift/installer](https://github.com/openshift/installer/branches), then grab the "built from" and "release image". For example [4.9.5](https://docs.openshift.com/container-platform/4.9/release_notes/ocp-4-9-release-notes.html#ocp-4-9-5) looks like:

```bash
$ ./openshift-install version
./openshift-install 4.9.5
built from commit 8223216bdcef5de56b52240ab7160ca909a9e56c
release image quay.io/openshift-release-dev/ocp-release@sha256:386f4e08c48d01e0c73d294a88bb64fac3284d1d16a5b8938deb3b8699825a88
release architecture amd64
```

Create a new branch from "resource-pool", then rebase on the "built from" commit, like so:

```bash
git branch release-4.X-resource-pool release-4.9.5-resource-pool
git checkout release-4.X-resource-pool
git rebase 8223216bdcef5de56b52240ab7160ca909a9e56c
```

Update the "release image" because we don't seem to have the tools that hack it in there later. In this case, the value you would grab is `uay.io/openshift-release-dev/ocp-release@sha256:386f4e08c48d01e0c73d294a88bb64fac3284d1d16a5b8938deb3b8699825a88`. Then, paste that into `pkg/asset/releaseimage/default.go` like so:


```go
var (
	// defaultReleaseImageOriginal is the value served when defaultReleaseImagePadded is unmodified.
	defaultReleaseImageOriginal = "quay.io/openshift/okd@sha256:66857f330c4b6199ce319c9b5b5ff3bd7ff3d4c75113cd3d33302216312d345f"
	// defaultReleaseImagePadded may be replaced in the binary with a pull spec that overrides defaultReleaseImage as
	// a null-terminated string within the allowed character length. This allows a distributor to override the payload
	// location without having to rebuild the source.
	defaultReleaseImagePadded = "\x00_RELEASE_IMAGE_LOCATION_\x00XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\x00"
	defaultReleaseImagePrefix = "\x00_RELEASE_IMAGE_LOCATION_\x00"
	defaultReleaseImageLength = len(defaultReleaseImagePadded)
)
```