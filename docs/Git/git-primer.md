# Git Primer

Git can be a complex application, but for the focus of using it during the `mineOps` series we can focus on just a collection of terms and basic functionality.

## Definition of Terms

Working Directory - The `working directory` is the directory where you have content that you want to manage with `git`

Commit - A `commit` is a full snapshot of the contents of the `working directory` and of the files tracked by `git`. A `commit` keeps track of of these changes in a unique hash sometimes called a `SHA`. An example of this is: `3f51ebcb621890d47bbf3a3c42bec6bffa360d91`

Index - The `index` in Git is essentially a staging area. This is where snapshots of changes are placed via `git add` before they are `commited`. The `index` is crucial to Git as it sits between the `working directory` and all the various `commits` of the repository.

Branch - Git `branches` can be explained as simply a pointer to a specific `commit`.

## Common Commands

### Working Directory
* `git status` - Shows the currnet status of the `working directory`
* `git add <filename>` - Adds a file to the `index` from the `working directory`
* `git commit` - Starts the process of creating a `commit` based on the current `index`

### Remote Repository
* `git fetch` - Download objects from a remote repository
* `git pull` - Fetch from and integrate objects from a remote repository or another local branch
* `git push` - Update remote objects from the local `working directory` `commits`
