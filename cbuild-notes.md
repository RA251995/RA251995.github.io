# C Build
## Build Process
Preprocess ⟶ Compile (`gcc`) ⟶ Assemble (`as`) ⟶ Link (`ld`) ⟶ Locate

| File Type                                   | Extention |
|---------------------------------------------|-----------|
| Source files                                | *.c *.h   |
| Preprocessed files                          | *.i       |
| Compiled files                              | *.s       |
| Assembled files                             | *.o       |
| Library files (*goes directly into linker*) | *.a       |

---
### **GNU Toolchain**
#### Naming
`<ARCH>-<VENDOR>-<OS>-<ABI>-<TOOL>`  
Examples: 
- `arm-none-eabi-gcc`  
- `arm-linux-gnueabi-as` 

#### **Useful GNU tools** (*binutils*)

| Tool      | Use                                                                       |
|-----------|---------------------------------------------------------------------------|
| `size`    | Lists section sizes for object and executable files                       |
| `nm`      | List symbols from object files                                            |
| `objcopy` | Copies and translates object files                                        |
| `objdump` | Displays info from object files (Disassemble: `objdump -d <object-file>`) |
| `readelf` | Displays info from elf files                                              |
| `gdb`     | GNU debugger                                                              |

#### **Other useful tools**

| Tool      | Use                             |
|-----------|---------------------------------|
| `ldd`     | Pint shared object dependencies |

---
#### **General gcc options**
```bash
    -c                  # Compile and assemble files; Donot link
    -o <FILE>           # Compile, assemble and link to <FILE>
    -g                  # Generate debugging information in executable
    -Wall               # Enable all warning messages
    -Werror             # Treat all warnings as errors
    -I<DIR>             # Include <DIR> to look for header files
    -ansi-std=<STD>     # Specify <STD> to use (c89, c99, ...)
    -v                  # Verbose output
    -S                  # Compile only; Donot assemble and link
    -E                  # Preprocess only; Donot compile, assemble and link
    -D<MACRO-NAME>      # Complie time switch
```
#### Linker flags
```bash
    -T <SCRIPT>         # Linker <SCRIPT>
    -l <LIB>            # Link with <LIB>       
    -L <DIR>            # Include library <DIR>
    -map <FILE>         # Output memory map <FILE> from result of linking
    -O<#>               # Optimization level (<#> = 0,1,2,3)
    -Os                 # Optimizate for memory size
    -z stacksize=<SIZE> # Amount of stack space to reserve
    -shared             # Produce shared library (dynamically linked library)
    -Wl, <OPTION>       # Pass <OPTION> to linker from compiler
    -Xlinker <OPTION>   # Pass <OPTION> to linker from compiler
```
#### Architecture specific gcc options
```bash
    -mcpu=<NAME>        # Target ARM processor and architecture (cortex-m0plus, ...)
    -march=<NAME>       # Target ARM architecture (armv7-m, thumb, ...)
    -mtune=<NAME>       # Target ARM processor (cortex-m0plus, ...)
    -mthumb             # Generate code in Thumb states (ISA)
    -marm               # Generate code in ARM state (ISA)
    -mthumb-interwork   # Generate code that support calls between ARM and Thumb (ISA)
    -mlitte-endian      # Generate code for little endian mode
    -mbig-endian        # Generate code for big endian mode
```
#### Bare metal options
```bash
    -nostdlib           # Do not use the standard system startup files or libraries when linking
    -nostartfiles       # Do not use standard system startup files when linking
    -ffreestanding      # Assert that compilation targets a freestanding environment
```
## Preprocessor Directives
```c
    #define, #undef                         // Constants, features (booleans), Macro functions */
    #ifdef, #ifndef, #else, #elif, #endif   // Conditional compilation
    #include                                // Include files
    #error, #warning                        // Compiler errors and warnings
    #pragma                                 // Compiler instructions
```
### Include guards
```c
    #ifndef ... #define ... #endif
    #pragma once
```
---
### Libraries
- **Static**    : Create using GNU tool ar
- **Shared**    : Preinstalled onto target  

