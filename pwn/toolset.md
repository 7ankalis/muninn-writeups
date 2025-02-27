# Toolset

## GDB

There are a lot of debugging tools out there like [Radare](https://www.radare.org/r/) and [Hopper](https://www.hopperapp.com/) for Linux ;  [Immunity Debugger](https://www.immunityinc.com/products/debugger/) and [WinGDB](http://wingdb.com/) for Windows and  [IDA Pro](https://www.hex-rays.com/products/ida/) and [EDB](https://github.com/eteran/edb-debugger) for many platforms. We'll be using [GNU Debugger](https://www.gnu.org/software/gdb/).

```shell-session
$ sudo apt-get update
$ sudo apt-get install gdb
$ echo 'set disassembly-flavor intel' > ~/.gdbinit
```

```shell-session
# Pass the file to gdb 
$ gdb -q file
    q: Quiet mode: no copyright messages

# info COMMAND
gef➤  info functions
gef➤  info variables

# disassemble FUNCTION
gef➤  disas _start

# break FUNCTION
gef➤  b <function name>
gef➤  b *<ADDRESS>
To disable, enable, or delete any breakpoint by number
we use the gef➤  info breakpoints then delete or disable 
whatever we like.

# examine FUNCTION
gef➤  x/FMT <ADDRESS>
gef➤  x/CountFormatSize <ADDRESS>
    - Count of number of times we want to examine.
    - Format of result: x(hex), s(string), i(instruction)
    - Size of memory to examine: b(byte), h(halfword),
    w(word), g(giant, 8 bytes)
    Example: gef➤  x/4ig $rip
    Note: if we don't specify the Size or Format, 
    it will default to the last one we used.
```



> You may notice through debugging that some memory addresses are in the form of <mark style="color:blue;">`0x00000000004xxxxx`</mark>, rather than their raw address in memory <mark style="color:blue;">`0xffffffffaa8a25ff`</mark>. This is due to <mark style="color:red;">`$rip-relative addressing`</mark> in Position-Independent Executables `PIE`, in which the memory addresses are used relative to their distance from the instruction pointer <mark style="color:red;">`$rip`</mark> within the program's own Virtual RAM, rather than using raw memory addresses. This feature may be disabled to reduce the risk of binary exploitation.

### Plugins

For gdb there are good plugins that can helps us through the debugging process to unveil important information like Stack content, registers content...etc. the most famous and maintained ones are GEF, pwndbg, and PEDA .

#### GEF

```shell-session
$ wget -O ~/.gdbinit-gef.py -q https://gef.blah.cat/py
$ echo source ~/.gdbinit-gef.py >> ~/.gdbinit
```

#### pwndbg

```shell-session
$ git clone https://github.com/pwndbg/pwndbg
$ cd pwndbg
$ ./setup.sh
```

Or through [this link](https://github.com/pwndbg/pwndbg/releases) and t hen the command below for DEB-based OSs:

```shell-session
$ apt install ./pwndbg_2023.07.17_amd64.deb
```

#### PEDA

```shell-session
$ git clone https://github.com/longld/peda.git ~/peda
$ echo "source ~/peda/peda.py" >> ~/.gdbinit
```

Here's how to install all three of them:

```shell-session
$ cd ~ && git clone https://github.com/apogiatzis/gdb-peda-pwndbg-gef.git
$ cd ~/gdb-peda-pwndbg-gef
$ ./install.sh
```

## Ghidra

It is important to note that ghidra have both disassembling and hex dumping tool not only a decompiling tool. Also any ghidra-equivalent tool would do the job for us beginners so whatever we choose would fit out needs for now.

```shell-session
$ sudo apt install ghidra
```

## File's behavior and exposing potential vulnerabilities&#x20;

### Static analysis

#### File type analysis

Usually at the starting and medium level we won't be needing all of the commands to grasp the idea of what the file is but here's an extended list:

```bash
$file binary_name           # Basic file type
$ldd binary_name            # Check shared library dependencies (may reveal useful function libraries)
$readelf -h binary_name     # Display ELF file header for more detailed format and architecture information
$readelf -l binary_name     # Show program headers to understand segments loaded into memory
$readelf -s binary_name     # List symbols (functions, variables) defined in the binary
$objdump -x binary_name     # Full binary header information and sections
```

#### File Security

```bash
$objdump -d binary_name | grep 'call'
       # Look for direct calls to functions that might be insecure (like `gets` or `system`)
$checksec --file=binary_name  # From pwntools:$pwn checksec binary_name  equivalent if installed

```

```bash
$objdump -h binary_name       # Lists sections and their memory addresses, helpful for understanding layout
$readelf -W --sections binary_name
```

{% hint style="info" %}
Work in progress. More content to come.
{% endhint %}
