# Git Notes

## Files

### `.gitignore`

List of files/directories that should not be tracked.

* Each line indicates a file/directory to ignore
* Directories should be specified with a trailing forward slash (`/`)
* Children of directories listed in `.gitignore` will also be ignored
* Globs work
* Comments can be added by beginning a line with a pound sign (`#`)
* Pattern that start with exclamation point (`!`) are negated

### `.git/HEAD`

Stores a reference to the current commit.

### `.git/objects/`

Files in this directory are git objects. These are `blob`s for files and `tree`s for files. Everything is identified by a 40 character SHA-1 hash. The first two characters in the hash identify the folder and the last 38 are the filename. Hashes are generated using a header and the contents of the object to be hashed.

### `.git/index`

Information about the current staging area.

### `.git/refs/`

References to branches and tags (local and remote). These are short strings that are easier to remember than SHA-1 hashes.

## Objects

### blob

A file

### tree

A directory

### commit

Like a tree, but the top level is the SHA-1 hash of a tree, information about the author, when the commit was made, and a commit message

## Commands

### init

```bash
git init
```

Create a new git repository. This will add a `.git` folder to the directory.

### clone

```bash
git clone git://repository-location/repo.git [name]
```

Create a copy of any existing repo. If `name` is provided, the repo will be copied into a directory with the name `name`. Otherwise, the directory will be named using the repository's name (e.g. `repo` from `repo.git`).

### status

```bash
git status
```

Display the current branch, staged files, modified files (tracked, and modified since last commit, but not staged), and untracked files.

### add

```bash
git add <name>
```

Stages (and tracks if previously untracked) the provided file(s). Directories are added recursively.

### diff

```bash
git diff
```

Display the lines in each file that have changed since the last commit.

`git diff` only shows the diff for files that have not been staged. To see the diff for staged files, pass the `--staged` flag.

### commit

```bash
git commit
```

Create a new commit object for the files that have been staged.

Calling `git commit` on its own will launch a text editor for you to write a commit message.

Alternatively, you can use the `-m` flag to pass a message inline. These can also be chained together.

```bash
git commit -m "This is the commit message" -m "This is an extra commit message"
```

The `-a` flag will add any tracked files to the staging area without having to use `git add`.

```bash
git commit -a -m "Look ma, no git add"
```

The `--amend` flag allows you to "fix" the last commit. This is useful if you forget to stage a file or need to change the commit message.

### rm

```bash
git rm <name>
```

Untrack a file/directory and also remove it from the local filesystem. Globs can also be used here.

The `-f` flag must be used to remove a modified and staged file.

The `--cached` flag will untrack a file but will not remove it from the filesystem.

### mv

```bash
git mv <from_name> <to_name>
```

Move/rename a file without having to explicitly having to untrack the old name and start tracking the new one.

### log

```bash
git log
```

Display commits on the current branch.

* `-p` shows the diff for each commit.
* `--stat` shows file diff stats for each commit
* `--pretty=oneline|short|full|fuller|format:"..."` changes how each commit is displayed
* `--graph` will add add ASCII to display commits as a graph
* `--abbrev-commit` only shows the first 7 SHA-1 hash characters
* `--oneline` combintes `--pretty=oneline` and `--abbrev-commit`
* `-<n>` (where `n` is a number) shows only the last `n` commits

### reset

```bash
git reset HEAD <filename>
```

Unstage a staged file.


### checkout

```bash
git checkout -- <filename>
```

Resets a file to to its state from the last commit.

### remote

```bash
git remote
```

Display the name of remote repositories. `-v` flag shows expanded version.

```bash
git remote add <name> <url>
```

Add a remote repository with the reference name `name`. When you `git clone`, an `origin` remote will automatically be created.

```bash
git remote show <name>
```

Displays information about a remote repository

```bash
git remote rename <current> <new>
```

```bash
git remote rm <name>
```

Removes the reference to a remote repository

```bash
git remote prune origin
```

Remove remote branch references

### fetch

```bash
git fetch <name>
```

Downloads the data from a remote branch.

### pull

```bash
git pull <name>
```

Fetch and merge a remote repo

### push

```bash
git push <remote name> <branch name or tag name>
```

Uploads the provided branch/tag to the remote repository.

```bash
git push origin --tags
```

You can also upload all tags using the `--tags` flag.

### tag

```bash
git tag
```

List available tags. You can also use the `-l` flag to filter tags.

```bash
git tag <name>
```

Create a tag which the provided `name`.

```bash
git tag -a <name> -m "message"
```

Annotated tags can be created using the `-a` flag. These tags have a message attached using the `-m` flag.

```bash
git tag <name> <hash>
```

You can add a tag to a previous commit by providing that commit's hash.
