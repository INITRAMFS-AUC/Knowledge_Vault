---
tags:
  - csce/debugging
---

## to compile 

`g++ <file-name> -g -o <executable-name>`

`-g` debugging symbols
`-o` output executable

___

## runtime 

`run`
`lay next`

`br <symbol (file,method,variable,line)>`

`info <command>`
`info break` : show breakpoints 

`enable 1` : enable (ID of breakpoint)
`disable 1` : disable (ID of breakpoint)

>[!note] Overloaded functions 
>when putting a breakpoint on an overloaded function GDB will put the breakpoints on all the functions of the same name


`print <var>`
`inspect <stl structure>`
### force execution
`set <var> = <value>`



### step into 
`n`
### step over 
`s`
### step out
`fin`

### continue
`continue` or `c`

---

## multithreading 

### Compiling
need `-lpthread` flag when compiling


### Runtime 

`info threads ` or `th`

switching threads
`t <id>` 

backtrace of the thread
`bt`

switching frames 
`f <id>`


---

## conditions 

`b <function> if <variable> <operator> <value>`

break on function if condition met

___

``--release`` and ``--g`` flags

compiler optimization flags
```
-03
-02
-01 (Default)

-g3 debugging with optimiation level 3
```






