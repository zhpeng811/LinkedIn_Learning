## Bash files
* **.bash_profile** is read when Bash is invoked as a login shell
* **.bashrc** is executed when a new shell is started

## Commands
* echo
    * **-n** don't print a trailing newline
    * **-e** enable backslash escaped characters like \n and \t
    * **-E** option to manually disable special characters, in case it is enabled by default
* typeset
    * makes the variable private to the function
* declare
    * **-l** converts string to lowercase
    * **-u** converts string to uppercase
    * **-r** does not allow assigning new value, similar to the **const** keyword
    * **-a** indexed array, indexing with integers (normal array)
    * **-A** associative array, indexing with strings (dictionary/hashmap/JSON)
 
# Files
* command &> file
    * *file* gets **stdout** and **stderr** from *command*, file is either created or overwritten
* command | command2
    * *command2*'s **stdin** comes from *command*'s **stdout**
* command 2>&1 | command2
    * *command2*'s **stdin** comes from *command*'s **stdout** and **stderr**
* command |& command2
    * same as above, *command2*'s **stdin** comes from *command*'s **stdout** and **stderr**
* command >> file
    * appends **stdout** of the *command* to the end of the *file*
* command &>> file
    * appends **stdout** and **stderr** of the *command* to the end of the *file*
* exec N< myfile
    * opens file descriptor *N* for reading from file *myfile*
* exec N> myfile
    * opens file descriptor *N* for writing to file *myfile*
* exec N<> myfile
    * opens file descriptor *N* for reading and writing with file *myfile*
* exec N>&- or exec N<&-
    * closes file descriptor *N*
* lsof
    * see what file descriptors for a process are open
    * use "**lsof -p $$**" to see what file descriptor the current shell has opened

# Unset or Null variables
* ${variable :- value}
    * if *variable* is unset/null, return *value*; otherwise, return value of *variable*
    * e.g. x = 5; $(x :- 3) == 5; $(y :- 8) == 8;
* $(variable := value)
    * if *variable* is unset/null then *variable* is assigned value & returned
    * e.g. $(z :- 10) = 10; z == 10;
* $(variable :?)
    * if *variable* is unset/null display an error and exit script
* $(variable :+ value)
    * if var is unset/null return nothing, otherwise return *value*;
* $(varible : offset)
    * value of *variable* starting at offset
    * e.g. str = "Hello World"; $(str:2) == "llo World";
* $(variable : offset:len)
    * value of *variable* starting at offset up to length *len*
    * e.g. str = "Hello World"; $(str:2:3) == "llo";
* $(#variable)
    * returns the length of *variable*
* $(variable#prefix)
    * remove matching prefix
* $(variable%post)
    * remove matching postfix/suffix
