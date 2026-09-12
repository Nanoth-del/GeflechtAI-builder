# Generic iOS build controller

Workflow-only controller for building an unsigned iOS IPA from a selected
private source repository. The source tree, Xcode output, SwiftPM details, and
packaging files stay on the ephemeral macOS runner; the Actions log contains
only short status messages and sanitized compiler diagnostics.

## Required secret

This workflow is currently configured for the `GeflechtEQ` project. Create a
repository secret named `GEFLECHTEQ` using a fine-grained token restricted to
the `Nanoth-del/GeflechtEQ` repository with only the permissions needed for
this run: `Contents: Read and write`. The workflow uses this one secret for
both private-source checkout and Release publication because the project owner
explicitly accepted that shared-token boundary.

Using separate source-read and release-write credentials is safer and remains
recommended for a general-purpose builder. If this repository is reused for
other projects, restore separate least-privilege secrets and keep the builder
private or protect the job with a reviewed environment.

Do not use a broad personal token when this repository is public. Anyone who
can start the workflow could otherwise choose a repository that the token can
read or a Release that it can modify. A private builder or tokens scoped only
to the intended projects is the recommended deployment boundary.

## Workflow inputs

Run `Generic iOS unsigned IPA builder` with:

- `source_repository`: `owner/repository` for the private source.
- `source_ref`: branch, tag, or commit SHA.
- `project_path`: relative `.xcodeproj` or `.xcworkspace` path.
- `scheme`: Xcode scheme to archive.
- `app_name`: resulting application product name without `.app`.
- `source_runtime`: `auto`, `none`, `mlx`, or `llama`.
- `release_repository`: optional Release destination; defaults to the source repository.
- `publish_release`: whether to publish the unsigned IPA as a prerelease.

The workflow uses a shallow private checkout, validates all path and repository
inputs, keeps Git author metadata on a bot noreply address, and withholds
checkout, toolchain, package, compiler, and release command output from the
public log.
