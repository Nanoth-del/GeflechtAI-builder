# Generic iOS build controller

Workflow-only controller for building an unsigned iOS IPA from a selected
private source repository. The source tree, Xcode output, SwiftPM details, and
packaging files stay on the ephemeral macOS runner; the Actions log contains
only short status messages and sanitized compiler diagnostics.

## Required secret

Create a repository secret named `SOURCE_ACCESS_TOKEN`. Use a narrowly scoped
GitHub App or fine-grained token that can read the selected source repository.

If `publish_release` is enabled, also create `RELEASE_ACCESS_TOKEN` with the
minimum permission needed to create a release in the selected
`release_repository`. Keeping source-read and release-write credentials
separate is recommended; the same token can be used only when the project
owner deliberately accepts that broader scope.

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
