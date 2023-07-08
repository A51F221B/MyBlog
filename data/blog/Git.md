---
title: Basics of Git
date: '2023-07-08'
tags: ['git', 'github', 'version control']
draft: false
summary: Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
---

## Git Workflow

What we do on daily basis when using git.We commit changes in repository.In git we have a special area called `staging Area` or `Index` which does not exist in most other version control systems.It is essentially what we are proposing for next snapshot.Staging area allows us to review our work before taking a snapshot.We can also un-stage changes from the staging area.The staging area is filled when we use the `git add` command.When we add the files using the above mentioned command , they are in the staging area i.e. their state is recorded but note that at this point in time no snapshot is taken yet.These are only the changes we are proposing to be committed. The snapshot is taken when we commit those added changes using `git commit -m "type message here"` command.

![image1](/static/images/git/imgs/20220311144857.png)

Now we commit these changes :

![image2](/static/images/git/imgs/20220311145131.png)

> One very important thing to note is that once the files are committed, the staging area does not become empty ! This means that it stores the file1,file2 until another set of files is added or these files are changed and added using `git add`

## Staging Area or Index

### Staging files

`git status` : tells you the status of repository

`git add [.] [*.extention] [filename]` : staging changes

### Committing changes

`git commit -m "type message"` : commit changes

if we do not use `-m` tag then git would consider that you want to enter a longer message, so it will open `COMMIT_EDITMSG` git file so you can add a longer version of the message.

![image3](/static/images/git/imgs/20220311153418.png)

#### Best practices

- Commit should not be too big or too small.
- We don't want to commit after very few changes.
- The whole point of committing is to have check points to which we can go back, if needed.
- Commit often, but sensibly.
- Only commit when you want a state to be recorded.
- Also commit things that are different , separately. for example you fixed a bug and a typo, u should commit them separately so if one of them needs to be changes or reversed , it can be done easily without affecting the other one.
- Make meaningful commit messages.
- Don't do too many things in one thing.

#### Skipping the staging area

Do this with caution.
`git commit -am "message"` this will directly add and commit the changes to the repository.

> File deletion and renaming can be done using simple unix commands or using file manager.

### Ignoring files and directories

There are some files or folder which we don't want git to keep track of. To do this we create a file called `.gitignore` which is a an extension of a blank name. This only works if we have not yet included those files in the staging area.To solve this problem we have to remove those files from staging area. To see what files are present in the staging area we can use the command `git ls-files`.We can use the `git rm --cached -r [file or folder]` .

