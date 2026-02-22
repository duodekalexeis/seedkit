
Reproducing a tagged seedkit build
==================================

Seedkit uses [reproducible builds](https://reproducible-builds.org/),
which allows users to personally reproduce an identical binary from
given source files.

This allows technical users who have reviewed a certain set of source
code to build local binaries and tar files directly from that source
code, and to verify that they are bit-for-bit identical with the published
versions.

This procedure does NOT need to be done on an air-gapped live system
like [Tails](https://tails.net/) - you can build and verify the build
artifacts on any system, and then confirm that the binary you are using
later on your secure system is identical.


## Dependencies

* Go: seedkit is written in Go, and a reproducible build requires the
  same version of Go as was used for the build, which should be the
  latest stable release, currently Go 1.26.0 (try `go version` to see
  if you have one already installed).

  If not, install the latest stable release of Go using the
  [official Go installation instructions](https://golang.org/doc/install).


* Goreleaser - seedkit uses [goreleaser](https://goreleaser.com) to
  build the seedkit releases, and this provides a convenient way to
  replicate an identical build process.

  Install 

  ```bash
  go get -u github.com/goreleaser/goreleaser
  ```

* A unix-like shell environment - this recipe should work directly on
  Linux and Mac, but on Windows will probably require a Windows
  Subsystem for Linux (WSL) environment.

  Install using the official [WSL installation instructions](https://docs.microsoft.com/en-us/windows/wsl/install).


## Procedure

* Check [the latest seedkit release](https://github.com/duodekalexeis/seedkit/releases/latest)
  to get the tag version (a string like `vX.Y.X`).

* Then in a shell environment, do:

```bash
# e.g.
cd /tmp

# Set VTAG and TAG variables for the seedkit release you want to reproduce
VTAG=v0.3.9
TAG=${VTAG#v}
echo -e "$VTAG\n$TAG"

# Set an OS_ARCH variable for the OS+architecture combination you are wanting to check.
# The architecture will be x86_64 or arm64 - check with `uname -m` if you're not sure.
# But note that if your tails machine uses a different architecture you should use that
# (or verify both architectures, for completeness).
# Then download the release tarball/zipfile for the tag and your OS_ARCH e.g.
OS_ARCH=Linux_x86_64
OS_ARCH=Darwin_amd64
OS_ARCH=Windows_x86_64
# Set $SUFFIX appropriately
if [ ${OS_ARCH%_*} == Windows ]; then
  SUFFIX=zip
else
  SUFFIX=tar.gz
fi
echo seedkit_${OS_ARCH}.$SUFFIX
# Download the release archive and the release checksums
curl -L -o seedkit_${OS_ARCH}.$SUFFIX \
  https://github.com/duodekalexeis/seedkit/releases/download/$VTAG/seedkit_${OS_ARCH}.$SUFFIX
ls -l seedkit_${OS_ARCH}.$SUFFIX
# Check the checksum of your archive against the release checksums
sha256sum seedkit_${OS_ARCH}.$SUFFIX
curl -L https://github.com/duodekalexeis/seedkit/releases/download/$VTAG/seedkit_${TAG}_checksums.txt

# Assuming these match, you can now extract the binary from the release archive
mkdir seedkit_release_$TAG
cd seedkit_release_$TAG
# For tarballs:
tar zxvf ../seedkit_${OS_ARCH}.$SUFFIX
# For zip files:
unzip ../seedkit_${OS_ARCH}.$SUFFIX
ls -l seedkit

# Now clone the seedkit repository for your tag
cd ..
git clone --depth 1 --branch $VTAG https://github.com/duodekalexeis/seedkit
# (ignore the warnings about being in `detached HEAD` state)

# Change to the seedkit directory
cd seedkit

# And use goreleaser to do a local build
goreleaser --skip=publish,sign --clean

# Note that we cannot verify the checksums on the tarballs/zipfiles you have just built,
# as they will differ because of the local build metadata e.g. your build user, timestamps,
# umask-affected permissions, etc.
# Instead, we need to verify the checksums of the binaries *within* the archives.

# Extract the newly built binary
# For tarballs:
tar zxvf dist/seedkit_${OS_ARCH}.$SUFFIX
# For zip files:
unzip dist../seedkit_${OS_ARCH}.$SUFFIX
ls -l seedkit

# And now compare the checksums of your newly build binary and the release version
sha256sum seedkit
sha256sum ../seedkit_release_$TAG/seedkit
```

* If these match, you have verified that the release binary was built from the
  source code you cloned, and that reviews of that source code are valid for the
  binary you have verified.


* If you are using Tails, you can now use either the binary you have just built
  or the release binary you have verified on your Tails machine, and be confident
  at least that it matches the source code you have cloned. See the
  [installing seedkit on tails](https://github.com/duodekalexeis/seedkit/blob/main/recipes/installing_seedkit_on_tails.md)
  recipe for details on installing on Tails.

