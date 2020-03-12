<img src="https://www.artipie.com/logo.svg" width="64px" height="64px"/>

[![EO principles respected here](https://www.elegantobjects.org/badge.svg)](https://www.elegantobjects.org)
[![DevOps By Rultor.com](http://www.rultor.com/b/artipie/npm-adapter)](http://www.rultor.com/p/artipie/npm-adapter)
[![We recommend IntelliJ IDEA](https://www.elegantobjects.org/intellij-idea.svg)](https://www.jetbrains.com/idea/)

[![Build Status](https://img.shields.io/travis/artipie/npm-adapter/master.svg)](https://travis-ci.org/artipie/npm-adapter)
[![Javadoc](http://www.javadoc.io/badge/com.artipie/npm-adapter.svg)](http://www.javadoc.io/doc/com.artipie/npm-adapter)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/artipie/npm-adapter/blob/master/LICENSE.txt)
[![Hits-of-Code](https://hitsofcode.com/github/artipie/npm-adapter)](https://hitsofcode.com/view/github/artipie/npm-adapter)
[![Maven Central](https://img.shields.io/maven-central/v/com.artipie/npm-adapter.svg)](https://maven-badges.herokuapp.com/maven-central/com.artipie/npm-adapter)
[![PDD status](http://www.0pdd.com/svg?name=yegor256/npm-files)](http://www.0pdd.com/p?name=yegor256/npm-files)

Similar solutions:

   * [Artifactory](https://www.jfrog.com/confluence/display/RTF/npm+Registry)
   * [Package cloud](https://packagecloud.io/docs#node_npm)
   * [Verdaccio](https://github.com/verdaccio/verdaccio)

References:

   * [NPM registry internals](https://blog.packagecloud.io/eng/2018/01/24/npm-registry-internals/)
   * [CommonJS Package Registry specification](http://wiki.commonjs.org/wiki/Packages/Registry)
   * [The JavaScript Package Registry](https://docs.npmjs.com/misc/registry)

This is the dependency you need:

```xml
<dependency>
  <groupId>com.artipie</groupId>
  <artifactId>npm-adapter</artifactId>
  <version>[...]</version>
</dependency>
```

## How it works?

First, you upload a json file generated by `npm publish`. Then,
you call `Npm#publish`, which triggers `meta.json` file update/generation and `.tgz` source code archives creation.

This gives anything you need to respond on a series of requests generated by `npm intall <package_name>` command.

An example of `meta.json` can be found [here](https://registry.npmjs.org/axios)

## How NPM commands work with NPMSlice.

1. Let's say we use `npm publish` in the following way:

```bash
npm publish --registry=http://localhost:8080
```

By invoking such command npm sends following http request:

```json
method: "PUT"
url: "/@hello%2fsimple-npm-project"
headers: { 
  "connection": "keep-alive",
  "user-agent": "npm/6.13.6 node/v10.15.1 darwin x64",
  "npm-in-ci": "false",
  "npm-scope": "@hello",
  "npm-session": "52b4a33613bae565",
  "referer": "publish",
  "content-type": "application/json",
  "accept": "*/*",
  "content-length": "1331",
  "accept-encoding": "gzip,deflate",
  "host": "localhost:8080"
} 
```

With request body which contains `data` field which contains `base64` content of the package. 
Then we generate the meta data and `.tar` package and save it to asto.

2. Let's say now we want to install our package via command:

```bash
npm install --registry=http://localhost:8080  @hello/simple-npm-project
```

First NPM sends following request to get the meta data:

```json
method: "GET"
url: "/@hello%2fsimple-npm-project"
headers: { 
  "connection": "keep-alive",
  "user-agent": "npm/6.13.6 node/v10.15.1 darwin x64",
  "npm-in-ci": "false",
  "npm-scope": "@hello",
  "npm-session": "ee324ef71224a7ee",
  "referer": "install [REDACTED]",
  "pacote-req-type": "packument",
  "pacote-pkg-id": "registry:@hello/simple-npm-project",
  "accept":
   "application/vnd.npm.install-v1+json; q=1.0, application/json; q=0.8, */*",
  "accept-encoding": "gzip,deflate",
  "host": "localhost:8080"
} 
```

and expect following response body with meta info from asto like:

```json
{
  "name": "@hello/simple-npm-project",
  "_id": "@hello/simple-npm-project",
  "readme": "ERROR: No README data found!",
  "time": {
    "created": "2020-03-12T14:51:34.44"
  },
  "users": {},
  "versions": {
    "1.0.1": {
      "name": "@hello/simple-npm-project",
      "version": "1.0.1",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "readme": "ERROR: No README data found!",
      "_id": "@hello/simple-npm-project@1.0.1",
      "_nodeVersion": "10.15.1",
      "_npmVersion": "6.13.6",
      "maintainers": [
        {}
      ],
      "dist": {
        "integrity": "sha512-uTxRHajE8jAoeUMI32YjujfWTxD6D2Ng2hZoeR8jd8Wvx+3cJsda8mh64Cq4pFYvl65Za8OkzLAo2/vU/ibq9A==",
        "shasum": "d20a235fc4fe4f68b02649f349bfeb324178d9b1",
        "tarball": "http://127.0.0.1:8080/@hello/simple-npm-project/-/@hello/simple-npm-project-1.0.1.tgz"
      }
    }
  },
  "_attachments": {},
  "dist-tags": {
    "latest": "1.0.1"
  }
}
```

Then NPM using such meta info sends another request to get a package:

```json
method: "GET"
url: "/@hello/simple-npm-project/-/@hello/simple-npm-project-1.0.1.tgz"
headers: {
  "connection": "keep-alive",
  "user-agent": "npm/6.13.6 node/v10.15.1 darwin x64",
  "npm-in-ci": "false",
  "npm-scope": "@hello",
  "npm-session": "ee324ef71224a7ee",
  "referer": "install [REDACTED]",
  "pacote-req-type": "tarball",
  "pacote-pkg-id": "registry:@hello/simple-npm-project@http://127.0.0.1:8080/@hello/simple-npm-project/-/@hello/simple-npm-project-1.0.1.tgz",
  "accept": "*/*",
  "host": "127.0.0.1:8080" 
}
```

## How to contribute

Fork repository, make changes, send us a pull request. We will review
your changes and apply them to the `master` branch shortly, provided
they don't violate our quality standards. To avoid frustration, before
sending us your pull request please run full Maven build:

```
$ mvn clean install -Pqulice
```

To avoid build errors use Maven 3.2+.
