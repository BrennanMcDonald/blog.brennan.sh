---
title: "Using Github Actions to Automate Build and Deploy."
date: 2020-02-15
draft: false
category: "technology"
tags: ["github", "gh-actions", "automation", "devops","ci/cd" ]
summary: "Github Actions are a relativey new feature that was put in place to help people run CI/CD workflows directly from Github Repos. In my opinion, if you host your code on Github, they are the cleanest and fastest free form of CI/CD out there today."
---




- [Introduction](#introduction)
- [Elements of a Github actions workflow](#elements-of-a-github-actions-workflow)
  - [Trigger](#trigger)
  - [Runner](#runner)
  - [Actions](#actions)
    - [Checkout Action](#checkout-action)
    - [Custom Actions](#custom-actions)

## Introduction

Github Actions are a relativey new feature that was put in place to help people run CI/CD workflows directly from Github Repos. In my opinion, if you host your code on Github, they are the cleanest and fastest free form of CI/CD out there today.

## Elements of a Github actions workflow

Here we have an overview of the actions involved in a workflow and the layout that they occur in.

![Sections of a workflow](/img/gh-action-build-deploy/sections.png)

The core of a Github action workflow is a yaml file. There are hundreds of possible keys that exist to be used in this YAML file. However, today were only going to focus on the few important ones that are required for a basicly setup. We can start with the name for our workflow, it can be anything so don't worry about what you put here.


```yaml
name: Build and Deploy
```


### Trigger

The trigger is the action that tells your workflow to run. This can be anything from pushing to a branch, cron events, or even webhook events. More on actions can be found [here](https://help.github.com/en/actions/reference/events-that-trigger-workflows). For the sake of this guide, we will be using a _**push to branch**_ action, in specific we will be triggering our event on a push to the master branch. This is done by using the following 4 lines of code. 

``` yaml
on:
  push:
    branches:
      - master
```

This reads fairly cleanly so I wont explain it too much, but the root key is the `on:` key which says the following lines will define our trigger. We then have the `push:` key which says that the trigger will be an _**on push**_ trigger. Finally we have the `branches:` key which lists the branches we want our trigger to listen on. This can be any list of branches. For example, if we want this action to be for a pre-deploy envionment we can trigger on pushes to development branches.

``` yaml
on:
  push:
    branches:
      - alpha
      - beta
      - staging
```

### Runner

The runner is the host OS that our workflow will be running on. Due to the fact that Github Actions use [docker containers](https://www.docker.com/resources/what-container) under the covers, we have to specify what we want that container to run. 

At the time of writing this post, there are four available runners. One for Windows, two for Ubuntu Linux, and one for macOS.

| Virtual environment  | YAML workflow label            |
|----------------------|--------------------------------|
| Windows Server 2019  | `windows-latest` or `windows-2019` |
| Ubuntu 18.04         | `ubuntu-latest` or `ubuntu-18.04`  |
| Ubuntu 16.04         | `ubuntu-16.04`                   |
| macOS Catalina 10.15 | `macos-latest` or `macos-10.15`    |

In this guide, we're going to use the `ubuntu-16.04` (however the process would be the same with any of the Ubuntu runners) image as it'll be the easiest to use for our build process.

### Actions
Actions are denoted by the `setps:` key, which is followed by a list of groups of keys as illustrated in the image at the top of this page. Actions are the bulk of a github actions workflow. They describe the actions that will be executed in the workflow, from checking out the code, to running the actual build process, to deploying. We're going to spend some time learning 

#### Checkout Action



#### Custom Actions