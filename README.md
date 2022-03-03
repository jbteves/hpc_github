## Using GitHub on HPC Systems

GitHub is a common tool for developers, but using it on the HPC can be tricky.
This is because both GitHub and HPC place (very reasonable) constraints on behaviors for security.
This page is intended to be a guide to using GitHub on HPC systems.
NOTE: GitHub command line (gh) is NOT recommended!
It stores your credentials in plain text.
For most personal machines this is already not ideal, but on the HPC, it's a shared system where this is a particularly bad idea.
Please use HTTPS or SSH instead.

## Contents

This article covers the following:
- [Prerequisites](#prerequisites)
- [Using SSH keys with standalone git](#ssh-keys)
- [Using HTTPS protocol with standalone git](#https)


## Prerequisites

The following are prerequisites for using SSH/HTTPS:
- Basic git knowledge (fetch/pull and push)
- Basic GitHub knowledge (how to find a repository)
- Any working git installation

## SSH Keys

SSH keys have the strength that they are very secure, and the drawback that they require that you be logged into the head node as *the SSH protocol is prohibited on other nodes*.
In order to use SSH keys, you will need to take the following steps:
- [Generate a public and private key pair][gen_ssh_key_pair]
- [Add the public key to GitHub][add_key_to_github]
- Configure your remote to use SSH links

Configuration for SSH depends on whether you are cloning a new repository or replacing an existing one.
You may also configure git to *always* use SSH instead.
In both cases, to get the SSH link:
1. Navigate to the repository.
   ![show_repo](https://user-images.githubusercontent.com/26722533/156583556-8a870741-6601-4947-852e-ac814fab15d6.png)
1. Click "Code" with the down arrow.
   ![show_code_links](https://user-images.githubusercontent.com/26722533/156584801-44e5317d-bec6-48fd-acb2-2f61c480add7.png)
3. Select "SSH" from the "Clone" menu. 
   ![show_select_ssh](https://user-images.githubusercontent.com/26722533/156584545-01e7f43c-dd13-4544-bb0c-657532a48464.png)
5. Copy it into your clipboard with the button to the right of the link.
   ![show_button](https://user-images.githubusercontent.com/26722533/156584262-71943589-fd05-4f78-be3b-e405a8de8409.png)


### New Repository

Do the following:
1. Navigate to the place you would like to clone the repository to.
   Run the command
   ```
   git clone LINK
   ```
   with "LINK" the link you just copied.
1. It is now set up with an SSH as the remote.
   You will need to set up any remote branches manually.
   The process will look like
   ```
   git push -u origin BRANCH
   ```
   with BRANCH the current branch name to be pushed to remote.

### Old Repository

Navigate to the repository itself.
For any remote you are changing, run
```
git remote set-url REMOTE LINK
```
with REMOTE the remote name and LINK the SSH link.
If you do not know the name of your remote, it is most commonly going to be the default, "origin."
To check, you can run
```
git remote -vv
```
to see all remotes.

### Configure all repositories to use SSH

This is a relatively straightforward and fast fix, with the caveat that *all GitHub links will resolve to SSH on your account*.
If you forget this, it can lead to trouble if you try to switch to HTTPS down the line!
Be wary!

```
git config --global url.ssh://git@github.com/.insteadOf https://github.com/
```

For an explanation of how this works, see [the git-config man page][git_config_man].

## HTTPS

The key advantage of HTTPS is that it easy and works on all nodes.
The key disadvantage of HTTPS is that it involves a bit more organization and typing.
HTTPS *used* to allow people to supply their username and password.
However, GitHub [deprecated][deprecate_pass_auth] this feature in 2021.
Now, you will need to use a Personal Access Token (PAT).
It is essentially a generated string of characters that is only valid for you to access GitHub for a set timeframe.
Additionally, unlike SSH keys, which come built with a software suite to decrypt them, these tokens must use a credential manager.
It is not secure to store this credential in plain text on biowulf, so do not do that.
Instead, use a password manager such as LastPass, Apple's Keychain, or similar.

The steps to use HTTPS authorization are as follows:
1. Generate a Personal Access Token (valid for no more than 30 days), following instructions [here][generate_pat].
1. Copy this token into a password manager of some sort.
1. Ensure your repositories or git configuration use HTTPS.

To ensure your repositories use HTTPS, follow the steps outlined in SSH but use "HTTPS" links instead.
In order to ensure your configuration forces HTTPS, use this configuration command instead:

```
git config --global url.https://.insteadOf git://
```

with the same caveat as in SSH: that *all links on this account will resolve to HTTPS* (and therefore require your PAT).
Note that without a password manager there are no effective ways of storing the PAT securely without doing some interesting tricks with `gpg` and `pass`, which are not recommended.

[gen_ssh_key_pair]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent "Generating an SSH key pair"
[add_key_to_github]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account "Adding an SSH public key to your GitHub Account"
[git_config_man]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-config.html "git-config(1) Manual Page"
[deprecate_pass_auth]: https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ "Token authentication requirements for Git operations"
[generate_pat]: https://docs.github.com/en/enterprise-server@3.3/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token "Creating a personal access token"
