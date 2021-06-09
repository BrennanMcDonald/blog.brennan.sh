---
title: "How I Automated With Ease Using Github Actions."
date: 2020-02-13
draft: false
category: "technology"
tags: ["github", "gh-actions", "automation", "devops","ci/cd" ]
summary: "About a week ago I decided to revamp my personal site and FINALLY get a working blog in order. Since you're reading this obviously the blog half worked out! But as someone who has always had a relative interest in DevOps and more recently started putting that intesrest into motion, I thought I would use this opportunity to enhance my skills in CI/CD"
---

- [Introduction](#introduction)
- [Building my workflows](#building-my-workflows)
  - [Hugo](#hugo)
    - [Trigger](#trigger)
    - [Checkout Action](#checkout-action)
    - [Setup Hugo Action](#setup-hugo-action)
    - [Build Action](#build-action)
    - [Github Pages Deploy](#github-pages-deploy)
  - [React Native](#react-native)
    - [Setup Action](#setup-action)
    - [Build Action](#build-action-1)
- [Conclusion](#conclusion)

## Introduction

#### [This is a narritave on my personal use case. If you want a guide on how to automate: click here](/posts/gh-actions-automate-build-deploy) <!-- omit in toc -->

About a week ago I decided to revamp my personal site and _FINALLY_ get a working blog in order. Since you're reading this the blog half obviously worked out! (Spoiler alert: so did the website half). But as someone who has always had an interest in code automation, and more recently someone who has started to expariment with it, I thought I would use this opportunity to enhance my skills in [continous integration](https://en.wikipedia.org/wiki/Continuous_integration) and [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery).

![model of the CI/CD methodology](/img/gh-action-build-deploy/ci-cd.png)

Before I thought about my build and deploy process, I had to think about the technologies I wanted to use to build the projects themselves. For the website I went with NodeJS using the React framework. I picked React due to the fact that it's the framework i'm most comfortable with and I would be able to write it most efficiently. As for the blog, I have history using Hugo to build static sites. Hugo is a framework that is efficient at generating static html for blogs. It seemed like the best framework for the job.

## Building my workflows

### Hugo

For my hugo workflow, I used a mix of built in and user created actions. The workflow is triggered by a code push to the master branch of the repo. I'm going to show the entire yaml file and then step through it, line-by-line.

``` yaml
name: HUGO Build and Deploy

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.62.2'

      - name: Build
        run: hugo --minify
        
      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          ACCESS_TOKEN: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          BRANCH: gh-pages
          FOLDER: public
```

Walking through this, section by section we can see the breakdown of the workflow logic.

#### Trigger

It all starts with a very simple trigger that builds on a push to the master. Since this project isn't fast moving, I can have the building of the static files driven by the event of a push. However if I were to be pushing raidly to this repo, I might decide to schedule a nightly build instead.

``` yaml
on:
  push:
    branches:
      - master
```

Im going to skip the runner because there isn't much to say about selecting ubuntu as the base image that I used.

#### Checkout Action
```yaml
- uses: actions/checkout@v1
  with:
    submodules: true
```
Her we have my first action, the checkout action. If you're familiar with github actions it should be simple enough to identify what this does. Essentially, this action pulls the most recent code base on the master branch and places it in our workflow's container. 

Something of note here, I have to use the checkout V1 module becasue the V2 module does not yet support submodules, which is required by this project because my theme is a [seperate git repo](https://github.com/BrennanMcDonald/hugo-theme-noteworthy) and i've included it as a submodule in the themes folder.

#### Setup Hugo Action

The next action is a great action from [peaceiris](https://github.com/peaceiris), which interstingly enough is written in [TypeScript](https://www.typescriptlang.org/), that simply pulls the latest hugo version and installs it on the workflow's runner image.

``` yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v2
  with:
    hugo-version: '0.62.2'
```


#### Build Action

Thanks to the simplicity of hugo's build system, an image isn't necessary for this step, all we need to do is run the `hugo` command in our working directory and we have liftoff.

``` yaml
- name: Build
  run: hugo --minify
```

#### Github Pages Deploy

Finally, once we've build hugo, we'll need to deploy. In this case im deploying to github pages because im fairly fond of it and use it for a few other projects already (may as well put all the eggs in one basket eh?). This action is a little more technical so i'll take a walk through what is going on under the covers.

```yaml
- name: Deploy
  uses: JamesIves/github-pages-deploy-action@releases/v3
  with:
    ACCESS_TOKEN: ${{ secrets.ACTIONS_DEPLOY_KEY }}
    BRANCH: gh-pages
    FOLDER: public
```

Here we're pulling [JamesIves's](https://github.com/JamesIves/) well used _**github-pages-deploy-action**_ action. We also have our first use of a [secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) in this workflow. In this case, that secret is my github personal access token which is used for copying my build artifacts (the html files) to the gh-pages branch and setting up the Github Pages settings. 

The core of this action is the `git` cli which is orchestrated wiht a couple TypeScript files. The two other arguments in this action are fairly straightforward, `BRANCH:` is the selected branch we are going to be pushing our build code to, and `FOLDER:` is the folder whos content will be pushed to the specified branch.

### React Native

This set of actions is similar to the previous example with two main differences, the setup and build actions. 

```yaml
name: Node.js CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Build
      uses: actions/setup-node@v1
      with:
        node-version: 10.x

    - run: npm install
    - run: npm run build --if-present
      env:
        CI: true

    - name: Deploy
      uses: JamesIves/github-pages-deploy-action@releases/v3
      with:
        ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
        BRANCH: gh-pages
        FOLDER: dist
```

#### Setup Action

```yaml
uses: actions/setup-node@v1
with:
  node-version: 10.x
```

The action we use for the NodeJS setup is provided by github under the [actions org](https://github.com/actions/). This action uses a [TypeScript installer](https://github.com/actions/setup-node/blob/master/src/installer.ts) to set up NodeJS on the runner.

#### Build Action

```yaml
- run: npm install
- run: npm run build --if-present
  env:
    CI: true
```

These two `run:` lines execute two lines of `npm cli` code. To NodeJS developers these should be self-explanitory, but for real developers: The first line installs all dependancies required for the build process, and the second one builds the code that is present in the working directory.

## Conclusion

I hope this has shed some light on how EASY automation with Github Actions is. I now have two files that I can drop into any React or HUGO project that will work. If you have any questions you think I can answer, feel free to reach out to me using the links to the left.