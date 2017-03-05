Release Process
====================
Meta: There should always be a single release engineer to disambiguate responsibility.

## Pre-release

Check all of the following:

- All dependencies have been updated as appropriate:
  - BDB
  - Boost
  - ccache
  - libgmp
  - libsnark (upstream of our fork)
  - libsodium
  - miniupnpc
  - OpenSSL

## A. Define the release version as:

    $ DWCASH_RELEASE=MAJOR.MINOR.REVISION(-BUILD_STRING)

Example:

    $ DWCASH_RELEASE=1.0.0-beta2

Also, the following commands use the `DWCASH_RELEASE_PREV` bash variable for the
previous release:

    $ DWCASH_RELEASE_PREV=1.0.0-beta1

## B. Create a new release branch / github PR
### B1. Update (commit) version in sources

    README.md
    src/clientversion.h
    configure.ac
    contrib/gitian-descriptors/gitian-linux.yml

In `configure.ac` and `clientversion.h`:

- Increment `CLIENT_VERSION_BUILD` according to the following schema:

  - 0-24: `1.0.0-beta1`-`1.0.0-beta25`
  - 25-49: `1.0.0-rc1`-`1.0.0-rc25`
  - 50: `1.0.0`
  - 51-99: `1.0.0-1`-`1.0.0-49`
  - (`CLIENT_VERSION_REVISION` rolls over)
  - 0-24: `1.0.1-beta1`-`1.0.1-beta25`

- Change `CLIENT_VERSION_IS_RELEASE` to false while DeepWebCash is in beta-test phase.

If this release changes the behavior of the protocol or fixes a serious bug, we may
also wish to change the `PROTOCOL_VERSION` in `version.h`.

Build and commit to update versions, and then perform the following command:

    $ bash contrib/devtools/gen-manpages.sh

Commit the changes.

### B2. Write release notes

git shortlog helps a lot, for example:

    $ git shortlog --no-merges v${DWCASH_RELEASE_PREV}..HEAD \
        > ./doc/release-notes/release-notes-${DWCASH_RELEASE}.md

Add the newly created release notes to the Git repository:

    $ git add doc/release-notes/release-notes-$DWCASH_RELEASE.md

Update the Debian package changelog:

    export DEBVERSION="${DWCASH_RELEASE}"
    export DEBEMAIL="${DEBEMAIL:-team@dw.cash}"
    export DEBFULLNAME="${DEBFULLNAME:-DeepWebCash Company}"

    dch -v $DEBVERSION -D jessie -c contrib/debian/changelog

(`dch` comes from the devscripts package.)

### B3. Change the network magics

If this release breaks backwards compatibility, change the network magic
numbers. Set the four `pchMessageStart` in `CTestNetParams` in `chainparams.cpp`
to random values.

### B4. Merge the previous changes

Do the normal pull-request, review, testing process for this release PR.

## C. Verify code artifact hosting

### C1. Ensure depends tree is working

https://ci.dw.cash/builders/depends-sources

### C2. Ensure public parameters work

Run `./fetch-params.sh`.

## D. Make tag for the newly merged result

In this example, we ensure master is up to date with the
previous merged PR, then:

    $ git tag -s v${DWCASH_RELEASE}
    $ git push origin v${DWCASH_RELEASE}

## E. Deploy testnet

Notify the DeepWebCash DevOps engineer/sysadmin that the release has been tagged. They update some variables in the company's automation code and then run an Ansible playbook, which:

* builds DeepWebCash based on the specified branch
* deploys it as a public service (e.g. betatestnet.dw.cash, mainnet.dw.cash)
* often the same server can be re-used, and the role idempotently handles upgrades, but if not then they also need to update DNS records
* possible manual steps: blowing away the `testnet3` dir, deleting old parameters, restarting DNS seeder

Then, verify that nodes can connect to the testnet server, and update the guide on the wiki to ensure the correct hostname is listed in the recommended dwcash.conf.

## F. Update the 1.0 User Guide

## G. Publish the release announcement (blog, dwcash-dev, slack)

### G1. Check in with users who opened issues that were resolved in the release

Contact all users who opened `user support` issues that were resolved in the release, and ask them if the release fixes or improves their issue.

## H. Make and deploy deterministic builds

- Run the [Gitian deterministic build environment](https://github.com/deepwebcash/deepwebcash-gitian)
- Compare the uploaded [build manifests on gitian.sigs](https://github.com/deepwebcash/gitian.sigs)
- If all is well, the DevOps engineer will build the Debian packages and update the
  [apt.dw.cash package repository](https://apt.dw.cash).

## I. Celebrate

## missing steps
DeepWebCash still needs:

* thorough pre-release testing (presumably more thorough than standard PR tests)

* automated release deployment (e.g.: updating build-depends mirror, deploying testnet, etc...)
