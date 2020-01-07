---
title: "Setting up Hexo on Windows"
date: 2015-10-11 15:45:43
categories:
  - Web
tags:
- hexo
- blog
---
Follow these instructions to install and configure the
[Hexo blog framework](https://hexo.io/) on Windows.

## Requirements

- [Git Bash](https://git-scm.com/)

## 1. Install Node.js

1. Install Node.js

    ![Node.js Windows Installer](node-js-installer.png)

2. Make sure to allow Node.js Windows Firewall access.

    ![Windows Firewall access](windows-firewall.png)

3. Verify the installation by running the following commands
inside the Git Bash:

    ```bash
    node -v
    npm -v
    ```

## 2a. Re-install hexo

For those re-installing Hexo from an existing hexo blog repository:

1. do NOT create the directory described step 1 but checkout your blog repo instead
2. cd into your repo directory and git checkout the `source` branch
3. continue with step 2 below but SKIP the ``hexo init`` command

## 2b. Install Fresh Hexo

Please note that all commands are to be executed within the Git Bash.

1. Create a new directory that will hold your blog

    ```bash
    mkdir myblog
    cd myblog
    ```
2. Install the hexo framework and the git deployer

    ```bash
    npm install hexo-cli -g
    npm install hexo-deployer-git --save
    hexo -v
    ```
3. Initialize your blog

    ```bash
    hexo init
    npm install
    hexo serve
    ```

4. Now browse to `http://localhost:4000` and enjoy your first blog

5. You're good to go, next steps would be:
    - customizing `_config.yml` to your likings
    - creating your first post by running ``hexo new``

## 3. Deploy

The hexo git deployer has taken all the hard work out of deploying your
blog to your master repo.

1. First load your Github private key inside the Git Bash

    ```bash
    eval `ssh-agent`
    ssh-add ~/.ssh/github_rsa_key
    ```

2. Now simply generate and deploy your blog

    ```bash
    hexo generate -d
    ```

## 4. Save Local Changes

**Do not forget** to save your local changes by commit pushing them to the `source` branch of your central repository.
