# markbind-deploy
A GitHub Action that builds and deploys your MarkBind site.

## Option Summary

Option        | Required |      Default | Remarks
:-------------|:--------:|-------------:|------------------------------------------------------------------------------
token         |   yes    |              | The token to be used for the service
service       |    no    |   'gh-pages' | The publishing service to deploy the site
purpose       |    no    | 'deployment' | The deployment purpose
domain        |    no    |           '' | The domain that the site is available at. Required if service chosen is surge
version       |    no    |     'latest' | The MarkBind version to use to build the site
keepFiles     |    no    |        false | Whether to keep the files in the published branch before pushing
rootDirectory |    no    |          '.' | The directory to read source files from
baseUrl       |    no    |           '' | The base URL relative to your domain
siteConfig    |    no    |  'site.json' | The site config file to use

## Option Details

### token
Currently two types of tokens are supported (correspond to the two supported publishing services):
- token for GitHub Pages
  - simply use `${{ secrets.GITHUB_TOKEN }}`
  - Note that you need to ensure that your have selected the branch that you want to deploy to in your GitHub settings
- token for Surge.sh
  - example: `${{ secrets.SURGE_TOKEN }}`
    - `SURGE_TOKEN` is the environment secret name
  - require registration with an email address
  - after retrieving the token, put the token as a repository secret in your
  - see [here](https://markbind.org/userGuide/deployingTheSite.html#previewing-prs-using-surge) for a detailed guide on how to retrieve the token

### service
Currently two types of publishing services are supported:
- GitHub Pages
  - 'gh-pages'
  - see [here](https://markbind.org/userGuide/deployingTheSite.html#publishing-to-github-pages) for more details
- Surge.sh
  - 'surge'
  - see [here](https://surge.sh/) for more details

### purpose
The purpose of the deployment. This can be either standard deployment or for PR previewing.
- Standard deployment
  - 'deployment'
  - This is the default value
- PR preview
  - 'pr-preview'
  - This is used when you want to build and preview a site when there is a PR made to the repository. Note that this does not work for PR from a fork(due to security reasons)

### domain
The domain that the site is available at. Required if service chosen is surge. Surge.sh allows you to specify a subdomain as long as it is not taken up by others.

- 'xxx.surge.sh'
  - A typical domain that you can specify. You have to ensure that 'xxx' is unique. Read [here](https://surge.sh/help/adding-a-custom-domain) on how to configure a custom domain with Surge.sh
- 'pr-x-domain'
  - Note that for PR Preview purposes, the domain will be prefixed with 'pr-x', which 'x' is the GitHub event number
  - You will need to specify what 'domain' is

### version
The MarkBind version to use to build the site.
- 'latest'
  - This is the latest published version of MarkBind
- 'master'
  - This is the latest, possibly unpublished version of MarkBind
- 'X.Y.Z'
  - This is the version of MarkBind with the specified version number
  - A sample version number is '3.1.1'

### keepFiles
Whether to keep the files in the published branch before pushing. This is a boolean parameter.
- false
  - This is the default value
- true
  - This will make the published branch keep the existing files before an update is made.

### rootDirectory (MarkBind CLI arguments)
The directory to read source files from.
- '.'
  - This is the default value
  - This is for the case that your source files of the MarkBind site are in the root directory of the repository
- './path/to/directory'
  - This is for the case that your source files of the MarkBind site are in a subdirectory of the repository
  - A sample path is './docs'

### baseUrl (MarkBind CLI arguments)
The base URL relative to your domain.
- ''
  - This is the default value
- '/reponame'
  - This is for deploying your site to GitHub Pages
  - Note that you will need to specify this in order to configure the relative URL correctly. This should also be specified in the site config file

### siteConfig (MarkBind CLI arguments)
The site config file to use.
- 'site.json'
  - This is the default value

# Usage
In essence, there are two parts to a GitHub Action workflow:
- The trigger event
- The jobs/steps to be run after the trigger event occurs

For our context, there two typical trigger events. This is written at the start of the workflow files:
(Assuming 'master' is the target branch)
1. Trigger the action whenever there is a push to the repository
```yaml
on:
  push:
    branches:
      - master
```

2. Trigger the action whenever there is a pull request to the repository
```yaml
on:
  pull_request:
    branches:
      - master
```

Then, specify a step to use this action:

E.g.
```yaml
name: MarkBind Deploy

on:
  push:
    branches:
      - master

jobs: 
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build & Deploy to GitHub Pages
        uses: tlylt/markbind-deploy@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          baseUrl: '/mb-test'
          version: '3.1.1'
```
The above script builds the site from the repository's root directory, with baseUrl of '/mb-test' ('mb-test' is the repository name), with MarkBind version 3.1.1.

Then, it will deploy the site to GitHub Pages. It runs everytime there is a push to the repository.

# Scenarios
## Deploy to Surge.sh
```yaml
name: MarkBind Deploy

on: [push]

jobs: 
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build & Deploy to GitHub Pages
        uses: tlylt/markbind-deploy@main
        with:
          token: ${{ secrets.SURGE_TOKEN }}
          service: 'surge'
          domain: 'mb-test.surge.sh'
```
Note that if you are using custom domain, you will need to ensure that it is configured with Surge.sh.

## Build with the development branch of MarkBind
```yaml
name: MarkBind Deploy

on: [push]

jobs: 
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build & Deploy to GitHub Pages
        uses: tlylt/markbind-deploy@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          baseUrl: '/mb-test'
          version: 'master'
```
'master' is specified so that the site is built with the latest (possibly unpublished) version of MarkBind.

## Build with files that is not at the root level of the repository
```yaml
name: MarkBind Deploy

on: [push]

jobs: 
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build & Deploy to GitHub Pages
        uses: tlylt/markbind-deploy@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          baseUrl: '/mb-test'
          rootDirectory: './docs'
```

## PR Preview for PRs made following a branching workflow
```yaml
name: MarkBind Deploy

on:
  pull_request:
    branches:
      - master

jobs: 
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build & Deploy to GitHub Pages
        uses: tlylt/markbind-deploy@main
        with:
          token: ${{ secrets.SURGE_TOKEN }}
          service: 'surge'
          purpose: 'pr-preview'
          domain: 'mb-test.surge.sh'
```
