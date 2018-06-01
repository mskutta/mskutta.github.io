---
layout: post
title: "Private NPM Packages in Bitbucket"
date:   2018-05-25 00:00:00 -0500
categories: sitecore
tags: npm bitbucket heroku docker sitecore 
author: Mike Skutta
excerpt: We have many custom NPM packages that we use internally that need to remain private. Some of these packages are gulp task related for Sitecore. Others are used for building out headless implementations using NodeJS running on Heroku. There are multiple ways to host NPM packages privately. We chose to use BitBucket to host the NPM packages privately because we were already using it to host other private repositories.
---

* content
{:toc}

## Overview

We have many custom NPM packages that we use internally that need to remain private.
Some of these packages are gulp task related for Sitecore.
Others are used for building out headless implementations using NodeJS running on Heroku.
There are multiple ways to host NPM packages privately.
We chose to use BitBucket to host the NPM packages privately because we were already using it to host other private repositories.

Here is a quick reference on hosting NPM packages privately in BitBucket.

## Bitbucket Setup

First, the contents of the NPM package needs to be committed to Bitbucket.
The ```package.json``` must reside at the root of the repo.
The repository must be marked as private.

For local development, each developer needs to authenticate against BitBucket using SSH Keys generated for that developer. If you do not have an SSH Key for BitBucket, please set one up using the following instructions: [SSH Keys](https://confluence.atlassian.com/bitbucket/set-up-an-ssh-key-728138079.html).

For Heroku to pull from a private BitBucket repository, an SSH Key must be generated and registered as an Access key in the repository. See here https://bitbucket.org/onenorth/oni-wp-express-hosting/admin/access-keys/ to setup a key and for instructions. Hold onto the public and private keys.

## Docker Support

The developer SSH Key needs to be temporarily copied into the docker container so the NPM package installation has access to BitBucket.
The SSH Key for BitBucket can be added to an environment variable and docker can pick it up and use it.

The following 3 techniques can be used to add the SSH Key to an environment variable.
This needs to be done in a bash shell.

1. Run ```SSH_KEY="$(cat ~/.ssh/id_rsa)" docker-compose up```
1. The key can also be registered once in the command window ```export SSH_KEY="$(cat ~/.ssh/id_rsa)"``` and then ```docker-compose up``` can be called on its own. Export has to be run in each command window opened.
1. Lastly the key can be registered in the bash profile: ```~\.bashrc``` as ```export SSH_KEY="$(cat ~/.ssh/id_rsa)"``` and you will never have to worry about it again.

The ```docker-compose.yml``` and ```Dockerfile``` need to be updated to support environment variables:

### docker-compose.yml
```Dockerfile
version: '3'
services:
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
      args:
        - SSH_KEY=${SSH_KEY:?SSH_KEY}
```

### Dockerfile
```Dockerfile
FROM node

# Specify working directory in docker container
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Make ssh dir
RUN mkdir /root/.ssh/

# Add the keys and set permissions
ARG SSH_KEY
RUN echo "$SSH_KEY" > /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa

# Add bitbuckets key
RUN ssh-keyscan -T 60 bitbucket.org >> /root/.ssh/known_hosts

# Install node dependencies
COPY package*.json ./
RUN npm install

# Multi-stage: Keep only what we need wiping the ssh key.
FROM node
WORKDIR /usr/src/app
COPY --from=0 /usr/src/app .
CMD ["npm", "run", "dev"]
```

## Heroku Support

The following was taken from [StackOverflow](https://stackoverflow.com/questions/10869796/npm-private-git-module-on-heroku)

Approach: setup and tear down the SSH environment in heroku's ```heroku-prebuild``` and ```heroku-postbuild``` scripts

Follow these instructions:

* Create two scripts in your repo, let's call them ```heroku-prebuild.sh``` and ```heroku-postbuild.sh```.
* Add the following to ```heroku-prebuild.sh```:

```Shell
#!/bin/bash
ts() { date +"%Y/%m/%d %H:%M:%S:"; }

# Generates an SSH config file for connections if a config var exists.

if [[ ! -z $GIT_SSH_KEY ]]; then
    echo "$(ts) Detected SSH key for git. Adding SSH config"
    echo ""

    # Ensure we have an ssh folder
    if [[ ! -d ~/.ssh ]]; then
        mkdir -p ~/.ssh
        chmod 700 ~/.ssh
    fi

    # Load the private key into a file.
    echo $GIT_SSH_KEY | base64 --decode > ~/.ssh/id_rsa

    # Change the permissions on the file to
    # be read-only for this user.
    chmod 400 ~/.ssh/id_rsa

    # Setup the ssh config file.
    echo -e "Host bitbucket.org\n"\
            " IdentityFile ~/.ssh/id_rsa\n"\
            " IdentitiesOnly yes\n"\
            " UserKnownHostsFile=/dev/null\n"\
            " StrictHostKeyChecking no"\
            > ~/.ssh/config
else
    echo "$(ts) !!! SSH KEY FOR GIT NOT FOUND !!!"
    echo ""
fi
```

* Add the following to ```heroku-postbuild.sh```:

```Shell
#!/bin/bash
ts() { date +"%Y/%m/%d %H:%M:%S:"; }

if [[ ! -z $GIT_SSH_KEY ]]; then
    echo "$(ts) Cleaning up SSH config"
    echo ""

    # Now that npm has finished running, we shouldn't need the ssh key/config anymore.
    # Remove the files that we created.
    rm -f ~/.ssh/config
    rm -f ~/.ssh/id_rsa

    # Clear that sensitive key data from the environment
    export GIT_SSH_KEY=0
fi
```

* Add the following to ```package.json```

```json
  "scripts": {
    "heroku-prebuild": "bash heroku-prebuild.sh",
    "heroku-postbuild": "bash heroku-postbuild.sh"
  }
```

* Create a base64 encoded version of your private key, and set it as the Heroku config var GIT_SSH_KEY.  This can be done in GIT BASH: ```cat id_rsa | base64 > id_rsa.64```.  Remove all new lines from the generated file before copying to the variable.

When Heroku builds your app, before npm installs your dependencies, the ```heroku-prebuild.sh``` script is run. This creates a private key file from the decoded contents of the GIT_SSH_KEY environment variable, and creates an SSH config file to tell SSH to use this file when connecting to bitbucket.org. npm then installs the modules using this SSH config. After installation, the private key is removed and the config is wiped.  Base64 encoding is used to remove the line feeds.

## Referencing NPM Packages from Bitbucket

Packages hosted in Bitbucket need to be registered in the package.json file.

### Installation
npm can be used to register a package hosted in Bitbucket using the following example:

```sh
npm install ssh://bitbucket.org/owner/my-npm-package.git --save
```

### Manually Register
You can also optionally manually register in the package.json file.

```json
{
    "dependencies": {
        "my-npm-package": "bitbucket:owner/my-npm-package"
    }
}
```

### Version Pinning
You can pin to a specific tagged version in Bitbucket by adding the hash + tag name to the end of the url.  A commit must be tagged with that version.  For example:

```json
{
    "dependencies": {
        "my-npm-package": "bitbucket:owner/my-npm-package#v1.0.0"
    }
}
```