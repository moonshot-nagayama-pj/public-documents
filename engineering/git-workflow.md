# Our Git workflow

Our team uses Git and GitHub to coordinate, manage, and track the development of our software.

There are many different ways to use these tools. Some Git commands can be destructive, or difficult to use in a team environment. Team members may have different levels of experience and ability using Git, but should all be able to help contribute to the code.

It is therefore important that we all have a shared understanding of how these tools should be used, so that we can work together without causing problems for other team members.

## Required knowledge

### Git

[_Pro Git_](https://git-scm.com/book/en/v2), the official Git book, is free and available in many languages.

Reading at least the first three chapters, through the end of "Git Branching," is strongly recommended.

### GitHub

It is important to understand the difference between Git and GitHub.

GitHub is optimized for a specific way of using Git. GitHub invented terms like "fork" and "pull request;" these are not a part of Git itself.

The official GitHub documentation is located in [GitHub Docs](https://docs.github.com/en). It is best to read _Pro Git_ before GitHub Docs.

### Unix command-line

The rest of this guide will use the `git` command-line interface. Learning how to use the CLI will enable you to use Git more effectively -- and when you search for help with Git on the Internet, the answers will usually use the CLI.

## Required setup

Before beginning to write code, Git needs to be configured with your personal information. There are also a few decisions we can make at the beginning to avoid confusing error messages or problems later.

### Turn on two-factor authentication (2FA)

Please [follow GitHub's instructions and turn on two-factor authentication](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication). This will soon become mandatory.

Using a password manager that supports TOTP, such as [`pass`](https://www.passwordstore.org/) + [`pass-otp`](https://github.com/tadfisher/pass-otp) or [1Password](https://1password.com/), is strongly recommended.

### Commit name and email address

Please consistently set both your name and email address in both Git and GitHub, so that your commits can be easily identified.

Setting your name is simple: just run `git config --global user.name 'Your Name'`

[GitHub's instructions explain how to set your email address](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address) across both GitHub and the CLI. It is strongly recommended that you do not use GitHub's private email address feature.

### Other Git settings

Please run all of the commands below.

```sh
# Make the default branch name in a new repository 'main' instead of 'master`
git config --global init.defaultBranch main

# Push the current branch to the branch with the same name on the remote.
git config --global push.default simple

# Automatically create the corresponding remote branch the first time that
# 'git push' is called
git config --global push.autoSetupRemote true

# Use merge, not rebase, to resolve conflicts during `git pull`
git config --global pull.ff true
git config --global pull.rebase false

# Do not modify line endings. The working copy will represent exactly what is
# stored in Git.
#
# See "Avoiding line-ending confusion" below for more information.
git config --global core.autocrlf false
```

### Avoiding line-ending confusion

If you are using Windows, your editor might default to using Windows-style [CRLF line endings](https://en.wikipedia.org/wiki/Newline#Representation). However, our standard is to use Unix-style LF line endings. Editing a file in Windows, saving it, and commiting it to Git can cause spurious changes in the commit log as well as program errors.

There is another problem: by [default on Windows](https://stackoverflow.com/questions/39408793/git-core-autocrlf-line-ending-default-setting), Git's `core.autocrlf` setting is set to `true`. `autocrlf` is an attempt to transparently translate text files from LF line endings to CRLF line endings for Windows users.

The intent of `autocrlf` is that Windows users can edit files as if they have CRLF line endings, but, when they are committed, Git transparently translates the files and stores them with LF line endings. In reality, this additional complexity does nothing but cause more confusion. For example, with this setting in place, attempting to run a [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) script in a Visual Studio Code Dev Container will fail, because an extra CR character will be added to the shebang, confusing the Linux kernel code that will interpret this as part of the interpreter's filename. [Microsoft's own "Dev Containers Tips and Tricks" page notes this problem](https://code.visualstudio.com/docs/devcontainers/tips-and-tricks#_resolving-git-line-ending-issues-in-containers-resulting-in-many-modified-files).

Git is the wrong layer to handle CRLF/LF confusion. `autocrlf` makes it very difficult to reason about Git's behavior, and also makes it more difficult to store Windows-specific files in a Git repository.

#### Turn off `autocrlf` in GitHub Desktop

If you followed the command-line instructions above, `autocrlf` is already turned off for the Git CLI.

However, **if you are using Windows**, GitHub Desktop uses a separate configuration. You will need to find this configuration file and set `autocrlf` to `false` there. According to [this comment](https://github.com/desktop/desktop/issues/10666#issuecomment-2031102981), the configration file is located at `%localappdata%\GitHubDesktop\app-{your_github_desktop_version}\resources\app\git\etc\gitconfig`.

#### Set your editor to LF

**If you are using Windows**, you need to configure your editor to use LF line endings globally. [This StackOverflow question](https://stackoverflow.com/questions/71240918/how-to-set-default-line-endings-in-visual-studio-code) explains how to do this for Visual Studio Code.

#### Other countermeasures against CRLF

[Defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) against mistakes is generally a good idea. In addition to asking Windows users to change the above settings, we also use two configuration files in our Git repositories to help avoid line-ending problems. Users do not need to make any changes to use these configuration files; most software automatically detects and uses them.

* [EditorConfig files](https://editorconfig.org/) files tell editors what kind of line ending to use. Please make sure that your editor supports EditorConfig.
* [gitattributes files](https://git-scm.com/docs/gitattributes) allow us to turn off `autocrlf` for all files in a repository.

Manually copying and synchronizing these files across our repositories would be difficult. Instead, we use [all-repos](https://github.com/moonshot-nagayama-pj/testbed-meta/tree/main/common-config) to automatically copy and update these configuration files across all of our repositories at once.

### Git credential helper

Many of our projects depend directly on other Git repositories, which are accessed over HTTPS. In order to build the project, you will need to have GitHub credentials stored in a Git credential helper. For many people, this may already be configured.

#### Checking for an existing helper

The easiest way to check if this prerequisite is fulfilled is by trying to `git clone` a private GitHub repository over HTTPS, like this one:

```sh
git clone https://github.com/moonshot-nagayama-pj/testbed-meta.git
```

If you are prompted for a username and password, `Ctrl-C` cancel the operation and read on. If you can successfully clone the repository without entering a username and password, this prerequisite is already fulfilled.

#### Configuring a helper

There are three ways to store your credentials in Git. The first two ways are recommended by GitHub, but are more heavyweight. The third is more lightweight and will work with any Git system, not just GitHub.

##### Using the GitHub CLI or Git Credential Manager

Follow [GitHub's instructions](https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git) for caching your GitHub credentials in Git.

##### Directly using a credential helper

1. Create a GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) and grant it access to the `repo` scope. Store your personal access token in your [password manager](https://en.wikipedia.org/wiki/Password_manager).
1. Set a [Git credential helper](https://git-scm.com/doc/credential-helpers), following the instructions for your platform. For example, on Arch Linux, you can use the libsecret helper by setting this configuration: `git config --global credential.helper /usr/lib/git-core/git-credential-libsecret`
1. Load the credentials into the helper by performing an HTTPS-based Git operation. For example, `git clone` this repository over HTTPS. When you are prompted for your username and password, enter your GitHub username and personal access token.

## GitHub Flow

GitHub publishes a suggested workflow, appropriately called [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow). We use GitHub Flow as the basis for our workflow. However, the GitHub Flow documentation leaves considerable room for ambiguity. This ambiguity must be resolved before GitHub Flow can be used effectively.

Here, we will discuss the details that take GitHub Flow from an idea to a concrete process. The section headers below use the same names as the English version of the GitHub Flow document, and are a commentary on those sections of the document.

### Create a branch

If you have write access to our repositories, do not fork them. Instead, clone our repositories and create a branch. Third-party contributors are welcome to create forks.

```sh
# This will create a branch locally.
git checkout -b <new-branch-name>

# After you have committed changes, this will push the branch to GitHub.
git push
```

### Make changes

When making a commit in Git, be careful about which files you are adding to the commit. Try to use `git add <files>` followed by `git status` and finally `git commit -m '<commit message>'`. Avoid using `git commit -am`, which might accidentally commit changes you do not want to add, or accidentally skip adding new files.

Use [`.gitignore`](https://git-scm.com/docs/gitignore) appropriately.

Commit messages should be written in English. They should be about one or two sentences in length. If you are looking at the commit log later, the commit message should help you quickly identify the change you are looking for. On the other hand, commit messages should not replace code comments. Important information about your code should be written in comments.

```sh
# Check what changes exist in your repository
git status

# Add files to a commit
git add <filenames>

# Double-check that you added the right thing
git status

# Commit the changes, adding a commit message
git commit -m '<commit message>'
```

### Create a pull request

Try to create pull requests for the smallest possible unit of work. Large pull requests are usually more difficult to review and understand.

Before creating a pull request, make sure that your code is up to date with the `main` branch:

```sh
git checkout main
git pull
git checkout <my-branch-name>
git merge main
git push
```

In your pull request description, make sure to state what GitHub issue is related to this pull request. If you use [one of the keywords that GitHub recognizes](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue), like `close` or `fix`, the related issue will be closed when the pull request is merged.

Reviewers should avoid using the "Request changes" option when submitting reviews. When "Request changes" is used, GitHub will block the PR from being merged until _that specific reviewer_ approves the PR. If that reviewer is unavailable -- for example, if they are on vacation -- this can make it difficult to merge a PR.

### Address review comments

Writing a code review is a good way to start a conversation.

After a first written code review, it is often a good idea to discuss the comments in person. However, important decisions made during those discussions should be written down in the review afterward.

Reviewers should try to be clear about the importance of their comments: when a reviewer requests a significant or non-obvious change, they should clarify whether it necessary for this pull request, or can be addressed in a future pull request.

Avoid use of the "Resolve conversation" button. This hides pull request conversations that are considered complete. However, those conversations might be useful later on. The pull request might be easier to understand when all converstations are still visible.

### Merge your pull request

In general, the pull request author should merge the code. Reviewers and approvers should not generally merge pull requests. This is especially important when implementing automated deployment pipelines, where a merge might trigger the deployment of new code to a server. The author might want this to happen at a specific time, under their control.

### Delete your branch

After merging a pull request, delete the merged branch.

_After merging a pull request, delete the merged branch._

A delete button will appear after you press the merge button.

It is critical to delete branches that are no longer in use. Otherwise, GitHub repositories become more and more cluttered with abandoned and completed branches.

Do not re-use old branches for new pull requests.

Also, keep your local copy of the repository clean. After pressing the delete button in GitHub, run the following commands on your local copy:

```sh
git checkout main
git pull
git branch -d '<your branch name>'
git remote prune origin
```

This will delete the branch locally and clean up the tracking branch that pointed to the branch in GitHub's repository.

## Shared practices

### Rebasing is discouraged

There are two general philosophies on how to incorporate changes into a branch: [merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) and [rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing).

Examples in this document use simple merging. Additionally, force-pushing is banned on the `main` branch of our repositories; this effectively bans rebasing and the rewriting of history on the main branch.

Rebasing creates several risks and irritations.

* Rebasing a branch while it is under review could make it very difficult for a reviewer to understand what changes have been made in response to their review comments.
* Rebasing a branch that is used by more than one person could cause serious confusion and make it difficult for others to integrate their work with your changes.
  - The so-called "[golden rule of rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing)" is never to use it on a public branch
* Rewriting commit history makes it very difficult to establish when changes were made, and who made those changes. This is especially important when commits are cryptographically signed; rebasing erases the original author's signature.
* The author of the source-control system Fossil has [many more reasons](https://fossil-scm.org/home/doc/trunk/www/rebaseharm.md) why Fossil, unlike Git, does not have a rebase command.

If you still feel that you should use rebase on your own branches, and you are confident that you know how to use it, please be sure to consider the above when using rebase on your private branches.

### Configuration and enforcement

To the greatest extent possible, policies should be enforced automatically.

We enforce Git policy at two levels:

* GitHub project/repository configuration. This configuration is automatically applied using OpenTofu. Even if you have admin privileges, do not apply changes manually to GitHub repository configuration.
* Configuration files in Git repositories, such as `.gitattributes` and `.editorconfig`. These files are applied using [all-repos](https://github.com/asottile/all-repos/). Do not manually apply changes to these files.

### Creating new repositories

New repositories should be created programmatically using OpenTofu, as described above.

After creation, the GitHub Slack bot in \#testbed-software-github-notification should be informed about the new repository:

```
/github subscribe moonshot-nagayama-pj/<repository name>
```

## Useful tools

### Improved shell prompts

When using the command-line, it can be difficult to keep track of the current state of your repository. Custom prompts like [bash-git-prompt](https://github.com/magicmonty/bash-git-prompt) and [zsh-git-prompt](https://github.com/olivierverdier/zsh-git-prompt) can help with this.

## The Future

### Cryptographically signing code

Git is a decentralized version control system. One of the ramifications of that design choice is that there is no way to validate that a Git commit was made by the author in the Git commit log. Anyone can create a Git commit with any author and any email address.

One way to increase trust in this system is to cryptographically sign commits and release tags. As of July 2024, we do not require the signing of commits. However, in the future, we may do so.

If you are comfortable using GPG, or you wish to learn about using GPG, you can [learn more about commit signing from GitHub's documentation](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits).
