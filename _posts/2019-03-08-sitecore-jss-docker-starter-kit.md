---
layout: post
title: "Sitecore: JSS Docker Starter Kit"
date:   2019-03-08 00:00:00 -0500
categories: sitecore
tags: sitecore jss docker
author: Mike Skutta
excerpt: I was experimenting with Sitecore JSS and ran into an issue installing it due to my local install of NodeJS.  Generally, I do all of my NodeJS development within Docker containers. I take this approach because I concurrently develop with multiple different versions of NodeJS across projects.  Node Version Manager (NVM) can work, but you need to know what version of NodeJS to use based on the project. Only one version of NodeJS is supported at a time.
---

* content
{:toc}

## Overview

I was experimenting with Sitecore JSS and ran into an issue installing it due to my local install of NodeJS.  Generally, I do all of my NodeJS development within Docker containers. I take this approach because I concurrently develop with multiple different versions of NodeJS across projects.  Node Version Manager (NVM) can work, but you need to know what version of NodeJS to use based on the project. Only one version of NodeJS is supported at a time. 

I wanted the ability to install Sitecore JSS within a Docker, create an app, and then expose the files on the host.  Exposing the files on the host allows committing them to version control and editing them with your favorite editor.  I needed to do this with out using NodeJS locally.

## Implementation

Typically when doing Docker based development for NodeJS applications, the application files are on the host, shared via volume into the container. node_modules are part of the base image. The execution of the code is handled within the container.  NodeJS does not need to be installed locally.  

Based on this practice I needed to take a multiple staged approach to set this up:

1. Install the Sitecore JSS CLI into the base image.  This allows running `jss` commands within the container.
1. Run `jss create` within the container to create the application.
1. Copy the application files to a folder that is shared with the host, skipping `node_modules`.
1. Recreate the image with `npm install` already executed.
1. Run the container based on the image.

Running these steps in an automated fashion allows users to install Sitecore JSS from scratch and have a fully running instance.

I created a starter kit located here: https://github.com/onenorth/Sitecore-JSS-Docker-Starter-Kit.  This starter kit contains everything needed to get started by running a single command.  Other helper scripts have been provided to quickly run common tasks.

## Getting Started

### First run
The very first time you run this starter kit, execute the following command from bash:

```
sh 01-quick-start.sh
```

This command should only be run once when creating the project for the first time.  This command populates the **jss** service with the default Sitecore JSS files.

Once this is done. you should be able to navigate to `http://localhost:3000/` and see the site.

### Each Additional Run
Each additional run, standard docker commands can be run:

```
docker-compose up
```

### Running after getting latest / updates

Items in the containers may need to be updated when getting latest or updating the package.json.
Perform the following to make sure the containers are updated.

1. Run __`docker-compose down`__
1. Run __`docker-compose up --build`__

### Troubleshooting

When all else fails and the containers will not spin up, perform the following tasks.

> WARNING, all containers, volumes, and images will be removed.

1. Run __`docker stop $(docker ps -a -q)`__
1. Run __`docker system prune --all`__ or __`docker system prune --all --volumes`__ from Git Bash to wipe out the images.

## Running JSS Commands

The JSS site is running within the context of a Docker Container.  The JSS documentation https://jss.sitecore.com/docs/getting-started/quick-start calls out running `jss` commands.  These commands need to be run within the Docker container.

One way commands can be run within the container is by opening a shell into the container.  To open a shell, run the following command:

```
sh 99-shell.sh
```

> Note: The container needs to be running when running the above script.

Now, you can run any of the `jss` commands called out by the documentation.

### JSS Command Scripts

To save time, several shell scripts have been provided that handle running JSS commands for you.

* `02-remove-sample-content.sh` - This script removes all of the sample code in the JSS project.  This creates a blank canvas.
* `03-jss-scaffold.sh` - This executes `jss scaffold` within the container.
* `11-jss-setup` - This executes `jss setup` within the container. Be sure the Sitecore Website folder is correctly mapped to '/usr/src/website' in docker-compose.yml
* `12-jss-deploy-config.sh` - This executes `jss deploy config` within the container.
* `13-jss-deploy-app.sh` - This executes `jss deploy app --includeContent --includeDictionary` within the container.

## Troubleshooting

When running `jss deploy app --includeContent --includeDictionary`, if endpoint cannot be found, ping host.docker.internal from within the container.  Register that IP address in the HOSTS file for the JSS site.

## Summary

I just wanted to share the approach I used to get a fresh install of Sitecore JSS into Docker without needed NodeJS locally. I hope you find this helpful.

