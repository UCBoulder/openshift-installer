# This branch applies our resource pool modifications on top of the OKD 4.7 release

## To compile openshift-installer:

For linux:

```sh
TAGS="okd" hack/build.sh
tar -cvf openshift-install-linux-4.7.0-0.okd-2021-07-03-190901-resource-pool.tar.gz bin/openshift-install README.md
```

For mac:

```sh
go generate ./data
SKIP_GENERATION=y GOOS=darwin GOARCH=amd64 TAGS="okd" hack/build.sh
tar -cvf openshift-install-mac-4.7.0-0.okd-2021-07-03-190901-resource-pool.tar.gz bin/openshift-install README.md
```

Generate sha256sums:

```sh
sha256sum *.tar.gz > sha256sum.txt
```

## To bump version:

Rebase on correct `release-4.X-okd` branch from [vrutkovs](https://github.com/vrutkovs/installer/branches), then grab the "Pull From: " container ref of your desired release. For example [4.7.0-0.okd-2021-07-03-190901](https://github.com/openshift/okd/releases/tag/4.7.0-0.okd-2021-07-03-190901) looks like:

```
Name:           4.7.0-0.okd-2021-07-03-190901
Digest:         sha256:66857f330c4b6199ce319c9b5b5ff3bd7ff3d4c75113cd3d33302216312d345f
Created:        2021-07-03T20:10:00Z
OS/Arch:        linux/amd64
Manifests:      493
Metadata files: 1

Pull From: quay.io/openshift/okd@sha256:66857f330c4b6199ce319c9b5b5ff3bd7ff3d4c75113cd3d33302216312d345f

Release Metadata:
  Version:  4.7.0-0.okd-2021-07-03-190901
  Upgrades: <none>
  Metadata:
```

In this case, the value you would grab is `quay.io/openshift/okd@sha256:66857f330c4b6199ce319c9b5b5ff3bd7ff3d4c75113cd3d33302216312d345f`. Then, paste that into `pkg/asset/releaseimage/default.go` like so:


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