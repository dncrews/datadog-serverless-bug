# serverless-plugin-datadog issue

Per [the documentation](https://github.com/DataDog/serverless-plugin-datadog#how-do-i-use-this-with-serverless-webpack), this package is compatible with serverless-webpack.

## Issue 1: Does not properly exclude aws-sdk

**Background**

Per [the webpack documentation](https://github.com/serverless-heaven/serverless-webpack#aws-sdk), since `aws-sdk` is included in all Lambdas, you should exclude it. The expected way to do this is

```yml
# serverless.yml
custom:
  webpack:
    includeModules:
      forceExclude:
        - aws-sdk
```

**Expected Behavior**

```sh
$ cd ./no-dd
$ yarn # or `npm install`
$ yarn pkg # or `npm run pkg`
```

1. Examine the resulting `.serverless/without-datadog.zip`. The filesize should be 1.3MB
2. Unzip this file.
3. The filesize of the directory is roughly 7.3 MB
4. You will see two files:
    * `handler.js`: 6 MB
    * `handler.js.map`: 1.3 MB


**Actual Behavior**

```sh
$ cd ./dd
$ yarn # or `npm install`
$ yarn pkg # or `npm run pkg`
```

1. Examine the resulting `.serverless/with-datadog.zip`. The filesize will be 11.5 MB
2. Unzip this file.
3. The filesize of the directory is roughly 66.1 MB
4. You will see the following:
    * `package-lock.json`: 10 KB (trivial)
    * `package.json`: 195 B (trivial)
    * `node_modules/`: 59.8 MB
    * `datadog_handlers/`
      * `hello.js`: 6 MB (same size)
      * `hello.js.map`: 1.3 MB (same size)

----

## Issue 2: Multiple Uses of a handler file multiplies the filesize

**Background**

Because of the way that the DataDog plugin wraps the functions, all of the files are duplicated. Today, this caused my Lambda size to become 3 times the original size:

***Admittedly, I went a little overboard with the number of functions to prove my point, since my functions were very small. I had this issue myself today with a Lambda with 17 rather-large functions, and my filesize was too big to be deployable***

**Expected Behavior**

```sh
$ cd ./no-dd # same as before)
```

1. Examine the resulting `.serverless/without-datadog.zip`. The filesize should be 1.3MB
2. Unzip this file.
3. The filesize of the directory is roughly 6.3 MB
4. You will see two files:
    * `handler.js`: 6 MB
    * `handler.js.map`: 1.3 MB


**Actual Behavior**

```sh
$ cd ./dd-multi
$ yarn # or `npm install`
$ yarn pkg # or `npm run pkg`
```

1. Examine the resulting `.serverless/with-datadog-many.zip`. The filesize will be 87.9 MB
2. Unzip this file.
3. The filesize of the directory is roughly **499.7 MB** (too large to fit into a single Lambda)
4. You will see the following:
    * `package-lock.json`: 10 KB (trivial)
    * `package.json`: 195 B (trivial)
    * `node_modules/`: 59.8 MB
    * `datadog_handlers/`
        * `hello0.js`: 6 MB (same)
        * `hello0.js.map`: 1.3 MB (same)
        * `hello1.js`: 6 MB (same)
        * `hello1.js.map`: 1.3 MB (same)
        * `hello2.js`: 6 MB (same)
        * `hello2.js.map`: 1.3 MB (same)
        * ... etc
        * `hello59.js`: 6 MB (same)
        * `hello59.js.map`: 1.3 MB (same)
