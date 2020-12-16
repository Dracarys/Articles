# Xcode command line tools 

## 安装
工欲善其事，必先利其器。所以学习之前，先看看如何安装。首先你的有 Xcode，没安装请先移步 App Store 进行下载安装，一键式操作，非常便捷。之后，就可以安装 Command line tools 了，打开 Terminal，直接键入如下命令：

```shell
xcode-select --install
```
此时会弹出提示，确认安装，等进度条结束即可。结束后，进入 `/Library/Developer/CommandLineTools/usr/bin/` 查看一下，具体有哪些工具可用。

长长一串，有点眼晕啊，但是细心观察可以发现很多都是跟版本控制、Swift、metal相关的，这些我们不去讨论，重点集中在那些通用型较强的工具上。比较多，很多还牵涉到编译到相关知识，不足之处望斧正。

## ar
ar 工具应该是来自 architecture 的简写（猜的，未考证），可以通过它对二进制的静态库进行一些编辑操作，例如：对某个架构的静态库进行解包，删除其中的一些目标文件（.O 文件）等等。

>⚠️ 注意：必须是 Non-fat 静态库，否则会提示类型错误。

## lipo

## nm

## nmedit

## size

## ranlib

## as

## cmpdylib

## objdmp

## libtool

## segedit

## strip

## rebase

## otool

### otool 支持的架构标识（arch_type），大小端不限：

ppc64 x86_64 x86_64h arm64 ppc970-64 arm64_32 arm64e ppc i386 m68k hppa sparc m88k i860 veo arm ppc601 ppc603 ppc603e ppc603ev ppc604 ppc604e ppc750 ppc7400 ppc7450 ppc970 i486 i486SX pentium i586 pentpro i686 pentIIm3 pentIIm5 pentium4 m68030 m68040 hppa7100LC veo1 veo2 veo3 veo4 armv4t armv5 xscale armv6 armv6m armv7 armv7f armv7s armv7k armv7m armv7em arm64v8

### otool 命令格式：

otool [-arch arch_type] [-fahlLDtdorSTMRIHGvVcXmqQjCP] [-mcpu=arg] [--version] <object file> ...

### otool 支持的参数：

	-f print the fat headers
	-a print the archive header
	-h print the mach header
	-l print the load commands
	-L print shared libraries used
	-D print shared library id name
	-t print the text section (disassemble with -v)
	-x print all text sections (disassemble with -v)
	-p <routine name>  start dissassemble from routine name
	-s <segname> <sectname> print contents of section
	-d print the data section
	-o print the Objective-C segment
	-r print the relocation entries
	-S print the table of contents of a library (obsolete)
	-T print the table of contents of a dynamic shared library (obsolete)
	-M print the module table of a dynamic shared library (obsolete)
	-R print the reference table of a dynamic shared library (obsolete)
	-I print the indirect symbol table
	-H print the two-level hints table (obsolete)
	-G print the data in code table
	-v print verbosely (symbolically) when possible
	-V print disassembled operands symbolically
	-c print argument strings of a core file
	-X print no leading addresses or headers
	-m don't use archive(member) syntax
	-B force Thumb disassembly (ARM objects only)
	-q use llvm's disassembler (the default)
	-Q use otool(1)'s disassembler
	-mcpu=arg use `arg' as the cpu for disassembly
	-j print opcode bytes
	-P print the info plist section as strings
	-C print linker optimization hints
