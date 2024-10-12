###### gdb: https://users.ece.utexas.edu/~adnan/gdb-refcard.pdf
1. p \[***x*** | ***d*** | ***u*** | ***o*** | ***t*** | ***a*** | ***c*** | ***f***\] ***expr***: show value of expr \[with hexadecimal | signed decimal | unsigned decimal | octal | binary | address, absolute and relative | character | floating point format\]
  -  p \****array***\[@***len***:1\]: print array \[with len\]
2. c\\\[ontinue\]: continue running your program
3. n\\\[i\]: next source \[or machine instruction\]
4. s\\\[i\]:: step source \[or machine instruction\]
6. source ***/path/of/script***: read, execute GDB commands from file
7. b: set breakpoint at next instruction
    -  b \[***file***:\]***func***: set breakpoint at function \[in file\]
    -  b \[***file***:\]*ln*: set breakpoint at line number \[in file\]
    -  break \****addr***: set breakpoint at address
    -  break . . . if ***expr***: break conditionally on nonzero expr
8. cond ***n*** \[***expr***\]: new conditional expression on breakpoint n \[make unconditional if no expr\]
9. info
  -  i\\\[nfo\] ***b***\[***reak***\]***: show defined breakpoints
  -  i\\\[nfo\] ***w***\[***atch***\]: show defined watchpoints
  -  i\\\[nfo\] ***f***\[***rame***\] \[***addr***\]: describe selected frame, or frame at addr 
  -  i\\\[nfo\] ***args***: show arguments of selected frame
  -  i\\\[nfo\] ***locals***: show local variables of selected frame
  -  i\\\[nfo\] \[***reg | all-reg***\] \[***rn . . .***\]: show register values \[for regs rn\] in selected frame; all-reg includes floating point
1. quit: exit gdb
2. b\\\[ack\]t\[race\] \[***n***\]: print trace of all frames in stack \[or of n\]
3. f\\\[ame\] \[***n***\]: select frame number n or frame at address n \[if no n, display current frame\]
4. up \[***n***:1\]: select frame n frames up
5. down \[***n***:1\]: select frame n frames down
6. help ***command***: show command help doc
7. r\\\[un\] \[***arglist***\]: start your program \[with arglist\]
8. show
  -  show ***env***: show all environment variables
9. set
  -  set env ***var*** ***string***: set environment variable
  -  set ***var=expr***: evaluate expr without displaying it; use for altering program variables
1. unset
  -  unset env ***var***: unset environment variable
2. cd ***dir***: change working directory to dir
3. pwd: print working directory
4. shell ***command***: execute arbitrary shell command string
5. make . . . : call make . . . 
6. layout
  -  layout src: change layout to source code
  -  layout asm: change layout to assemble
  -  layout regs: change layout to registers
7. watch ***expr***: set a watchpoint for expression expr
8. delete \[***n***\]: delete all breakpoints \[or n\]
9. enable \[***once | delete***\] \[n\]: enable all breakpoints \[or n\] \[and disable | delete when reached again when reached\]
10. disable \[***\n***]: disable all breakpoints \[or n\]
11. ignore ***n*** ***count***: ignore breakpoint ***n***, ***count*** times
12. commands **n** \[***silent***\] ***command-list***: execute GDB command-list every time breakpoint n is reached. \[silent suppresses default display\]
13. clear: delete breakpoints at next instruction
  -  clear \[***file***:\]***func***: delete breakpoint at function \[in file\]
  -  clear \[***file***:\]*ln*: delete breakpoint at line number \[in file\]
14. https://stackoverflow.com/questions/5691193/gdb-listing-all-mapped-memory-regions-for-a-crashed-process
15. -p

###### rg:
1. -g
2. --type-list
3. --glob
4. --no-ignore
5. -w
6. --fixed-strings
7. --hidden
8. --files
9. --no-ignore-vcs
10. -l
11. --global
12. --iglob
13. --color
14. --no-heading
15. --with-filename
16. --line-number
17. --column
18. --sort-files
19. -j
20. --type-not
21. 

###### fd:
1. -l
2. -X
3. -v
4. -V
5. --ignore
6. --hidden
7. --no-ignore-vcs
8. --type
9. --exclude
10. --no-ignore
11. 

###### fzf:
1. --multi
2. --border
3. --no-sort
4. --history
5. --ansi=true
6. --keep-left
7. --prompt=
8. --with-nth=
9. --keep-right
10. --pointer=
11. --toggle-sort=
12. --info=inline-right
13. --maker=
14. --layout=
15. --highlight-line
16. --preview="bat"
17. --scrollbar=
18. --color=always
19. --style=numbers
20. --theme=gruvbox
21. --line-range=
22. --preview-window
23. --print-query
24. --query=
25. --wrap
26. --bind=
27. 

###### exa:

###### r2:

###### sed:
1. \\\[-i \<text\> \<file\>\]: append \<text\> into \<file\>\

###### dpkg:
1. \\\[-l\]: list all installed package
2. \\\[-L \<name\>\]: cheak the files included in an installed package by \<name\>
3. \\\[--contents \<file\>\]: cheak the files included in a \<.deb file\>
4. \\\[-r \<name\>\]: uninstalled package by \<name\>
5. \\\[-i \<file\>\]: install package from \<.deb file\>

###### update-alternatives:
1. \\\[--get-selections\]: list all master alternative names and their status
2. \\\[--query \<name\>\]: check alternative info by \<name\>

###### gcc/g++:
1. \\\[-c\]: 
2. \\\[-L\]: 
3. \\\[-l\]
4. \\\[-O\[n:3\]\]: 
5. \\\[-g\]: 
6. \\\[-o\]: 
7. \\\[-f\<a\>\]: 
  -  \\\[a:PIC\]:
  -  \\\[a:dump-class-hierarchy\]
  -  \\\[a:PIE:\] 
  -  \\\[a:no-default-inline\]
8. \\\[-W\<a\>\]:\
  -  \\\[a:all\]:
  -  \\\[a:l,-rpath,./\]
9. \\\[-shared\]: 
10. \\\[-no-pie\]: 
11. \\\[-\]: 
12. \\\[-o\]: 
13. \\\[-std=\]
14. -v -x c -o /dev/null - < /dev/null

###### ldd \<file\>: 

###### ar:

###### git:
1. \\\[log --pretty-format --author]\
2. \\\[clone\]
3. \\\[submodule update --remote\]
4. \\\[diff --name-status]
5. \\\[push\]
6. \\\[status --porcelain --ignore-submodules\]
7. \\\[add\]
8. \\\[commit -m\]
9. \\\[clean -fxnd\]
10. \\\[branch\]
11. \\\[show\]
12. \\\[remote -v\]
13. \\\[stash --include-untracked\]
14. \\\[merge\]
15. \\\[switch\]
16. \\\[checkout -b -C -c -v\]
20. \\\[pull\]
21. \\\[rebase -i -continue\]
23. \\\[reset --hard HEAD\]
24. \\\[fetch\]
25. \\\[diff-index HEAD --quit ...\]
26. \\\[revert HEAD ...\]
27. \\\[-C ...\]

###### objdump:
1. -d: 
2. -r: 
3. -h
4. -t
5. -R
6. --section-name

###### readelf:
1. -d
2. -r
3. -a
4. -D
5. -S
6. -h
7. --all
8. --dynamic

###### eu-readelf
1. -d
2. -r
3. -a

###### ps -ayux

###### kill -9 ...

###### relinfo

###### cmake
1. -E
2. --build
3. --config
4. -D

###### luarocks
1. --lua-version
2. --global
3. install
4. 
