# RISC OS Variable Handling

* `*Eval`
  * `*Eval <expression>`
  * *Eval evaluates an integer or string expression.
  * The expression analyser has the following operators:
  * +                       addition or string concatenation
  * -, *, /, MOD            integer operations
  * =, <, >, <=, >=, <>     string or integer comparison
  * >>, <<                  arithmetic shift right and left
  * >>>                     logical shift right
  * STR, VAL                conversion between strings and integers
  * AND, OR, EOR, NOT       (bitwise) logical operators
  * RIGHT, LEFT             substring extraction
  * LEN                     length of a string
  * LEAFNAME                extract the leaf filename from a string
  * DIRNAME                 extract the directory filename from a string
  * CANONICALISE            convert a filename string to its canonical form
  * TIMEFORMAT              format the current time in the format specified
  * SET                     determine whether a system variable is set
  * MODULEVERSION           return the version number of a module as a number (eg 101 for 1.01), or -1
  * if not present
  * 
  * Brackets can also be used.

* `*Key <keynumber> [<value>]`: *Key sets the function keys.

* `*Keyboard`
  * `*Keyboard [<country name>]`
  * *Keyboard with no parameter displays the currently selected keyboard.
  * Type *Countries to list available countries.

* `*Set`
  * `*Set <varname> <value>`
  * *Set assigns a string value to a system variable. Other types of value can be assigned with *SetEval
  * and *SetMacro.

* `*SetEval`
  * `*SetEval <varname> <expression>`
  * *SetEval evaluates an expression and assigns it to a system variable. Other types of value can be
  * assigned with *Set and *SetMacro.
  * "*Help Eval" describes the expression syntax.

* `*SetMacro`
  * `*SetMacro <varname> <value>`
  * *SetMacro assigns a macro value to a system variable. Other types of value can be assigned with `*Set` and `*SetEval`.

* `*SetPS`
  * `*SetPS [<printer server name>|<station number>]`
  * *SetPS changes the current printer server name or number. If no argument is supplied the current
  * setting reverts to the initialised state and will use configured state if the next open (or save)
  * does not explicitly quote either a name or a number

* `*SetType`
  * `*SetType <filename> <file type>`
  * *SetType sets the file type of the named file to the given textual file type or hexadecimal number.
  * If the file is not already datestamped then it is also stamped with the current time and date.
  * "*Show File$Type*" displays currently known file types.

* `*Show`
  * `*Show [<variablespec>]`
  * *Show lists system variables matching the name given, or all system variables if no name is
  * specified. Variables can be set with *Set, *SetEval and *SetMacro.

* `*ShowFree -FS <Filing system name> | -Path <Path> <Device>`: *ShowFree shows the amount of free space on devices.

* `*Unset <varname>`: *Unset deletes a system variable.
