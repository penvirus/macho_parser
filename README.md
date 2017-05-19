# macho_parser

This macho parser is intentionally incomplete.  If you have any feature request, please contact me.  I could enhance the module if I have time :-)

In addition to parse common meta data of Mach-O, I used it mainly to retrieve "purposely hidden" information within the MachO binary.  See the following examples to learn more.

# APIs
## get_header()
return a namedtuple regarding to a Mach-O header for either 32-bit and 64-bit
## get_load_commands()
return a generator for each load commands
## get_segments()
return a generator for each segments for either LC_SEGMENT or LC_SEGMENT_64
## get_sections()
return a generator for each sections for either SECTION or SECTION_64
## get_section_data(segname, sectname)
return the data within specific segment and specfic section

# Examples
C source code to put some information into Mach-O structure
```
#include <stdio.h>

static char str[] __attribute__ ((section("__EXAMPLE_SEG,__example_sect"))) = "Hello World!";

int main()
{
        printf("%s\n", str);
        return 0;
}
```

compile the program
```
$ clang -segprot __EXAMPLE_SEG rwx r example.c
```
Note: the -segprot tell clang to make the segment read-only

retrieve the header
```
>>> from macho_parser.macho_parser import MachO
>>> with MachO('/tmp/a.out') as m:
...     print m.get_header()
...
mach_header_64(magic=4277009103, cputype=16777223, cpusubtype=-2147483645, filetype=2, ncmds=16, [snip]
```

list each load commands
```
>>> with MachO('/tmp/a.out') as m:
...     for lc in m.get_load_commands():
...             print lc
...
load_command(cmd=25, cmdsize=72)
load_command(cmd=25, cmdsize=472)
load_command(cmd=25, cmdsize=232)
load_command(cmd=25, cmdsize=152)
load_command(cmd=25, cmdsize=72)
load_command(cmd=2147483682, cmdsize=48)
load_command(cmd=2, cmdsize=24)
load_command(cmd=11, cmdsize=80)
load_command(cmd=14, cmdsize=32)
load_command(cmd=27, cmdsize=24)
load_command(cmd=36, cmdsize=16)
load_command(cmd=42, cmdsize=16)
load_command(cmd=2147483688, cmdsize=24)
load_command(cmd=12, cmdsize=56)
load_command(cmd=38, cmdsize=16)
load_command(cmd=41, cmdsize=16)
```

list each segments
```
with MachO('/tmp/a.out') as m:
...     for seg in m.get_segments():
...             print seg
...
segment_command_64(cmd=25, cmdsize=72, segname='__PAGEZERO\x00\x00\x00\x00\x00\x00',[snip]
segment_command_64(cmd=25, cmdsize=472, segname='__TEXT\x00\x00\x00\x00\x00\x00\x00[snip]
segment_command_64(cmd=25, cmdsize=232, segname='__DATA\x00\x00\x00\x00\x00\x00\x00[snip]
segment_command_64(cmd=25, cmdsize=152, segname='__EXAMPLE_SEG\x00\x00\x00',[snip]
segment_command_64(cmd=25, cmdsize=72, segname='__LINKEDIT\x00\x00\x00\x00\x00\x00',[snip]
```

list each sections
```
>>> with MachO('/tmp/a.out') as m:
...     for sect in m.get_sections():
...             print sect
...
section_64(sectname='__text\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00', segname='__TEXT\x00[snip]
section_64(sectname='__stubs\x00\x00\x00\x00\x00\x00\x00\x00\x00', segname='__TEXT\x00[snip]
section_64(sectname='__stub_helper\x00\x00\x00', segname='__TEXT\x00\x00\x00\x00\x00[snip]
section_64(sectname='__cstring\x00\x00\x00\x00\x00\x00\x00', segname='__TEXT\x00\x00[snip]
section_64(sectname='__unwind_info\x00\x00\x00', segname='__TEXT\x00\x00\x00\x00\x00[snip]
section_64(sectname='__nl_symbol_ptr\x00', segname='__DATA\x00\x00\x00\x00\x00\x00[snip]
section_64(sectname='__la_symbol_ptr\x00', segname='__DATA\x00\x00\x00\x00\x00\x00[snip]
section_64(sectname='__example_sect\x00\x00', segname='__EXAMPLE_SEG\x00\x00\x00'[snip]
```

retrieve data within __EXAMPLE_SEG.__example_sect
```
>>> with MachO('/tmp/a.out') as m:
...     print m.get_section_data('__EXAMPLE_SEG', '__example_sect')
...
Hello World!
```

# Links

https://pypi.python.org/pypi/macho_parser
