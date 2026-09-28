# Git/Github usage

Here is a quick introduction to Git and Github usage to make sure that you understand how to collaborate on ARCUS code.

A deeper dive into git and its commands can be found on the [official documentation](https://git-scm.com/docs).

Git works using commands in the terminal. If you feel uncomfortable writing all these commands by hand, many tools are available as a replacement to achieve the exact same goal :
* [Github desktop](https://github.com/apps/desktop). A user-friendly git tool to manage all your code with an interface rather than commands (recommended for beginners).
* [VSCode built-in source control](https://code.visualstudio.com/docs/sourcecontrol/overview). If you are coding using VSCode, it already has a build-in source control tool to help you manage all your Git actions. It offers the same features without the need of downloading another app.

## Setting up git

If you have already [setup your environnment](../code-and-simulation/setup_up.md), you should have cloned the repo. However, to start pushing code, you must setup your git information to link your commits to your Github account. Use these commands by filling in your credentials :

```bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR.EMAIL@EXAMPLE.COM"
```

## Workflow

This section will explain the basic workflow of a task completion through the use of Git and Github.

### Updating

Before making any changes, always make sure your version of the code is up to date with the Github. Use this command to fetch updates :
```bash
git pull
```

### Branches

Git branches allow people to work simultaneously on different features of the code. Generally, one branch represents one feature or addition. For example, in our case, one branch could represent a specific task. Use this command to create a branch :

```bash
git branch NEW_BRANCH_NAME
```

You can then move to this branch :

```bash
git switch BRANCH_NAME
```

You can view which branch you are currently in using :

```bash
git branch
```

or

```bash
git status
```

### Making changes

Your code has 3 main states in which it can be in the workflow :

#### Modified

You modified the code to fix a bug, add a feature, etc. Git automatically detects that a change has been made.

#### Staged

You can **stage** a file in order to "prepare" a file to be committed. It is useful if you do not want to commit all the files that you modified. It is achieved using this command :

```bash
git add FILENAME
```

or, to stage all changes at once :

```bash
git add .
```

#### Committed

Once files are staged, you can **commit** them. A commit is essentially a "save" of the current state of the code. This helps with the code history and is the main feature of Git. You are able to go back to a commit to view a previous state of the code. You can commit files using :

```bash
git commit -m "RELEVANT MESSAGE"
```

It is important that you choose your commit messages wisely as they will be the only information available to trace back previous versions of the code. A commit message contains a brief description of the changes that were made and the new features that were added in this exact commit.

Feel free to make commits as you need. They are very helpful to trace back changes and prevent loss of progress.

### Pushing your changes

Commits are created locally on your computer. To send them to the Github for everyone to see your changes, you need to **push** your commits. This is made using :

```bash
git push
```

If you are pushing from a new, locally created branch, you may need to use a more specific command :

```bash
git push --set-upstream origin NEW_BRANCH_NAME
```

### Merging a branch

Once a task/feature is completed and you are ready to send your code for review, you will need to create a **Pull request**. This will allow another member to review your code, suggest improvements and approve it before adding it to the `main` branch.

This is done directly on Github, on the repo of the branch to be merged, in the `Pull request` section. Create the pull request using the `New pull request` button, add a relevant title and description (if needed) and follow the steps to create the pull request. You will be able to assign reviewers to it.

The reviewers will be able to add comments to your code to suggest improvements or corrections. You will be notified of those. The reviewers can request changes before fully approving the merge.

If coding on the `arcus` repo, your code will have to pass the **CI/CD** pipeline to be merged to main. This is essentially an automated verification/testing to ensure code functionnality and quality. You can exexute the `scripts/lint-apply.sh` script to automatically apply lint formatting to your code to pass these tests.