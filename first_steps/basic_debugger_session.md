## Basic Debugger Session

To debug a program, start radare with the `-d` option. Note that

You can attach to a running process by specifying its PID, or you can start a new program by specifying its name and parameters:

```
$ pidof mc
32220
$ r2 -d 32220
$ r2 -d /bin/ls
$ r2 -a arm -b 16 -d gdb://192.168.1.43:9090
...
```

In the second case, the debugger will fork and load the debugee `ls` program in memory.

It will pause its execution early in `ld.so` dynamic linker. Therefore, do not expect
to see any entrypoint or shared libraries at this point.

You can override this behavior
by setting another name for an entry breakpoint. To do this, add a radare command
`e dbg.bep=entry` or `e dbg.bep=main` to your startup script, usually it is `~/.config/radare2/.radare2rc`.

Another way to continue until a specific address is by using the `dcu` command. Which means: "debug continue until" taking the address of the place to stop at. For example:

```
dcu main
```

Be warned that certain malware or other tricky programs can actually execute code before `main()` and thus you'll be unable to control them. (Like the program constructor or the tls initializers)

Below is a list of most common commands used with debugger:
```
> d?			; get help on debugger commands
> ds 3			; step 3 times
> db 0x8048920  ; setup a breakpoint
> db -0x8048920 ; remove a breakpoint
> dc			; continue process execution
> dcs			; continue until syscall
> dd			; manipulate file descriptors
> dm			; show process maps
> dmp A S rwx	; change page at A with size S protection permissions
> dr eax=33		; set register value. eax = 33
```

Maybe a simpler method to use debugger in radare is to switch to visual mode.

That way you will neither need to remember many commands nor to keep program state in your mind.

To enter visual debugger mode use `Vpp`:

```
[0xb7f0c8c0]> Vpp
```

The initial view after entering visual mode is a hexdump view of current target program counter (e.g., EIP for x86).
Pressing `p` will allow you to cycle through the rest of visual mode views.
You can press `p` and `P` to rotate through the most commonly used print modes.
Use F7 or `s` to step into and F8 or `S` to step over current instruction.
With the `c` key you can toggle the cursor mode to mark a byte range selection
(for example, to later overwrite them with nop). You can set breakpoints with `F2` key.

In visual mode you can enter regular radare commands by prepending them with `:`.
For example, to dump a one block of memory contents at ESI:
```
<Press ':'>
x @ esi
```
To get help on visual mode, press `?`. To scroll help screen, use arrows. To exit help view, press `q`.

A frequently used command is `dr`, to read or write values of target's general purpose registers.
For more compact registers value representation you might use `dr=` command.
You can also manipulate the hardware and extended/floating point registers.

