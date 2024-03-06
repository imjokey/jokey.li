+++
author = "Jokey Li"
title = "How to make your commit verified in github fastly"
date = "2024-03-06"
description = "How to make your commit verified in github fastly"
featured = true
tags = [
    "github",
    "security"
]
categories = [
    "Commit Verify",
    "GITHUB"
]
series = ["Neteworks"]
thumbnail = "images/materials/elephent.jpeg"
+++

## Why I should have a commit signature verification

If you  have a pull request to an external repo, it would show a "Unverified" status ,like blow:

![](/images/articles/2024-03-06-16-07-00-image.png) 

## How cloud I do that fastly

The following instructions will help you config quickly to achive that.

### Generate ed25519 type ssh keys

```
ssh-keygen -t ed25519 -C "me@jokey.li"
# You can use all prompt default options if you like.
```

### Add keys to ssh-agent

```bash
# start ssh-gaent 
eval "$(ssh-agent -s)"

# add keys
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519 # or if you are in linux to use this command
```

### Add github identify config to ssh config

```bash
vi ~/.ssh/config
# Add fllowing config in it.
Host github.com
 AddKeysToAgent yes
 IdentityFile ~/.ssh/id_ed25519
```

### Add ssh `id_ed25519.pub` to github must be with **both** sign and **auth** types.

Ref Github Docs: [Adding a new SSH key to your GitHub account - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

### Setting git global config use a ssh gpgsign verify.

```bash
   git config --global gpg.format ssh
   git config --global user.signingkey  ~/.ssh/id_ed25519.pub
   git config --global commit.gpgsign true 
```

### Just have a  test to check if it works.

```bash
   # add some new changes, commit and push your changes.
   git commit -am "update cmd"
   git push
```

Login to github to check commit entries,  whether there is a "Verified" status . like below:

![](/images/articles//2024-03-06-15-54-16-image.png)

Ref: 

[About commit signature verification - GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)

[Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

[Adding a new SSH key to your GitHub account - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

[Tell Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)

[Signing commits - GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
