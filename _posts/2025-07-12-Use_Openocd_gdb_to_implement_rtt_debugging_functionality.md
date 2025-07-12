---
title:
date: 2025-07-12
categories: [Embedded]
tags:
  [
    assmebly,
    nvim,
    openocd,
    debug,
    RTOS,
    GNU,
    makefile,
    Linux,
    iwctl,
    git,
    stow,
    dotfiles,
  ]
pin: [true]
---

# Background

When I debug embedded Rust code, the RTT service is a good choice for printing information on the host PC.
However, using the RTT service to debug a Rust project with GDB/OpenOCD requires some setup steps. Here are the steps you need to know.

# 1. Start the OpenOCD service

I prefer to use a standalone terminal to run `openocd.cfg`.

```shell
openocd -f openocd.cfg
#or you also execute this command
openocd
```

# 2. Start the GDB service

There are many methods to start the GDB service, depending on the IDE you use.

## 1. Using the command line to debug the project

If you are debugging the project using the command line without an IDE, you generally need to add an `openocd.gdb` file to configure RTT. Here is my `openocd.gdb` file:

```txt

target extended-remote :3333

# print demangled symbols
set print asm-demangle on

# set backtrace limit to not have infinite backtrace loops
set backtrace limit 32

# detect unhandled exceptions, hard faults and panics
break DefaultHandler
break HardFault
break rust_begin_unwind
# # run the next few lines so the panic message is printed immediately
# # the number needs to be adjusted for your panic handler
# commands $bpnum
# next 4
# end

# *try* to stop at the user entry point (it might be gone due to inlining)
break main
monitor arm semihosting enable

monitor rtt server start 9090 0

#run this command to get address and size : rust-nm -S target/thumbv7em-none-eabihf/debug/rtt_prints|grep RTT|awk '{print $1}'
monitor rtt setup 0x20000000 0x00000030 "SEGGER RTT"
monitor rtt start


load

# start the process but immediately halt the processor
stepi

```

And the you just execute this command:

```

arm-none-eabi-gdb -q -x openocd.gdb

```

## 2. Using Neovim to debug the project

I always use Neovim to debug my Rust projects. Here is my RTT configuration for GDB/OpenOCD, using the `nvim-dap` and `nvim-dap-ui` plugins:

```lua
local dap = require("dap")

dap.configuration.rust = {
  {
				name = "Debug --openocd --load",
				type = "cppdbg",
				request = "launch",
				preLaunchTask = "cargo build",
				MIMode = "gdb",
				miDebuggerServerAddress = "localhost:3333",
				miDebuggerPath = "arm-none-eabi-gdb",
				cwd = "${workspaceFolder}",
				program = function()
					return program()
				end,

				stopAtEntry = false,
				postRemoteConnectCommands = {
					{
						text = function()
							return "load " .. program()
						end,
					},
					{
						text = "monitor reset halt",
					},
					{
						text = "monitor rtt server start 9090 0",
					},
					{
						--run this command to get address and size : rust-nm -S target/thumbv7em-none-eabihf/debug/rtt_prints|grep RTT|awk '{print $1}'

						text = 'monitor rtt setup 0x20000000 0x00000030 "SEGGER RTT"',
					},
					{
						text = "monitor rtt start",
					},
				},
				setupCommands = {
					{
						description = "Enable pretty-printing",
						text = "-enable-pretty-printing",
						ignoreFailures = true,
					},
				},
			},

}

```

And then execute this to startup the gdb service.

```vim
lua:require("dap").continue()

```

# 3. Monitor the output on port 9090

The RTT output needs to be displayed in another terminal. Execute the following command in a separate terminal:

```shell
telenet localhost 9090
```
