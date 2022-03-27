# Projects

This document describes the organization and workflow for Optakt projects, and acts as a centralized table of contents for project documentation.

## Organization

Any Optakt project should have at least the following:

* Continuous Integration (CI) using [GitHub Actions](https://github.com/features/actions)
* Automatic Releases when a tag is pushed (using [`goreleaser`](https://github.com/goreleaser/goreleaser) and a GitHub Action)
* A Pull Request (PR) template that guides contributors into the requirements for a successful PR
* Proper documentation
    * License at the root of the repository
    * README that describes the project, links to Godoc, shows build status and clearly enumerates the project dependencies
    * Each executable and API within the project should have their own dedicated documentation
        * For executable, document purpose, flags and give example uses
        * For APIs, document purpose and describe endpoints with expected inputs and outputs
    * Document on getting started/contributing
    * Architecture document
        * This document should explain clearly how this project will interact with other components
        * For complex projects, this should also include a model of how the internal components of the project interact

### Continuous Integration

We use GitHub Actions for our CI, since it allows us not to depend on any external services besides GitHub, while being a free and simple solution.

Projects should have one action for verifying the integrity of incoming PRs. This can include but is not limited to:

* Verifying that formatting is correct
* Verifying that there are no linter warnings
* Verifying that tests are passing
* Verifying that calling `go generate` does not produce any change
* Verifying that calling `go mod tidy` does not produce any change
* Verifying that the project builds successfully

Another common action between all repositories is one that upon a version of the project being tagged, releases it and generates a changelog for it.
This can be done using [`goreleaser`](https://github.com/goreleaser/goreleaser).

### Architecture Documents

When writing architecture documents, using diagrams is often the most efficient way to express a mental model.
For this purpose, we use a simple language to generate consistent diagrams in SVG formats, called [`nomnoml`](https://nomnoml.com/).
Using the [official website for the language](https://nomnoml.com/), diagrams can be written, previewed live and exported to SVG.

When using `nomnoml` diagrams, make sure to always include their sources within the repository:

```text
.
├── architecture.md
├── src
│   └── mydiagram.nomnoml
└── svg
    └── mydiagram.svg
```

!!! example ""

    <figure markdown>
      ![SVG Diagram](https://raw.githubusercontent.com/optakt/flow-dps/master/docs/svg/api.svg){ width="100%" }
    </figure>

    ```text
    #title: api
    #direction: bottom
    #background: #262626
    #stroke: #fff
    #fill: #888
  
    [<flow> Flow Network]
    [<external> Flow DPS Live]
    [<external> Flow DPS Indexer]
    [<indexer> Index]
    [<dps> DPS API]
    [<access> DPS Access API]
    [<rosetta> Rosetta API]

    [<label> SubmitTransaction]

    [Flow DPS Indexer]-->[Index]
    [Flow DPS Live]-->[Index]
    [Flow DPS Live]<--[Flow Network]
    [Index]->[DPS API]
    [DPS API]->[DPS Access API]
    [DPS API]->[Rosetta API]
    [SubmitTransaction]-[Rosetta API]
    [Flow Network]<-[SubmitTransaction]

    #.flow: fill=#262626 stroke=#00bff3 visual=ellipse dashed title=bold
    #.dps: fill=#262626 stroke=#fbb363 title=bold visual=receiver
    #.rosetta: fill=#262626 stroke=#fbb363 title=bold visual=transceiver
    #.access: fill=#262626 stroke=#fbb363 title=bold visual=transceiver
    #.indexer: fill=#262626 stroke=#fbb363 visual=database title=bold
    #.external: dashed
    ```

## Git Workflow

### Branches

Currently, we use a branch-based git workflow, where internal contributors have the permissions to create any number of branches on the repository. In most cases, a feature can be implemented on a single branch and a PR is then opened against master to merge it. For large features however, we use feature branches, against which we create multiple smaller PRs. This makes the review process more digestible and forces us to split large features into smaller chunks.

### Pull Requests

Each PR has to be related to a specific GitHub Issue, and the issue should be included in the body of the PR, by writing `Fixes #issueNumber`. This allows GitHub to automatically close the issue once the PR gets merged against master.

For a Pull Request to be considered ready, it needs to fulfill some requirements. It should:

* Have at least one reviewer
* Be [labelled](#labels) appropriately
* Successfully run the CI
* Have an up-to-date branch with the PR's target
* Update any tests, documentation etc. that is impacted by the PR's contents
* Not contain any leftover `FIXME` notes, and if `TODO` notes are added, those should be linked to a GitHub issue

Reviewers should only block a PR by `Requesting Changes` if one or more elements from the Pull Request must be changed before the PR can get merged. If a reviewer only gives feedback for nitpicks and style, for example, they should most likely just leave comments but not request changes. This keeps the reviewing process smoother.

Once a PR has been reviewed and approved by at least one reviewer, it can be merged. The only way we allow merging is by squashing all the PR's commits into a single one, as this keeps the `master` branch and the changelog clean and easy to follow.

!!! warning "Closing Pull Requests"

    When closing a Pull Request, make sure to always specify the reason for closing it by adding a comment to the PR while closing it.
    This will ensure that collaborators can follow the process and understand the reasons for the PR being closed, and ensures that the repository has a well-documented history.

    If you have issues with conflicts and/or your branch is messy, **NEVER** close the PR and delete the remote branch.
    Instead, simply create a new clean branch on which you cherry-pick your commits, clean up, and overwrite the original remote branch by using `git push --force`.
    This will update the PR and the previous history will be removed while preserving the original suggestion threads and PR metadata.
    Doing this also avoids duplicating information and having a confusing git history for the repository.

### Labels

Task prioritization is currently handled using labels:

* `must` means the task is of the highest priority and must be implemented as soon as possible.
* `should` means the task should be done at some point, but there is no urgency at the moment.
* `could` means the task could be done if we want to, but it is not required.

Each project then has its own custom labels for different types of issues and areas of the code.

### `go generate`

Anything that gets generated should ideally be generated by running `go generate ./...` at the root of the repository. For example, in Flow DPS, the protobuf files are currently generated this way. In the Flow Rosetta repository however, `go generate` is used to generate constants that are used by the API to return version information.

### Releases

Optakt projects use a GitHub Action that, whenever a new tag is pushed on master, compiles the binaries of the project for a given set of operating systems and architectures, computes a checksum and generates a changelog automatically.

When the action runs successfully, it automatically transforms the tag into a GitHub release, sets the description of the release as the generated changelog, and adds the precompiled binaries and checksums to the release as downloadable files.

## Documentation

* [Flow DPS](https://github.com/optakt/flow-dps)
    * [Introduction](https://github.com/optakt/flow-dps/blob/master/docs/introduction.md)
    * [Architecture](https://github.com/optakt/flow-dps/blob/master/docs/architecture.md)
    * [Database](https://github.com/optakt/flow-dps/blob/master/docs/database.md)
    * [API](https://github.com/optakt/flow-dps/blob/master/docs/dps-api.md)
    * [Snapshots](https://github.com/optakt/flow-dps/blob/master/docs/snapshots.md)
    * Binaries
        * [`flow-dps-client`](https://github.com/optakt/flow-dps/blob/master/cmd/flow-dps-client/README.md)
        * [`flow-dps-server`](https://github.com/optakt/flow-dps/blob/master/cmd/flow-dps-server/README.md)
        * [`flow-dps-live`](https://github.com/optakt/flow-dps/blob/master/cmd/flow-dps-live/README.md)
        * [`flow-dps-indexer`](https://github.com/optakt/flow-dps/blob/master/cmd/flow-dps-indexer/README.md)
        * [`dictionary-generator`](https://github.com/optakt/flow-dps/blob/master/cmd/dictionary-generator/README.md)
        * [`create-index-snapshot`](https://github.com/optakt/flow-dps/blob/master/cmd/create-index-snapshot/README.md)
        * [`restore-index-snapshot`](https://github.com/optakt/flow-dps/blob/master/cmd/restore-index-snapshot/README.md)
* [Flow DPS Rosetta](https://github.com/optakt/flow-dps-rosetta)
    * [API](https://github.com/optakt/flow-dps-rosetta/blob/master/docs/rosetta-api.md)
* [Flow DPS Access](https://github.com/optakt/flow-dps-access)
