# bRPC Official Website
 
This project keeps all sources used for building up bRPC official website which's served at https://brpc.apache.org.

## Overview

The Apache bRPC docs are built using [Hugo](https://gohugo.io/) with the [Docsy](https://docsy.dev) theme.
This project contains the hugo project, markdown files, and theme configurations.

## Pre-requisites

- [Hugo extended version](https://gohugo.io/getting-started/installing)
- [Node.js](https://nodejs.org/en/)

## Environment setup

Install pre-requisites
```sh
$ sudo apt install npm
$ npm install
```

## Run local server


1. Clone this repository
```sh
git clone https://github.com/apache/incubator-brpc-website.git
```
2. Change to root directory: 
```sh
cd incubator-brpc-website
```
3. Run 
```sh
hugo server
```
4. Navigate to `http://localhost:1313`

5. If you want to generate the static pages, just run
```sh
hugo
```

# Note for PR

We choose master branch to hold all the site source change and asf-site for apache github website.
Please sent your PR to the master branch instead of asf-site.

## Update docs
1. Create new branch
2. Commit and push changes to content
3. Submit pull request to **master** branch
4. Generate static pagas and Submit pull request to **asf-site** branch
5. Staging site will automatically get created and linked to PR to review and test