---
title: Embedded development notes
date: 2024-12-20
categories: [Embedded]
tags: [nvim , openocd]
---

# Steps for installing Openocd
## Introduce tools chains
1. openocd 0.12.0
2. arm-gnu-toolchain-13.2.Rel1-x86_64-arm-none-eabi
it include arm-gcc and arm-gdb

## install openocd 0.12.0
it's package is in archlinuc AUR and pacman's repository , I tried to many times install it by using AUR,<br>
but not success. I think reason is my network . Cause I'm in china. So,I use pacman's openocd pakage to install.it work!<br>
On archlinux , I don't worried many dependency package when I install software.Pacman or AUR tool can be  reliable.

## install arm-none-eabi-gdb's dependency package

1. when we install arm-none-eabi tools chain in arm website. The arm-none-eabi-gdb's function is not full.
2. maybe you can see this error message " error while loading shared libraries: libncurses.so.5", [The answer is below here:](https://bbs.archlinux.org/viewtopic.php?id=254657)

   
   ```bash
   yay -S ncurses5-compat-libs
   ```
3. when you run arm-none-eabi-gdb gain,maybe you see this error:"arm-none-eabi-gdb: error while loading shared libraries:
<br>libcrypt.so.1: cannot open shared object file: no such file or directory" . [The answer is below](https://unix.stackexchange.com/questions/691479/how-to-deal-with-missing-libcrypt-so-1-on-arch-linux) :

```bash
sudo pacman -S core/libxcrypt-compat

```
4. install python's dependency
On archlinux , default python is than 3.8, however the arm-gnu-toolchain-13.2.Rel1-x86_64 only support python3.8.<br>
So,we should install python3.8. this package can be find on aur repository.

```bash
yay -S python38
```

# Configuration of Nvim

## package manager tool
I used 'lazy.nvim' to manage my plugins. You can find it at [github](https://github.com/folke/lazy.nvim).

## nvim directory structure

```text
├── init.lua
├── lazy-lock.json
├── lazyvim.json
└── lua
    ├── config
    │   ├── init.lua
    │   ├── keymaps.lua
    │   └── lazy.lua
    └── plugins
        ├── alignCommas.lua
        ├── arm-syntax-vim.lua
        ├── bufferline.lua
        ├── codeium.lua
        ├── dashboard.lua
        ├── formatCode.lua
        ├── indent-blankline.lua
        ├── lsp.lua
        ├── lualine.lua
        ├── mason-lspconfig.lua
        ├── neoscroll.lua
        ├── nvim-autopairs.lua
        ├── nvim-treesitter-context.lua
        ├── nvim-treesitter.lua
        ├── persistence.lua
        ├── smear-cursor.lua
        ├── telescope.lua
        ├── ThemeColor.lua
        ├── which-key.lua
        ├── wilder.lua
        └── workspace.lua


```
## lsp
language c: I use plugins is clangd.
### clangd.yaml configuration
In order to use clangd for indexing more accurately, you need to configure clangd.yaml.As follows:
```yaml
# clangd.yaml location:~/.config/clangd/clangd.yaml
CompileFlags:
  Add: 
      - -I/usr/arm-none-eabi/include
  Index:
      Background: true  # 启用后台索引
      StandardLibrary: No
```

## Essential considerations for debugging preemptive interrupts using openocd + arm-none-eabi-gdb
1. Triggered breakpoints must be removed
2. Using "continue" for debugging ,not single debugging
3. Preemptive interrupts can disrupt any step of fetching,assigning,or writing back.

# makefile
## compile progress without makefile

```shell
gcc main.c addFunc.c -I ../inc 
```

[refer document](https://seisman.github.io/how-to-write-makefile/)

## vpath/VPATH issue
this two command just used in "automatic variable" ,for example $<,$@,$^,etc

## Don't creat multiple makefiles
unknown error may be encounter,<br>
for example:"make: Nothing to be done for 'Makefile_xxx'."

## built-in rule notice
excutable filename must be the same as one of dependent filenames.

## auto-dependency issue
it will  cause some unexpected error without setting auto-dependency.
 for example :variable will not update in .h files.

* [ref](https://blog.csdn.net/qq_37460963/article/details/114657399)
* [ref1](https://www.cnblogs.com/xianyi-yk/p/14501540.html)
* [subst](https://blog.csdn.net/pure_dreams/article/details/79976367) 
* [-MM sed](https://blog.csdn.net/lunhui2016/article/details/84112523)
* [$* explanation](https://www.cnblogs.com/ydxt/p/5642331.html#:~:text=%E5%A6%82%E6%9E%9C%E7%9B%AE%E6%A0%87%E4%B8%AD%E7%9A%84%E5%90%8E%E7%BC%80,%22%24*%22%E5%B0%B1%E6%98%AF%E7%A9%BA%E5%80%BC%E3%80%82&text=%E8%A1%A8%E7%A4%BA%E5%88%97%E4%B8%BE%E5%BD%93%E5%89%8D%E7%9B%AE%E5%BD%95%E4%B8%8B,%E6%96%87%E4%BB%B6%E7%A1%AE%E5%AE%9E%E5%9C%A8%E6%9C%AC%E5%9C%B0%E5%AD%98%E5%9C%A8.)
* [patsubst](https://blog.csdn.net/wangqingchuan92/article/details/116452631)
* [makefile-example](https://airguanz.github.io/2018/04/27/makefile-dept.html)
## gcc -MM -I inc src/main.c >> a.d
## gcc -MM -I inc src/main.c > a.d

## phony target issue

```makefile
SHELL=/usr/bin/zsh

# macro definde
COMPILE.C = gcc
CFLAGS 		= -I inc	
OUTPUT_MESSAGE = no message
SRC = $(wildcard ./src/*.c)#notice: must explain source file detailed location
#SRC = main.c addFunc.c
# assign  include and source file 
vpath %.h inc
vpath %.c src

%:%.o
	$(COMPILE.C) $(CFLAGS) $^ -o  $@


%.o:%.c 
	$(COMPILE.C)  $(CFLAGS) $^ -c $@ 

include $(subst .c,.d,$(SRC))
%.d: %.c
	@set -e;rm -f $@ ;
	$(COMPILE.C) -MM $(CFLAGS) $< >$@.$$$$	
#sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ >$@ ;\ 
#rm -f $@.$$$$


.PHONY:clean debug 

clean:
	rm -rf ./src/*.d.* main
debug:
	@
```
for above sytax , use "make" or "make clean" is same without any anthor dependency.

## Makefile for bare metal development

### reference resource
1. [stm32 development  environment for arm-gcc](https://microdynamics.github.io/1.%20Breeze%20Mini%E5%9B%9B%E8%BD%B4%E9%A3%9E%E8%A1%8C%E5%99%A8/2.3%20STM32%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA-Makefile%E8%AF%A6%E8%A7%A3%28ARM-GCC%29/)


