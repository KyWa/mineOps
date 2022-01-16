# Git for mineOps

## Installing Git

As with Ansible there are a few ways to install Git, but the package manager of your system will probably be the best bet:

### RHEL/Fedora/CentOS

```sh
$ yum install -y git
```

### Debian/Ubuntu*

```sh
$ apt get -y git
```

Once you have Git installed, verify its installed and in your path. If you have verified you have Git installed we can proceed to the next section.

```sh
$ git --version
git version 2.32.0
```

There isn't much else that needs to be done here, but we will dig in further into some of the configuration options of Git as we learn about it.

## Understanding Git

If you are looking for an extreme deep dive and not one just focused on getting going enough for the sake of our "Business", I highly recommend taking a look at this YouTube Channel from [Dan Gitschooldude](https://www.youtube.com/watch?v=OZEGnam2M9s&list=PLu-nSsOS6FRIg52MWrd7C_qSnQp3ZoHwW&ab_channel=DanGitschooldude). He does an amazing job and goes over each topic (and some more advanced uses and tools) in greater detail than we will get to in this guide.

For the Git basics that are recommended for our business, I've created a [quick primer](./git-primer.md) for Git and its main commands and phrases used, but we will also go over some of these in detail in the next few sections. I would recommend if you are unfamiiliar with Git to take a quick look over that little primer just to get familiar with some of the terms we will use.

### Working Directory - Local Repository

The core feature of Git that most people will ultimately use can be outlined easily. There is the "working directory", which for our purposes will be the directory where we keep our Ansible code from the last section. This "working directory" is also called the "local repository". This local repository stores your files contained within, but most importantly the history of these files as a whole. To make a directory a Git repository, issue this command in the "top level" of the directory containing the files you wish Git to keep track of. For us in our example, we will be using the "modular_ansible" directory as our "top level":

```sh
$ pwd
~/modular_ansible

$ git init
Initialized empty Git repository in ~/modular_ansible/.git/
```

Once the above has been run our "working directory" where we have all of our files and directories such as `playbooks`, `roles` and `inventory` now exist as part of the local repository, but these files aren't being tracked by Git yet. Let's take a look at a command `git status` and see what it tells us:

```sh
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ansible.cfg
        inventory/
        playbooks/
        roles/

nothing added to commit but untracked files present (use "git add" to track)
```

With the git status command we can get some very important information about our new local repository. One nice thing about Git is that it is very good (sometimes annoyingly so) about telling you what is going on and what you need to do to get a clean and up-to date repository.

* `On branch master` - This first line tells us the name of the branch we are on (more on this topic later)
* `No commits yet` - Something you will only ever see on a fresh Git repository via git init
* `Untracked files` - Now this is where Git starts to show its power. This output shows all the files in the working directory that are not part of the `index`. We will discuss the index further in one of the next sections.
* `nothing added to commit but untracked files present (use "git add" to track)` - This is Git telling us that there is nothing committed to the `index`, but it is aware of new files that are untracked. Git is also kind enough to tell us what we need to do to track these files.

`git status` is a command you will get familiar with very quickly as it tells you the "state" of your local repository. Let's move on to the next topic, `commits`, but to learn about `commits` we need to understand the `index`.

### Index

The `index` in Git is a "staging area" for the files in the working directory. In the `git status` output from earlier, we noticed Git told us this: `Untracked files` and proceeded to list the files and directories in our local repository. These files are not part of the `index`, but are part of the working directory. If we issue the `git add` command from earlier that Git gave us, let's see what our `git status` looks like and we can see the `index` change.

```sh
$ git add ansible.cfg
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   ansible.cfg

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory/
        playbooks/
        roles/
```

Notice we have a new part of output,Changes to be committed. This status gives us some very interesting information:

* `use "git rm --cached <file>..." to unstage` - This shows us that we have a new staged file as part of our `index`
* `new file: ansible.cfg` - Git tells us about the file we added via `git add` and is the file thats part of our `index`
 
Let's go ahead and do a `git add` on the rest of the files in our working directory:

```sh
$ git add .
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   ansible.cfg
        new file:   inventory/group_vars/customer-0/config.yml
        new file:   inventory/group_vars/customer-0/server-properties.yml
        new file:   inventory/group_vars/customer-1/config.yml
        new file:   inventory/group_vars/customer-1/server-properties.yml
        new file:   inventory/group_vars/minecraft/config.yml
        new file:   inventory/group_vars/minecraft/server-properties.yml
        new file:   inventory/host_vars/mineops-0.kywa.io/config.yml
        new file:   inventory/host_vars/mineops-0.kywa.io/server-properites.yml
        new file:   inventory/host_vars/mineops-1.kywa.io/config.yml
        new file:   inventory/host_vars/mineops-1.kywa.io/server-properites.yml
        new file:   inventory/hosts
        new file:   playbooks/minecraft-backup.yml
        new file:   playbooks/minecraft-config-change.yml
        new file:   playbooks/minecraft-install.yml
        new file:   playbooks/minecraft-restore.yml
        new file:   playbooks/minecraft-uninstall.yml
        new file:   roles/minecraft/files/eula.txt
        new file:   roles/minecraft/tasks/backup.yml
        new file:   roles/minecraft/tasks/config-change.yml
        new file:   roles/minecraft/tasks/main.yml
        new file:   roles/minecraft/tasks/restore.yml
        new file:   roles/minecraft/tasks/server-prep.yml
        new file:   roles/minecraft/tasks/start-server.yml
        new file:   roles/minecraft/tasks/uninstall.yml
        new file:   roles/minecraft/templates/minecraft.service.j2
        new file:   roles/minecraft/templates/server.properties.j2
```

Our `index` has grown and we no longer have any `Untracked files` in our working directory and our entire repository. The only item we have left in our `git status` that Git is telling us about is `Changes to be committed`. With that, lets move on to learning about commits.

### Commits

Commits are "snapshot" of the working directory and the files contained inside that Git is keeping track of. As we saw in the `git status` above in the last section, Git told us we had `Changes to be committed` based on the files we added to the `index`. If we issue the `git commit` command on its own, we will be brought to your default text editor. This will have you input what is called a `commit message` which is human slang for "what are you committing to this local repository?". There is another method using `git commit` where you do not go into an editor, but are able to pass in the `commit message` in-line via: `git commit -m "my message"`.

For the sake of this guide and getting going, lets use the quicker method:

```sh
$ git commit -m "initial commit, adding repo files"
[master (root-commit) 7ba22bd] initial commit, adding repo files
 Committer: Kyle Walker <kylewalker@Kyles-Mac-mini.kywa.io>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

27 files changed, 437 insertions(+)
 create mode 100644 ansible.cfg
 create mode 100644 inventory/group_vars/customer-0/config.yml
 create mode 100644 inventory/group_vars/customer-0/server-properties.yml
 create mode 100644 inventory/group_vars/customer-1/config.yml
 create mode 100644 inventory/group_vars/customer-1/server-properties.yml
 create mode 100644 inventory/group_vars/minecraft/config.yml
 create mode 100644 inventory/group_vars/minecraft/server-properties.yml
 create mode 100644 inventory/host_vars/mineops-0.kywa.io/config.yml
 create mode 100644 inventory/host_vars/mineops-0.kywa.io/server-properites.yml
 create mode 100644 inventory/host_vars/mineops-1.kywa.io/config.yml
 create mode 100644 inventory/host_vars/mineops-1.kywa.io/server-properites.yml
 create mode 100644 inventory/hosts
 create mode 100644 playbooks/minecraft-backup.yml
 create mode 100644 playbooks/minecraft-config-change.yml
 create mode 100644 playbooks/minecraft-install.yml
 create mode 100644 playbooks/minecraft-restore.yml
 create mode 100644 playbooks/minecraft-uninstall.yml
 create mode 100644 roles/minecraft/files/eula.txt
 create mode 100644 roles/minecraft/tasks/backup.yml
 create mode 100644 roles/minecraft/tasks/config-change.yml
 create mode 100644 roles/minecraft/tasks/main.yml
 create mode 100644 roles/minecraft/tasks/restore.yml
 create mode 100644 roles/minecraft/tasks/server-prep.yml
 create mode 100644 roles/minecraft/tasks/start-server.yml
 create mode 100644 roles/minecraft/tasks/uninstall.yml
 create mode 100644 roles/minecraft/templates/minecraft.service.j2
 create mode 100644 roles/minecraft/templates/server.properties.j2
 
$ git status
On branch master
nothing to commit, working tree clean
```

If we look at the top of the output, we will see a message stating who the `Committer` was. When using Git for the first time (and depending on version this output may be different), Git needs to know who is actually `committing` these changes. This information has important use later on and we will discuss the `git config` command in a few sections. The `git commit` command we ran primarily tells us how many files were "changed" in the local repository and how many "insertions". As we will see later in this guide, when we edit files and remove lines, we will see "deletions" as well as "insertions". This is a handy way of having a 30k foot view of the repository and its latest `commit` changes. `Commit` messages are very important as they can/should be used to outline what is contained within a `commit`. Our first `commit` we said it was the "initial commit, adding repo files". Now, how do we view this commit message? Through `git log` we can see all `commits` in a local repository and their `commit` messages. Here is the `git log` from our test repo:

```sh
$ git log
commit 7ba22bd8923180278deaf938785841b740d9eae4 (HEAD -> master)
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 14:34:34 2022 -0600

    initial commit, adding repo files
```

We have some new items here from Git, one of the largest being the long string you see: `7ba22bd8923180278deaf938785841b740d9eae4`. This is what is called the hash or SHA of the `commit`. This is a unique hash for this repository to identify "that point in history". The `commit message` is for humans to be able to have some form of identifier for a specific `commit`, although unlike the `commit` hash, a `commit` message doesn't have to be unique. We've discussed a lot about `commits` and mentioned them being a pointer to a specific place in time, so let's move on to `branches` which are pointers themselves, but to a specific `commit` history.

### Branches

As mentioned above Git `branches` as essentially a pointer to a specific `commit`. If all you ever do is use Git locally without ever working with a team or a remote repository, Git `branches` may not make a lot of sense at first. However, Git `branches` have some amazing benefits, but can be confusing if you aren't aware the current state of your local repository.

We see in our `git status` that our current `branch` is master. This happens to also be the default `branch` for most if not all Git repositories, but it doesn't have to be. You can actually set the default `branch` to be whatever you'd like it to be called. Some have chosen to go with "main" or "trunk" or whatever they feel best describes the "core" of their repository. In our `git log` command in the previous section, we see a single `commit hash`. Our master `branch` in this local repository is currently targeting that `commit`. Remember when we first did `git status` and it said this:

```sh
$ git status
On branch master

No commits yet
```

This message `No commits yet` is telling us that currently the master `branch` isn't targeting anything, but this is no longer the case since our first `commit`. Our `git status` output should now look like this:

```sh
$ git status
On branch master
nothing to commit, working tree clean
```

What if by now we have a few customers on our platform with their Minecraft Servers and we are going to try to update our platform. In theory we would have a development server to test these items on. For the sake of argument, I mean both the virtual server and the newer Minecraft Server we would have a development version of. In our Ansible examples we had just customer servers in place with some global variables for all Minecraft Servers to use. If we had a development environment we could have unique vars for those dev servers, or we could have a dev `branch` in Git that is used to test these changes before they get merged into the master `branch`.

#### Note

In this particular example, Ansible on its own may not be the best example for using a dev `branch` as we run all of this Ansible by hand for managing these Minecraft Servers. If we were using something like [Ansible Tower](https://www.ansible.com/products/controller) (now called Red Hat Ansible Automation Platform) or [AWX](https://github.com/ansible/awx) which allows you to automate and schedule your Ansible playbooks, this would make more sense. Later in this series, we will be discussing different applications which may benefit more from unique branches.

### Git Config

In the `commits` section we spoke briefly about the `git config`, but went into no details about what it is. When you issue your first `git config --global` command this will create a hidden file in your home directory called `.gitconfig`.

```sh
$ cat ~/.gitconfig
[user]
name = Kyle Walker
email = kywadev@gmail.com
[pull]
    rebase = false
[init]
    defaultBranch = master
```

The output above is my `.gitconfig` for my local machine. To setup your user (as mentioned above during the `commit` section) you can issue these commands to generate do this for you:

* `git config --global user.name "Kyle Walker"`
* `git config --global user.email "kywadev@gmail.com"`

There are many other `git config` options you may end up using, but having your user set to identify you as the author of `commits` is all most people go with. If you wish to review all configuration options, the official Git docs have some very detailed information: https://git-scm.com/docs/git-config

## Using Git

We've discussed the major concepts and commands of working with a local Git repository, but before we move on I'd like to go over what a typical "workflow" is like for someone using Git. In our business we just got a new customer who needs a Minecraft Server. Let's see what this process would be like now that we use Git.

First we would create our Ansible files in the same fashion as was done before, we would update the `inventory` file to include this new Minecraft Server. We would also create the required variable files in `group_vars` and `host_vars` for this new customer. Once we have done that, it's time to run our Ansible playbook and then add these files to Git. For the sake of it being "just us" on this team we will use these files being added to Git as our process of "the work has been done". Let's see what our local repository looks like now:

```sh
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   inventory/hosts

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        inventory/group_vars/customer-2/
        inventory/host_vars/mineops-2.kywa.io/

no changes added to commit (use "git add" and/or "git commit -a")
```

As we can see, we now have a different type of `git status`, but we've seen all of this before when we did our first `git add` of `ansible.cfg`. We have Changes not staged for `commit`, this being the pre-existing and already tracked inventory hosts file, and we have more `Untracked files` which we added for our new customer.

Before we add any files to our local repository, let's take a look at a new Git command called `git diff` and see what the inventory hosts file shows us.

```sh
$ git diff inventory/hosts
diff --git a/inventory/hosts b/inventory/hosts
index 510d09b..181a8ea 100644
--- a/inventory/hosts
+++ b/inventory/hosts
@@ -1,9 +1,13 @@
 [minecraft:children]
 customer-0
 customer-1
+customer-2

 [customer-0]
 mineops-0.kywa.io

 [customer-1]
 mineops-1.kywa.io
+
+[customer-2]
+mineops-2.kywa.io
```

`git diff` shows us the additions (remember this from earlier during our first `commit`?) via the `+` sign on each line of the inventory hosts file. This is quite handy in case you were working on something, had to step away, lost your place and were unsure of where you left off. It has more uses later on as we will see in this guide.

Once you've created the new files to be tracked in Git, you would then add them via `git add`:

```sh
$ git add -A
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   inventory/group_vars/customer-2/config.yml
        new file:   inventory/host_vars/mineops-2.kywa.io/config.yml
        new file:   inventory/host_vars/mineops-2.kywa.io/server-properites.yml
        modified:   inventory/hosts
```

We are now ready in theory to create another `commit` to this repository. This time, lets just use `git commit` with no `-m` flag being passed in. For my system I have an environment variable set called `EDITOR=vim`. If you do not want to alter your systems default editor via this environment variable, you can add something to your Git configuration file through this command `git config --global core.editor "vim"`. The other alternative is an environment variable called `GIT_EDITOR` which can be set to any editor as with the `EDITOR` env var. When we run `git commit`, we are met with a screen that looks somewhat like this (note I am using `vim` as my `EDITOR`):

```sh

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#       new file:   inventory/group_vars/customer-2/config.yml
#       new file:   inventory/host_vars/mineops-2.kywa.io/config.yml
#       new file:   inventory/host_vars/mineops-2.kywa.io/server-properites.yml
#       modified:   inventory/hosts
#
```

Git gives us a clear idea of what it is wanting, a `commit message`, but with this we are able to more easily create multi-line `commit messages` for the sake of being explicit. This isn't required, but its something to be aware of if you grow your team and you require more detailed `commit messages` for some tools in the future such as [git-chglog](https://github.com/git-chglog/git-chglog). As for the `commit message` in this format, you can create them like this after running `git commit`:

```sh
Adding new customer
- customerID 98765 via ticket #00005
- deployed by Kyle

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#       new file:   inventory/group_vars/customer-2/config.yml
#       new file:   inventory/host_vars/mineops-2.kywa.io/config.yml
#       new file:   inventory/host_vars/mineops-2.kywa.io/server-properites.yml
#       modified:   inventory/hosts
#
```

Once you are satisfied with your `commit message` you can write and quit out of the editor (in `vim` this is `:wq`). And the output afterwards is the same as any `commit`, we can also check the `git log` and see our multi-line `commit message`:

```sh
$ git commit
[master 552b08d] Adding new customer - customerID 98765 via ticket #00005 - deployed by Kyle
 4 files changed, 9 insertions(+)
 create mode 100644 inventory/group_vars/customer-2/config.yml
 create mode 100644 inventory/host_vars/mineops-2.kywa.io/config.yml
 create mode 100644 inventory/host_vars/mineops-2.kywa.io/server-properites.yml
 
$ git log
commit 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4 (HEAD -> master)
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 15:24:35 2022 -0600

    Adding new customer
    - customerID 98765 via ticket #00005
    - deployed by Kyle

commit 7ba22bd8923180278deaf938785841b740d9eae4
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 14:34:34 2022 -0600

    initial commit, adding repo files
```

As for using Git, there is really only one other item to address as it relates to a local repository and that is reverting `commits` with `git revert`. Depending on the scope and size of changes in the `commit` you wish to `revert`, you could either edit the files (if it was a small change) and create another `commit`, or you could use `git revert` if it was a large amount of files and changes. Please note that `git revert` can only be used to `revert commits`, but by doing so creates a new `commit` marking the `revert`. You will need to get the `commit hash` via `git log` or some other method and then you can run `git revert 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4`. Let's see what that looks like and what happens when we run this:

```sh
$ git revert 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4
Revert "Adding new customer"

This reverts commit 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#       deleted:    inventory/group_vars/customer-2/config.yml
#       deleted:    inventory/host_vars/mineops-2.kywa.io/config.yml
#       deleted:    inventory/host_vars/mineops-2.kywa.io/server-properites.yml
#       modified:   inventory/hosts
#

$ git revert 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4
Removing inventory/host_vars/mineops-2.kywa.io/server-properites.yml
Removing inventory/host_vars/mineops-2.kywa.io/config.yml
Removing inventory/group_vars/customer-2/config.yml
[master c6f7b7b] Revert "Adding new customer"
 4 files changed, 9 deletions(-)
 delete mode 100644 inventory/group_vars/customer-2/config.yml
 delete mode 100644 inventory/host_vars/mineops-2.kywa.io/config.yml
 delete mode 100644 inventory/host_vars/mineops-2.kywa.io/server-properites.yml

$ git status
On branch master
nothing to commit, working tree clean

$ git log
commit c6f7b7b392a52edbbde4be26e8ed00c8487a9403 (HEAD -> master)
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 15:45:28 2022 -0600

    Revert "Adding new customer"

    This reverts commit 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4.

commit 552b08d221fc9c6a1875f65d8bcfae7c1bf0aaa4
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 15:24:35 2022 -0600

    Adding new customer
    - customerID 98765 via ticket #00005
    - deployed by Kyle

commit 7ba22bd8923180278deaf938785841b740d9eae4
Author: Kyle Walker <kywadev@gmail.com>
Date:   Mon Jan 3 14:34:34 2022 -0600

    initial commit, adding repo files
```

Note we have a new `commit` showing the revert. If you do not wish to have revert `commits` in your `git log` (I would recommend that you do keep them there for the sake of history especially as you move to remote repositories and working with teams), you can use the `git reset` command which we are going to go over in quickly below.

If you need to revert changes in your `index` to "rollback" changes you made to existing files you can use `git reset --hard <COMMIT HASH>`. This can potentially be destructive so you should probably use `git checkout .` to reset your `index` to `HEAD` (latest `commit` for your `branch`). Please note this only works if you haven't `commited` any changes locally yet.

That wraps up just about everything you would typically do with Git in a local repository. It is time to learn about remote repositories and then working with a team once we have a remote repository working.

## Remote Repositories

So we've talked about Git a lot, but not once have I mentioned GitHub, GitLab, Bitbucket or any other of the big names in the Git repository world and that was on purpose. People often confuse GitHub and Git as being just different versions of the same thing and this couldn't be further from the truth. 

### Setting up a Remote Repository

Each of the big 3 source-code repository hosting services GitHub, GitLab and Bitbucket ultimately function as a remote repository. Setting up a new repository is roughly the same in each one, but we will not outline how to do that for each one. All that you really need is an account and then a name for the new repository. Here are links to each of the docs to create a remote repository:

* GitHub - https://docs.github.com/en/get-started/quickstart/create-a-repo
* GitLab - https://docs.gitlab.com/ee/user/project/working_with_projects.html#create-a-project
* BitBucket - https://support.atlassian.com/bitbucket-cloud/docs/create-a-git-repository/

When a new repository is set up, each of the big 3 will give you instructions on how to use the new remote repository. For this guide I will be using GitHub, but any repository hosting will work. The setup instructions it gives you outlines how to create a new repository from the command line to use this new remote repository. We do not need to do this as we have our own local repository already. Thankfully there are instructions for this use case as well. We will dive into this in the next section.

### Adding a Remote Location to your Local Repository

Once you have created a new remote repository in GitHub/GitLab/BitBucket, you will need to let your local repository know about this remote location. Thankfully you get these instructions once you setup a remote repository:

```sh
$ git remote add origin git@github.com:KyWa/mineops-modular_ansible.git
```

There are other instructions given but these will be outlined in the following section. The only item to note about the above command is the name `origin`. This is the name used for the remote repository that your local repository can work with. You can actually name `origin` whatever you wish in case you end up working with multiple Remote Repositories (which yes you can do, but we wont be worrying about that here). Just know that if for your local repository if you wanted to name it `booger` instead you could.

### Working with a Remote Repository

In your local repo once you have run the above command to add a `remote` target we can now begin to work with it. First let's use `git push` to push our local repository to the `remote`.

```sh
$ git push
fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin master

$ git push --set-upstream origin master
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Enumerating objects: 54, done.
Counting objects: 100% (54/54), done.
Delta compression using up to 12 threads
Compressing objects: 100% (45/45), done.
Writing objects: 100% (54/54), 7.43 KiB | 1.86 MiB/s, done.
Total 54 (delta 6), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (6/6), done.
To github.com:KyWa/mineops-modular_ansible.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

Running `git push` on its own at the moment throws an error that the current branch master has no upstream branch. We can remedy this with the command that Git gives us (see how helpful Git can be?). Let's break down what we this command is doing. 

* `git push` - pushing local changes to an upstream branch
* `--set-upstream` - starts the setup of targeting the upstream branch
* `origin` - the remote target used by the --set-upstream flag
* `master` - the name of the remote branch to push your local branch to

So we now have content in our remote repository, this is a huge step forward for our company. We no longer have our files being stored on our local machine and have Git tracking our files locally, with the big plus of being stored in GitHub (our remote repository).

Some of the other tasks you will do when working with a remote repository is getting changes from that remote repository. Now at the moment its just us in this company so there won't be much to get from our remote repository as we are the only ones doing any work. In the next section when we start "working with a team" we will dig more into pulling code from a remote repository after our "teammates" have made changes we need.

## Working With a Team

Our company has grown and we need another team member. Once this new team member has gotten set up they will start working and adding code to our remote repository. We now need to get this code so we can have these changes locally to ensure we don't have any issues with our own changes. There are a few methods to do this with the more common one being `git pull`. This command will technically do 2 other commands a `git fetch` and a `git merge`. `Fetch` will "fetch" all the objects and references in "remote" if there are any, but will not be part of your local Git history just yet. If you were to run a `git merge` once you have this (and there were changes in the upstream branch matching your local branch) it will update your local history by "merging" each history together (local and remote). The more simpler way is `git pull` which will handle both of these for you and is the most common path used to do this.

A note about working with a team and pulling code down or pushing new code. There can be things known as a "Merge Conflict". These are not fun, but we will not being dealing with these and you may not either if you ensure you have a clean "working directory" prior to doing a push or pull. For more information around "Merge Conflicts" check out this article: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/about-merge-conflicts

Now this is where more fun begins, but also where Git truly can shine if using a remote repository as outlined above. Why did I say its fun? Enter `git blame`. Despite its name, this doesn't send an email to your co-worker to tell them they did something wrong. What it does do however is show you all the lines of a file that Git tracks and who edited which line based on the current `commit`.

```sh
$ git blame ansible.cfg
^7ba22bd (Kyle Walker 2022-01-04 14:34:34 -0600 1) [defaults]
^7ba22bd (Kyle Walker 2022-01-04 14:34:34 -0600 2) roles_path = roles
^7ba22bd (Kyle Walker 2022-01-04 14:34:34 -0600 3) retry_files_save_path = /tmp
```

Now obviously there is more to working with a team through Git and `git blame` and `git pull`, but for our little company we have covered all we need for using Git. There isn't much more to discuss, but there will be more topics come up in later parts of this series.

## Git Extra Credit

If you really want to get the most Git, there are tools you can use to make life easier and expand on the functionality of Git. I will list these below, but none are required to use Git.

* [tig](https://opensource.com/article/19/6/what-tig) - A text-based repository browser that really helps "view" a repository
* [git-chglog](https://github.com/git-chglog/git-chglog) - CHANGELOG creator for your repository based on `commits`
* [GitHub Desktop](https://desktop.github.com/) - GUI repository manager from GitHub

## Moving On

In the next installment in the series we are going to move away from Ansible and traditional "servers" and start our journey to containers because our CEO heard the buzzword "containers" so we need to migrate to "containers". And if history has taught us anything, they are going to hear about another buzzword and we will probably have to migrate to that afterwards as well. See you in the next installment!