At [Github ignore](https://github.com/github/gitignore) we can see various ignore templates or recommendations for different programming languages. We can use these ignore files rather than making our own every time.

### Short Status

`git status -s` : get a short , to the point status.

### Reviewing Staged & Un-Staged changes

`git diff --staged` : to see what we have in staging area which are going to be committed. But note that these visual are terminal based and difficult to understand.So we can use the GUI alternative such as `KDiff3`,`P4Merge`,`WinMerge (for windows only)` or `VSCODE`.

#### Using `VScode` as our diff tool

- First we have to tell git that we are using vscode as our default diff tool. To do this type :
  `git config --global diff.tool vscode`

- Next we need to tell git how to launch vscode.
  `git config --global difftool.vscode.cmd "code --wait --diff $LOCAL $REMOTE"`

> you need to define `code` as environment variable so OS knows we are referring to vscode

- To check
  `git config --global -e`

Now add the following

![[Pasted image 20220311165426.png]]

So that vscode knows these are the commands we are going to use to call it as a diff tool.

After doing all the above steps we can simply use :
`git difftool` or `git difftool --staged`

> We don't use difftools now a days as they are part of working environment by default.

### Viewing history

`git log` : checking all the history of files committed. To close log file press `ESC` and after that press `q` separately.

`git log --oneline` : for a short summary.

`git log --online --reverse` : for reverse order

#### Viewing history of a specific commit

We can use the `git log` command to get the logs.To show a specific commit we can use the commit unique identifier.

`git show [id]| [tree] | [tags]`

or we can use

`git show HEAD~1` to see the last commit.

`git ls-tree HEAD~1` : to see all the files and folders in a commit

### Un-staging files

`git restore --staged [file name]`

or to restore everything

`git restore --staged .`

```bash
╭─asif221b@Mac ~/Documents/Webploit ‹main●›
╰─$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
 modified:   docs/.DS_Store
 new file:   docs/Demo.pdf

╭─asif221b@Mac ~/Documents/Webploit ‹main●›
╰─$ git restore --staged .
╭─asif221b@Mac ~/Documents/Webploit ‹main●›
╰─$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
 modified:   docs/.DS_Store

Untracked files:
  (use "git add <file>..." to include in what will be committed)
 docs/Demo.pdf

no changes added to commit (use "git add" and/or "git commit -a")
```

### Discarding local changes

`git restore [filename]` : this will lead to restore the file , same as that which was stored in staging area.

#### Restoring deleted file

Suppose we remove a file and then restore it.
`git rm file1.py`

Now we check the status
`git status -s`

we see that we have a deleted file.

Now we commit changes
`git commit -m "delete file"`

But we decide to restore it, so we look at history
`git log --oneline` to see what needs to be restored.

`git restore --source=HEAD~1 file1.py`

> HEAD~1 means the last commit

## Branching

One of the most important topics in `Git`.
Concepts in branching include

- Use branches
- Compare branches
- merge branches
- Resolve conflicts
- Undo a faulty merge
- Essential tools (stashing, cherry picking)

Branching allows us to isolate work from our main branch so we can work without messing the main branch.We can merge branches with main branch (which contains stable code).
We want to separate our messy work from our stable or main branch.

### Creating branch

`git switch -C branch_name`

### Comparing branches

From time to time we would want to merge our different branches with the main branch.So, its important to know how to compare both branches to see the difference in both branches.

Suppose we have another branch named `bugfix/signup-form`

First we check what changes are we going to merge from `bugfix` branch to master.To see the logs of `bugfix` branch we will type
`git log master..bugfix/signup-form`

If we want to see the compare the `bugfix` branch with `master` we can use
`git diff master ..bugfix/signup-form`

To only see the difference in files we can use
`git diff --name-status branchname`

### Stashing

When we switch branch, git resets the current working directory to the last commit of the branch we switch to, this could lead to the previous work being deleted if we have not committed it in that branch.

In those situations, we can stash those changes (meaning storing them at a safe place) and complete them and commit later.
`git stash push -m "message here "`

And after that we switch to another branch
To stash everything
`git stash push -am "message"`

To check stashes : `git stash list`

now we can switch the branch
`git switch bugfix/signup-form`

to switch back to master again
`git switch master`

After moving back to master , we want to commit those changes we put in the stash earlier.
To see those changes
`git stash show name_of_stash`

to apply this stash
`git stash apply name_of_stash`

Now we have applied the stash , so we can remove the files from the stash to clean things up.
`git stash drop name_of_stash`

or to remove all stashes : `git stash clear`

### Merging

Merging simply means bringing changes from one branch to another.
There are two types of merges in git

- Fast-Forward Merges
- Three-way Merges

### Fast-Forward Merge

> In Git , a branch is just a pointer to a chain of commits, like head pointer pointing to different nodes.What this means is ,when we commit, the head pointer (master branch) will move to the latest commit (or latest node)

![image4](/static/images/git/imgs/20220314194433.png)

In this case above, both `bugfix` and `master` branch are pointing to the same commit.Now suppose we switch to `bugfix` branch and make few commits :

![[Pasted image 20220314194539.png]]

Now we want to bring these changes to master as well.What will happen in the backend is that master branch pointer move `fast-forward` and will start pointing to the `bugfix` node i.e. merging the two branches.

![image5](/static/images/git/imgs/20220314194704.png)

### Three-Way Merge

Now we will take an example for three way merge. Suppose our bug fix branch is 2 commits ahead of master. But instead of merging these changes with master like in the fast-forward merge, we decide to go back to the master branch and add few changes there as well.So the diagram becomes :

![image6](/static/images/git/imgs/20220314195128.png)

Now both branches are divert, we have some changes in master that donot exist in bugfix and vice versa. Now if we try to merge , git cannot move the `master` pointer fast forward to point it to the latest bug fix commit.In that case git creates a new commit that combines the changes from both the branches! cool , isn't it?

![image7](/static/images/git/imgs/20220314195422.png)

This new commit is called **Merge Commit**.

![image8](/static/images/git/imgs/20220314195519.png)

![image9](/static/images/git/imgs/20220314195602.png)

`git merge branchname` : this will merge the branch to the current branch u are in.

Some people only prefer three-way merge even if fast forward is possible.

### Conflict

Conflicts occur when

- Same line of code is changed differently in two different branches.
- If a file is changed in one branch, but is deleted in other branch.
- If a same file is added twice in two different branches but the content is different.

In these cases git cannot figure out how to merge those changes.

When there is a conflict we need to make changes manually.
