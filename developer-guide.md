# MISC

* [Intro](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/intro/README.md)
* [Gitbook Deployment](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/gitbook-deployment/README.md)
* [Local Build](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/local-build/README.md) (_not recommended, only for functional testing_)
  * [Environment Setup](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/environment-setup/README.md)
  * [Workflow](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/workflow/README.md)

## Intro

A repo that served as an archive site for the organization UMJI-SSTIA.

In order to contribute, you may:

* If you have push access to this repo, simply clone this repo, edit the corresponding content, create a commit and push to the master branch. The Gitbook service will automatically fetch your commit and refresh building.
* Otherwise, you may either:
  * log in as `ji-sstia`, or
  * create pull request, or
  * send your patch to `ji_sstia@163.com`, or
  * raise an issue in [https://github.com/UMJI-SSTIA/ji-sstia-pages/issues](https://github.com/UMJI-SSTIA/ji-sstia-pages/issues) and describe your changes

If you would like to add a new page, you may:

1. create a new markdown file in a suitable subdirectory
2. index the file in `SUMMARY.md` following existed examples
- *(Just see the arrangement of the files and you will understand.)*

## Gitbook Deployment

The content is deployed automatically on [ji-sstia.gitbook.io/sstia](https://github.com/UMJI-SSTIA/ji-sstia-pages/tree/577cede8af4b0fdbf86fb5622279d61cef7011a5/ji-sstia.gitbook.io/sstia/README.md). It fetches the `master` branch of [https://github.com/UMJI-SSTIA/ji-sstia-pages](https://github.com/UMJI-SSTIA/ji-sstia-pages) automatically.

## Local Build

* Source: [link](https://github.com/GitbookIO/gitbook)
* Config guide: [link](https://github.com/GitbookIO/gitbook/blob/master/docs/config.md)

If you would like to test locally:

### Environment Setup

install nodejs & npm

```
sudo apt update
sudo apt install nodejs npm
# upgrade
sudo npm install n -g
sudo n stable npm i -g npm
# change npm source
npm config set registry http://registry.npm.taobao.org/
```

nvm

```
# assume using bash
curl https://gitee.com/mirrors/nvm/raw/master/install.sh | bash
source ~/.bashrc
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
nvm install 12.18.1
nvm use 12.18.1
```

install gitbook

```
# install gitbook-cli
sudo npm install -g gitbook-cli
# install gitbook; NOTE: it takes around 20mins
gitbook -V
# verify the installation of gitbook
gitbook -V
# optional for export PDF
sudo apt install calibre
```

### Workflow

* ~~`gitbook init` initialize your local directory~~ no need since we've already got our contents
* `git clone git@github.com:UMJI-SSTIA/ji-sstia-pages.git & cd ji-sstia-pages`
* `gitbook serve` deploy locally
  * by default `localhost:4000`
* `gitbook build` build static website pages into `./_book`
