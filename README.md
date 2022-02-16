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

5. If you want to generate the static pages in /public folder, just run
```sh
hugo
```

# Note for PR

We choose master branch to hold all the site source change and asf-site for apache github website.
Please sent your PR to the master branch instead of asf-site.

## How to modify the website pages

The following is the structure of the /content folder in which the files you will mainly modify. Take adding a new committer to the **Community** page and adding a new bRPC release version to **Download bRPC** page as an example, just find the corresponding `index.md` documents in /content folder then modify them. Modifying other files or pages is similar.

```
incubator-brpc-website
- content
| - en
| | - docs
| | | - community
| | | | - index.md
| | | - DownloadBRPC
| | | | - index.md
| | | - ...
| - zh
| | - docs
| | | - community
| | | | - index.md
| | | - DownloadBRPC
| | | | - index.md
| | | - ...
```

## Update docs
1. Create new branch
2. Commit and push changes to content
3. Submit pull request to **master** branch
4. Generate static pagas and Submit pull request to **asf-site** branch
5. Staging site will automatically get created and linked to PR to review and test

## Trouble shooting

You may encounter the **Piped Failed** problem when you execute the `hugo server` or `hugo` command, the solution is as follows.
``` sh
sudo launchctl limit maxfiles 65535 200000
ulimit -n 65535
sudo sysctl -w kern.maxfiles=100000
sudo sysctl -w kern.maxfilesperproc=65535
```
