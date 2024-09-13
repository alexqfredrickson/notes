# git

Git repositories are databases with object state and indices as primary data sources.

## Object Stores

Object stores contain blobs, trees, commits, and tags.

Trees record blob identifiers, path names, and metadata about files. They represent one level of directory information. They can reference other trees recursively to create a hierarchical structure.

Commits hold commit information, and point to trees. They form directed acyclic graphs.

## Indices

An index is a temporary binary file which describes the directory structure of an entire repository.

## Branches

Branches are based off of commits.

## Commands

### `git branch`

Creates or lists or deletes branches.

### `git merge`

Merges branches.

### `git reset`

Changes the `HEAD` reference to a given commit. `git reset --soft` do the same but the index and working directory contents will not change. 

### `git checkout`

Switches to a branch.

### `git status`

Queries the index state.

### `git rm`

Removes a file from a working directory, and an index.

### `git add`

Adds a file to the working directory, and an index.

### `git commit`

Creates a commit to stage tracked files.

### `git blame`

Shares who last modified each line of a file.

## "Git Flow" Branching

"Git Flow" branching is an old branching model. It kind of largely lost the fight across the industry to "trunk-based" branching.


![](git-flow.png)

## Errata

`HEAD` refers to the most recent commit.

