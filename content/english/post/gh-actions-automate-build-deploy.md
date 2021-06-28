---
title: "Using Github Actions to Automate Build and Deploy."
date: 2020-02-15
draft: false
category: "technology"
tags: ["github", "gh-actions", "automation", "devops","ci/cd" ]
summary: "Github Actions are a relativey new feature that was put in place to help people run CI/CD workflows directly from Github Repos. In my opinion, if you host your code on Github, they are the cleanest and fastest free form of CI/CD out there today."
meta_img: img/gh-action-build-deploy/tabs.PNG
---




- [Introduction](#introduction)
- [Elements of a Github actions workflow](#elements-of-a-github-actions-workflow)
  - [Trigger](#trigger)
  - [Runner](#runner)
  - [Actions](#actions)
    - [Checkout Action](#checkout-action)
    - [Third Party Actions](#third-party-actions)
    - [Custom Actions (Run)](#custom-actions-run)

## Introduction

Github Actions are a relatively new feature that was put in place to help people run CI/CD workflows directly from Github Repos. In my opinion, if you host your code on Github, they are the cleanest and fastest free form of CI/CD out there today. To access the actions for a Github repo, navigate to the repo's page and click on the "Actions" tab at the top.

![tabs](/img/gh-action-build-deploy/tabs.PNG)

Here you can check the status of existing workflows, and by clicking the new workflow button you can deploy an existing template or a blank workflow file.

![Sections of a workflow](/img/gh-action-build-deploy/new_workflow.PNG)


## Elements of a Github actions workflow

Github actions are `yaml` files that live in the `.github/workflows` folder.  Here we have an overview of the actions involved in a workflow and the layout that they occur in.

![Sections of a workflow](/img/gh-action-build-deploy/sections.png)

The core of a Github action workflow is a `yaml` file. There are hundreds of possible keys that exist to be used in this `YAML` file. However, today were only going to focus on the few important ones that are required for a basicly setup. We can start with the name for our workflow, it can be anything so don't worry about what you put here.


```yaml
name: Build and Deploy
```


### Trigger

The trigger is the action that tells your workflow to run. This can be anything from pushing to a branch, cron events, or even webhook events. More on actions can be found [here](https://help.github.com/en/actions/reference/events-that-trigger-workflows). For the sake of this guide, we will be using a _**push to branch**_ action, in specific we will be triggering our event on a push to the master branch. This is done by using the following 4 lines of code. 

```yaml
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

In a build-deploy setup, the checkout action is typically the first step that our workflow will do. The checkout action fetches the most recent version of the repo. This action is provided by github on the 'actions' organization. It is important to note that there are two versions of the checkout action provided by github actions. The key difference is that v1 supports checking out submodules.

```yaml
- uses: actions/checkout@v1
  with:
    submodules: true
```


#### Third Party Actions

Other actions can be importated from other people's github repositories. By using the `uses:` key, you can specify a repo that will be pulled, built, and run during a workflow. These actions can take arguments by using the `with:` key, which takes a list of key-value pairs similar to envionment variables. For example, deploying a built static page to github pages, we can use `JamesIves/github-pages-deploy-action@releases/v3` like below:

```yaml
- name: Deploy
  uses: JamesIves/github-pages-deploy-action@releases/v3
  with:
    ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
    BRANCH: gh-pages
    FOLDER: dist
```

We name the custom action "Deploy" and say we will be using the `JamesIves/github-pages-deploy-action@releases/v3` action. Then by passing 3 arguments of `BRANCH`, `FOLDER`, and `ACCESS_TOKEN`, we can specify the behaviour of the action.

#### Custom Actions (Run)

You can also specify custom bash commands using the `run:` key.

```yaml
- run: npm install
- run: npm run build --if-present
```