---
## Make

| File name               | Description                       |
|-------------------------|-----------------------------------|
|*.d, *.dep       | Dependency file generated by make |
|*.mk, makefile, Makefile | Makefile                          |

### Makefile    
```makefile
    target: prerequisite1, prerequisite2, prerequisite3
        command1
        command2
```
```makefile
    #                   # Comment
    include <makefile>  # Include another <makefile>
    \                   # Line continuation
    $(<VAR>)            # Access variable <VAR> 
```
#### Variables
- Recursively expanded variables (**=**)    : Expands whenever used (Eg.: `CSTD=c89`)
- Simply expanded variables (**:=**)        : Expands once at time of definition (Eg: `ARCH:=$(shell arch)`)

#### Automatic Variables : *Variables in a receipe with a scope*
```makefile
    $@ # Target
    $^ # All prerequisites
    $< # First prerequisite
``` 

---
#### Pattern matching
```makefile
    % # Pattern matching operator
```
Example
```makefile
    %.o : %.c
        gcc -c $^ -o $@
# Example Result
    main.o : main.c
        gcc -c main.c -o main.o
```
For every *.c file, associate a .o with same filename  
```makefile
    OBJS:=$(SRCS:.c=.o)
# Example Result
    SRCS:=  main.c \
            myfile.c
    OBJS:=  main.o \
            myfile.o
```
---
```makefile
    .PHONY: <all|clean|debug|...> # To make sure that make doesnot confuse a target name with a file
```
#### **Functions & Dynamic variables**
```makefile
    $(shell <cmd>)
    ifeq ($(shell uname -s), 'Linux')
        CC=gcc
    endif
```
**Overiding variables**
```bash
    make <target> <var>=<val>
``` 
**Special variables**  

| Variable   | Meaning                   |
|------------|---------------------------|
| `CC`       | Compiler                  |  
| `CPP`      | Preprocessor program      |
| `AS`       | Assembler program         |
| `LD`       | Linker                    |
| `CFLAGS`   | C program flags           |
| `CPPFLAGS` | C preprocessor flags      |
| `ASFLAGS`  | Assembler flags           |
| `LDFLAGS`  | C program linker flags    | 
| `LDLIBS`   | Extra flags for libraries | 
    
## Memory
#### Flash Memory
- Read/Write page
- Erase block
## Data memory
- Can be allocated at 
    - Compile-time (`.data`, `.bss`)
    - Runtime (`.stack`, `.heap`)
### Segments  

| Segment  |                                                           |
|----------|-----------------------------------------------------------|
| `.stack` | Temporary data storage like temporary variables           |
| `.heap`  | Dynamic data storage                                      |
| `.data`  | non-zero initialized global and static data               |
| `.bss`   | zero initialized and uninitialized global and static data *(block starting symbol)* |  

## Code memory
### Segments 

| Segment                                               |              |
|-------------------------------------------------------|--------------|
| `.intvecs`                                            | Vector table |
| `.text`                                               | User program |
| `.const` / `.rodata`                                  | `const` data |
| `.cinit`, `.pinit`, `.init` / `.fini`, `.init_array`, `.ctors`, `.dtors` | Used at initialization (non-zero initial variable values) |
| `.bootloader`                                         |              |

---
### C Keywords
- Variable type : Fundamental, Dervied, Enumerated
- Type qualifier : `const` - Mapped to ROM (`.const` / `.rodata`)
- Type modifier : `unsigned`, `signed`, `short`, `long`
- Storage class : `auto` (Default, Automatically allocated and deallocated on `.stack`), `static`, `extern`, `register`
- `malloc` v/s `calloc`  
    - `malloc` : No initialization
    - `calloc` : Zero initialization
