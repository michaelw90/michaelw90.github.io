---
layout: post
title:  "GitHub Actions for Linting PHP"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "GitHub Actions are a very powerful way to add continuous integration practices into your local code. Here we use PHP Linting to add confidence to our code."
hidden: true
---

#### Preface

Continuous Integration is something that in the development world is pretty key in ensuring any work that is being created is of the quality expected within the project. There are tools out there such as Bamboo, TeamCity or CircleCI which enable users to run various tests or scripts against code repositories.

GitHub has realised this and enhanced their offering by enabling the option of GitHub Actions. You build up workflows for what you want to happen in different scenarios and decide when they run. The example we're going to use here is running some very basic linting (checking code for errors) against your PHP repository.

#### Getting Started

For this we're going to be starting off with a [PHPLint](https://github.com/overtrue/phplint) library that you can download through composer. We've set this up in a cmoposer.json file that sits within the root of the repository. Alongside this we've specified a `.phplint.yml` configuration file - taking the example from on the GitHub repository.

#### Action!

We're going to be creating a new workflow in a repository to run an action that has already been created for you. An action is a Docker container that is spun up by GitHub and then various commands ran within it if you define that as part of the workflow. 

The action we're going to be use is this one - [php-lint](https://github.com/michaelw90/php-lint) - which has a very simple Docker container within it that installs composer, a few other packages and adds in the `entrypoint.sh` file. 

Within the `entrypoint.sh` file you can do almost anything, the one in the repository is very simple and just runs the phplint command against the current repository. However, you could have an [npm](https://github.com/actions/npm) action that simply takes any command passed in from the Workflow and runs it with NPM. 

#### Generating Workflow

Workflows have triggers associated with them, they can be for almost anything including (but not limited to) Pull Requests, Pushes, Issues and Forks. In this workflow we're going to be wanting to run our linting against every pull request, so we'll be listening to that. 

As mentioned above, we then have actions that are triggered. Our action in this example is very simple as our `entrypoint.sh` doesn't accept any commands etc, and simply runs the command that we expect. 

There is a workflow editor that you can use, which if you're getting started is very simple to use. However, it might be easier to copy & past the code into the relevant file below, to get the linting running.  Workflows sit within the `.github` folder and we're going to call ours `main.workflow`. The code:


    workflow "PHP Linting" {
      resolves = ["Execute"]
      on = "pull_request"
    }

    action "Execute" {
      uses = "michaelw90/php-lint@master"
    }


This creates a new workflow with one action, the workflow is triggered off "pull_request" and then executes the action that we mentioned above. There are no commands, or similar to change here, just a very straight forward workflow to lint your code.

With this in the main repository, when you raise a pull request against that repository, you should expect to see status checks appearing at the bottom of the PR outlining how the linting process goes, if it fails you can click on detail to explore the cause and then remedy the issues


#### Next Steps

You can use this action by going to the [GitHub Marketplace](https://github.com/marketplace/actions/php-lint) and enabling the action on your repository of choice.

This is a very brief outline of how to get started with a GitHub Action & Workflow. This is probably the most basic example that there is to use, however simple it can potentially help save against those rogue Linting errors before you find them when you've released your code!


<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>