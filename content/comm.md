+++
title = "comm - compare sorted files, line by line"
date = "2022-04-20"

[taxonomies]
categories=["unix"]
tags=["comm"]
+++

`comm` is a command line tool that allow you to compare two sorted files, line by line.

<!-- more -->

By default, its output will be structured as a table with three columns:

- First column: contains lines that are unique to the first file
- Second column: contains lines that are unique to the second file
- Third column: contains lines that appear in both files

Example:

```sh
$ cat file1
arduino_language_server
beancount
bicep
bsl_ls
ccls
clangd
clarity_lsp
clojure_lsp
codeqlls
crystalline

$ cat file2
angularls
ansiblels
arduino_language_server
asm_lsp
awk_ls
bashls
beancount
bicep
bsl_ls
ccls

$ comm file1 file2
        angularls
        ansiblels
                arduino_language_server
        asm_lsp
        awk_ls
        bashls
                beancount
                bicep
                bsl_ls
                ccls
clangd
clarity_lsp
clojure_lsp
codeqlls
crystalline

# -12 (similar to -1 -2) will suppress column 1 & 2
$ comm -12 file1 file2
arduino_language_server
beancount
bicep
bsl_ls
ccls
```
