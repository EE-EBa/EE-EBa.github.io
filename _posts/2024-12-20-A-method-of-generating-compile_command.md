---
title: A method of generating compile_command.json
categories: [Embedded , Makefile]
tags: [makefile , bear]
---



  In the past,I used .sh file to generate compile_commands.txt to support lsp's function and code complement.<br>
Recently,I found a issue,if I didn't open a source file  that it including a function I want to check ,  <br>
then I use a shortcut "gd:go to definition",I was failed to go to real place.only to a declaration's place. <br>

  This is because compile_commands.txt [do not include the relations of .c and .h](https://neovim.discourse.group/t/clangd-lsp-jump-to-definition-takes-me-to-h-file/2153).But compile_commands.json can do it.<br>
therefore,I am going to use [bear](https://github.com/rizsotto/Bear) to generate it.The detail command which is as below:

```shell
make clean
bear -- make
```

  From above command,we must clean make before use bear.
