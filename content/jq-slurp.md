+++
title = "jq --slurp"
date = "2022-08-18"

[taxonomies]
categories=["unix"]
tags=["jq"]
+++

`jq --slurp` conflates multiple JSON document inputs into a single array. This allows you to run a `jq` filter only once
for all inputs, instead of once _per document_.

<!-- more -->

`jq` can transform JSON in various ways, by selecting, iterating, reducing and otherwise mangling JSON documents. For
instance, running the command `jq 'map(.price) | add'` will take an array of JSON objects as input and return the sum of
their "price" fields.

By default, it'll take a single JSON document as input, but with the `--slurp` option you can tell `jq` to read the
entire input and put it in an array, allowing you to parse multiple JSON documents in one go.

This is extraordinarily useful for files that contain a distinct JSON object per line, for example a log file:

```sh
$ cat log.txt
{"id": 1, "log": { "message": "Some Log Message Here!", "level": "INFO" }}
{"id": 2, "log": { "message": "More messages!", "level": "WARN" }}
{"id": 3, "log": { "message": "Something went wrong!", "level": "ERROR" }}

$ jq --slurp '.[].id' log.txt
1
2
3

$ jq --slurp 'map(select(.log.level == "ERROR") | .log.message)' log.txt
[
    "Something went wrong!"
]
```
