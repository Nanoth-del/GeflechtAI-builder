# GeflechtEQ isolated unsigned IPA builder

This public controller builds one private iOS project on an ephemeral macOS
runner. The private source tree, Xcode output, package-resolution details,
archive, IPA, checksums, and audit logs never leave the runner except for the
verified unsigned IPA published to the project's private prerelease.

## Public metadata boundary

This repository intentionally exposes only the controller, its target project
identity, and an immutable source commit SHA supplied at dispatch. It does not
publish source files, private credentials, build paths, compiler diagnostics,
checksums, or an IPA artifact. The workflow accepts no repository, project,
scheme, application-name, or Release-destination input.

The IPA necessarily retains runtime bundle metadata required by iOS, such as
the bundle identifier, app name, and version. It is not a signed or installable
distribution artifact; re-sign it privately before installation.

## Required one-time GitHub configuration

Before dispatching, create an environment named `private-release` in this
repository. Limit it to the `main` branch and protect approval according to
your collaboration model. Store these **environment secrets** there, not as
repository secrets:

- `SOURCE_ACCESS_TOKEN`: fine-grained token restricted to the private source
  repository with **Contents: Read** only.
- `RELEASE_WRITE_TOKEN`: separate fine-grained token restricted to the same
  repository with only the permissions required to create the private
  prerelease and upload its asset.

After migration, remove the legacy shared repository secret rather than
keeping a read/write credential available to every workflow in this public
repository. Never paste a token into source, workflow inputs, release notes,
issues, or Actions logs.

## Dispatch contract

Run **GeflechtEQ private unsigned IPA build** with the exact, lowercase,
40-character source commit SHA. The workflow rejects branches, tags, short
SHAs, and arbitrary source / Release destinations.

The workflow performs all of the following privately:

1. Fetches and verifies the requested immutable source revision using the
   source-read credential without persisting it in Git configuration.
2. Resolves packages, cleans, and archives with signing disabled.
3. Packages the IPA and audits it for source/build products, profiles,
   signatures, source paths, runner paths, credential patterns, and unwanted
   archive metadata.
4. Publishes only to the private prerelease, downloads that asset back using
   the release-write credential, and verifies its SHA-256 digest before
   cleanup.
5. Removes the private checkout, archive, derived data, IPA copies, audit
   directory, verification copy, and every captured log with `if: always()`.
