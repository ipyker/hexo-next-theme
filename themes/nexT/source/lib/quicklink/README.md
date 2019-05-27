<h1 align="center"><a href="https://github.com/GoogleChromeLabs/quicklink">Quicklink</a> for <a href="https://github.com/theme-next">NexT</a></h1>

<h2 align="center">Introduce</h2>

Quicklink is a JavaScript plugin that faster subsequent page-loads by prefetching in-viewport links during idle time. Chrome, Firefox, Edge are supported without polyfills.

<h1 align="center">Installation</h1>

<h2>If you want to use the CDN instead of clone this repo, please jump to the Step 3.</h2>

<h2 align="center">Step 1 &rarr; Go to NexT dir</h2>

Change dir to **NexT** directory. There must be `layout`, `source`, `languages` and other directories:

```sh
$ cd themes/next
$ ls
bower.json  _config.yml  docs  gulpfile.coffee  languages  layout  LICENSE.md  package.json  README.md  scripts  source  test
```

<h2 align="center">Step 2 &rarr; Get module</h2>

Install module to `source/lib` directory:

```sh
$ git clone https://github.com/theme-next/theme-next-quicklink.git source/lib/quicklink
```

<h2 align="center">Step 3 &rarr; Set it up</h2>

Enable module in **NexT** `_config.yml` file:

```yml
quicklink:
  enable: true
```

**And, if you wants to use the CDN, then need to set:**

```yml
vendors:
  ...
  quicklink: //cdn.jsdelivr.net/npm/quicklink@1/dist/quicklink.umd.js
```

<h1 align="center">Update</h1>

```sh
$ cd themes/next/source/lib/quicklink
$ git pull
```
