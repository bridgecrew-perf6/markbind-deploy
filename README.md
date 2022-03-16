# markbind-deploy

Option        | Required |      Default | Remarks
:-------------|:--------:|-------------:|----------------------------------------------------------------------------------------------------------
token         |   yes    |         null | The token to be used for the service. For GitHub Action, simply use `${{ secrets.GITHUB_TOKEN }}`.
service       |    no    |   'gh-pages' | the publishing service, currently supports GitHub Pages ('gh-pages') and Surge.sh ('surge')
purpose       |    no    | 'deployment' | the deployment purpose, could be standard deployment('deployment') or for PR preview('pr-preview')
domain        |    no    |           '' | the domain to use. Required if service is surge.sh
version       |    no    |     'latest' | the MarkBind version to use. e.g. '3.1.1', or 'master' to build from the latest master branch of MarkBind
rootDirectory |    no    |          '.' | the directory to read source files from
baseUrl       |    no    |           '' | the base URL relative to your domain
siteConfig    |    no    |  'site.json' | the site config file to use

# Usage
Use as a step within your GitHub Action workflow. E.g. within your `.github/workflows/build.yml`
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
          version: '3.1.1'
```
The above script builds the site from repository root directory, with baseUrl of '/mb-test' ('mb-test' is the repository name), with MarkBind version 3.1.1.
Then, it will deploy the site to GitHub Pages. It runs everytime there is a push to the repository.
# Scenarios
- Deploy to Surge.sh
- Build with different versions of MarkBind
- Build with docs file that is not in the root level
- PR Preview