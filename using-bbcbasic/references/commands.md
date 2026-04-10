# BBC BASIC keyword list

* `ABS`: This function gives the magnitude (absolute value) of a number (<factor>).
* `ACS`: This function gives the arc cosine of a number (<factor>).
* `ADVAL`: This function gives the value of the specified analogue port or buffer.
* `AND`: Bitwise logical `AND` between two integers. Priority 6.
* `APPEND`: This command appends a file to the program and renumbers the new lines.
* `ASC`: This function gives the `ASCII` code of the first character of a string.
* `ASN`: This function gives the arc sine of a number (<factor>).
* `ATN`: This function gives the arc tangent of a number (<factor>).
* `AUTO`
  * This command generates line numbers for typing in a program.
  * `AUTO [<base number>[,<step size>]]`
* `BEAT`: This function gives the current microbeat number.
* `BEATS`
  * `BEATS <expression>`: set the number of microbeats in a bar.
  * As a function `BEATS` gives the current number of microbeats.
* `BGET`: This function gives the next byte from the specified file: `BGET#<channel>`.
* `BPUT#<channel>,<number>`: put byte to open file.
* `BPUT#<channel>,<string>[;]`: put string to open file, with[out] newline.
* `CALL<expression>[,<variable>]^`: Call machine code.
* `CASE <expression> OF`: start of `CASE`..`WHEN`..`OTHERWISE`..`ENDCASE` structure.
* `CHAIN`: Load and run a new Basic program.
* `CHR$`: This function gives the one character string of the supplied `ASCII` code.
* `CIRCLE [FILL] <x>,<y>,<r>`: draw circle outline [solid].
* `CLEAR`: Forget all variables.
* `CLG`: Clear graphics screen.
* `CLOSE#<channel>`: close specified file.
* `CLS`: Clear text screen.
* `COLOUR <a> [TINT <t>]`: set text foreground colour, and tint. Sets background if <a> >= 128.
* `COLOUR <a>,<p>`: set palette entry for logical colour <a> to physical colour <p>.
* `COLOUR <red>,<green>,<blue>`: set colour to <red>,<green>,<blue>.
* `COLOUR <a>,<red>,<green>,<blue>`: set palette entry for a to <red>,<green>,<blue> physical colour.
* `COLOUR <a> [TINT <t>]`: set text foreground colour, and tint. Sets background if a>=128.
* `COLOUR <a>,<p>`: set palette entry for logical colour <a> to physical colour <p>.
* `COLOUR <red>,<green>,<blue>`: set colour to <red>,<green>,<blue>.
* `COLOUR <a>,<red>,<green>,<blue>`: set palette entry for <a> to <red>,<green>,<blue> physical colour.
* `COS`: This function gives the cosine of a number (<factor>).
* `COUNT`: This function gives the number of characters `PRINT`ed since the last newline.
* `CRUNCH`
  * This command removes specified spaces from the current program.
  * `CRUNCH <expression>`. The bits in the number mean:
  * 0: spaces before statements
  * 1: spaces in statements
  * 2: `REM` statements (except first)
  * 3: empty statements
  * 4: empty lines
* `DATA`
  * Introduces line of `DATA` to be `READ`. The list of items is separated by commas.
  * `LOCAL DATA, RESTORE DATA`: save and restore current `DATA` pointer.
* `DEF`
  * Define function or procedure: `DEF FN`|`PROC<name>[(<parameter list>)]`.
  * End function with =<expression>; end procedure with `ENDPROC`.
* `DEG`: This function gives the value in degrees of a number in radians.
* `DELETE`
  * This command deletes all lines between the specified numbers.
  * `DELETE <start line number>,<end line number>`
* `DIM fred(100,100)`: create and initialise an array.
* `DIM fred% 100`: allocate space for a byte array etc.
* `DIM(fred())`: function gives the number of dimensions.
* `DIM(fred(),n)`: function gives the size of the n'th dimension.
* `DIV`: Integer division, rounded towards zero, between two integers. Priority 3.
* `DRAW [BY] <x>,<y>`: graphics draw to [relative by] <x>,<y>.
* `EDIT`: This command calls the `ARM BASIC E`ditor.
* `ELLIPSE [FILL] <x>,<y>,<maj>,<min>[,<angle>]`: draw ellipse outline [solid].
* `ELSE`
  * Part of the IF..`THEN`..`ELSE` structure. If found at the start of line, it is part of the block
  * IF..`THEN`..`ELSE`..`ENDIF` structure.
  * `ELSE` can also appear in ON .. `GOTO`|`GOSUB`|`PROC` to set the default option.
* `END`
  * `END`: statement marking end of program execution.
  * `END=<expression>`: alter amount of memory allocated to `BASIC`.
  * As a function `END` gives the end address of memory used.
* `ENDCASE`: End of `CASE` structure at start of line. See `CASE`.
* `ENDIF`: End of block IF structure at start of line. See IF.
* `ENDPROC`: End of procedure definition.
* `ENDWHILE`: End of `WHILE` structure. See `WHILE`.
* `ENVELOPE`: `ENVELOPE` takes 14 numeric parameters separated by commas.
* `EOF#<channel>`: This function gives `TRUE` if at end of open file; else `FALSE`.
* `EOR`: Bitwise logical Exclusive-OR between two integers. Priority 7.
* `ERL`: This function gives the line number of the last error.
* `ERR`: This function gives the error number of the last error.
* `ERROR`
  * Part of `ON ERROR; LOCAL ERROR` and `RESTORE ERROR` statements.
  * Cause an error: `ERROR [EXT] <number>,<string>`.
* `EVAL`: This function evaluates a string: `EVAL("2*X+1")`.
* `EXP`: This function gives the exponential of a number (<factor>).
* `EXT`
  * This function gives the length (extent) of an open file: `EXT#<channel>`.
  * `EXT#<channel>`=<expression> sets the length of an open file.
* `FALSE`: This function gives the logical value 'false' i.e. 0.
* `FILL [BY] <x>,<y>`: flood fill from [relative to] point <x>,<y>.
* `FN`: Call a function with FNfred(<x>,<y>): define one with `DEF FN`fred(<a>,<b>).
* `FOR`: `FOR <variable> `= <start value> `TO <limit value> [STEP <step size>]`.
* `GCOL <a> [TINT <t>]`: set graphics foreground colour, and tint. Sets background if <a> >= 128.
* `GCOL <action>,<a> [TINT <t>]`: set graphics fore|background colour and action.
* `GCOL [<action>,]<red>,<green>,<blue>`: set colour to <red>,<green>,<blue>.
* `GET`: This function gives the `ASCII` value of the next character in the input stream.
* `GET$`
  * This function gives the next input character as a one character string.
  * `GET$#<channel>` gives next string from the file.
* `GOSUB <line number>`: call subroutine at line number.
* `GOTO <line number>`: go to line number.
* `HELP`: This command gives help on usage of the interpreter.
* `HIMEM`: This pseudo-variable reads or sets the address of the end of `BASIC`'s memory.
* `IF`
  * Single line if: `IF <expression> [THEN] <statements> [ELSE <statements>]`.
  * Block if: `IF <expression> THEN<newline>`
  *             `<lines>`
  * optional: `ELSE`
  *             `<lines>`
  * must:     `ENDIF`
* `INKEY 0 to 32767`: function waits <number> centiseconds to read character.
* `INKEY -255 to -1`: function checks specific key for `TRUE`|`FALSE`.
* `INKEY -256`: function gives operating system number.
* `INKEY$`: equivalent to `CHR$(INKEY`...)
* `INPUT [LINE]['|TAB|SPC]["display string"][,|;]<variable>`: input from user.
* `INPUT#<channel>,<list of variables>`: input data from open file.
* `INSTALL`: This command permanently installs a library: see `LIBRARY`.
* `INSTR(<string>,<sub string>[,<start position>])`: find sub string position.
* `INT`: This function gives the nearest integer less than or equal to the number.
* `LEFT$(<string>,<number>)`: gives leftmost number of characters from string.
* `LEFT$(<string>)`: gives leftmost LEN-1 characters.
* `LEFT$(<string variable>[,<count>])=<string>`: overwrite characters from start.
* `LEN`: This function gives the length of a string.
* `LET`: Optional part of assignment.
* `LIBRARY <string>`: functions and procedures of the named program can be used.
* `LINE`
  * `LINE <x1>,<y1>,<x2>,<y2>`: Draw a line
  * Part of `INPUT LINE` or `LINE INPUT` statement.
* `LIST`
  * This command lists the program:
  * `LIST [<line number>][,[<line number>]][IF<pattern>]`. list section [if pattern]
  * `LISTO <option number>`: list options. Bits mean:-
  * 0: space before line
  * 1: indent structure
  * 2: split lines at :
  * 3: don't list line number
  * 4: list tokens in lower case
* `LN`: This function gives the natural logarithm (base e) of a number (<factor>).
* `LOAD`: This command loads a new program.
* `LOCAL <list of variables>`: make things private to function or procedure.
* `LOCAL DATA`: save `DATA` pointer on stack.
* `LOCAL ERROR`: save error control status on stack.
* `LOG`: This function gives the common logarithm (base 10) of a number (<factor>).
* `LOMEM`: This pseudo-variable reads or sets the address of the start of the variables.
* `LVAR`: This command lists all variables in use.
* `MID$(<string>,<position>)`: gives all of string starting from position.
* `MID$(<string>,<position>,<count>)`: gives some of string from position.
* `MID$(<string variable>,<position>[,<count>])=<string>`: overwrite characters.
* `MOD`
  * Remainder after integer division between two integers. Priority 3.
  * The `MOD` function gives the square root of the sum of the squares of all the elements in a numeric
  * array.
* `MODE`
  * `MODE <number>|<string>`: set screen mode.
  * As a function `MODE` gives the current screen mode.
* `MOUSE <x>,<y>,<z>[,<t>]`: sets <x>,<y> to mouse position; <z> to button state [<t> to time].
* `MOUSE COLOUR <a>,<red>,<green>,<blue>`: set mouse palette entry for a to <red>,<green>,<blue> physical colour.
* `MOUSE OFF`: turn mouse pointer off.
* `MOUSE ON [<a>]`: sets mouse pointer 1 [or <a>].
* `MOUSE RECTANGLE <x>,<y>,<width>,<height>`: constrain mouse movement to inside rectangle.
* `MOUSE STEP <a>[,<b>]`: sets mouse step multiplier to <a>,<a> [or <a>,<b>].
* `MOUSE TO <x>,<y>`: positions mouse and pointer at <x>,<y>.
* `MOVE [BY] <x>,<y>`: graphics move to [relative by] <x>,<y>.
* `NEW`: This command erases the current program.
* `NEXT [<variable>[,<variable>]^]`: closes one or several FOR..`NEXT` structures.
* `NOT`: This function gives the number with all bits inverted (0 and 1 exchanged).
* `OF`: Part of the `CASE <expression> OF` statement.
* `OFF`
  * `OFF`: turn cursor off.
  * Part of `TRACE OFF, ON ERROR OFF` statements.
* `OLD`: This command recovers the program just after a NEW.
* `ON`: cursor on.
* `ON ERROR [LOCAL|OFF]`: define error handler.
* `ON <expression> GOTO|GOSUB|PROC.... ELSE`: call from specified list item.
* `OPENIN`: Open for Input: the function opens a file for input.
* `OPENOUT`: Open for Output: the function opens a file for output.
* `OPENUP`: Open for Update: the function opens a file for input and output.
* `OR`: Bitwise logical OR between two integers. Priority 7.
* `ORIGIN <x>,<y>`: sets <x>,<y> as the new graphics 0,0 point.
* `OSCLI <string>`: give string to Operating System Command Line Interpreter.
* `OTHERWISE`: Identifies case exceptional section at start of line. See `CASE`.
* `OVERLAY <string array>`: set an array of filenames for overlay libraries.
* `PAGE`: This pseudo-variable reads or sets the address of the start of the program.
* `PI`: This function gives the value of 'pi' 3.1415926535.
* `PLOT <n>,<x>,<y>`: graphics operation <n>.
* `POINT [BY] <x>,<y>`: set pixel at [relative to] <x>,<y>.
* `POINT TO <x>,<y>`: position pointer at <x>,<y> if not linked to mouse.
* `POINT [BY] <x>,<y>`: set pixel at [relative to] <x>,<y>.
* `POINT TO <x>,<y>`: position pointer at <x>,<y> if not linked to mouse.
* `POS`: This function gives the x-coordinate of the text cursor.
* `PRINT`
  * `PRINT [`'|TAB|`SPC]["display string"][<expression>][;]` print items in fields defined by @%
  * `PRINT#<channel>,<list of expressions>`: print data to open file.
* `PROC`: Call a procedure with `PROC`fred(<x>,<y>); define one with `DEF PROC`fred(<a>,<b>).
* `PTR`
  * This function gives the position in a file: `PTR#<channel>`.
  * `PTR#<channel>`=<expression> sets the position in a file.
* `QUIT`
  * `QUIT`: leave the interpreter.
  * As a function `QUIT` gives `TRUE` if `BASIC` was entered with a -quit option.
* `RAD`: This function gives the value in radians of a number in degrees.
* `READ <list of variables>`: read the variables in turn from `DATA` statements.
* `RECTANGLE`
  * `RECTANGLE [FILL] <xlo>,<ylo>,<width>[,<height>] [TO <xlo>,<ylo>]`:
  * Draw a rectangle outline [solid] or copy [move] the rectangle.
  * Width goes to the right. Height goes up the screen.
* `REM`: Ignores rest of line.
* `RENUMBER`
  * This command renumbers the lines in the program:
  * `RENUMBER [<base number>[,<step size>]]`
* `REPEAT`: start of `REPEAT`..`UNTIL` structure; statement delimiter not required.
* `REPORT`
  * `REPORT`: print last error message.
  * `REPORT$` function gives string of last error string.
* `RESTORE`
  * `RESTORE [+][<number>]`: restore the data pointer to first or given line, or move forward <number>
  * lines from the start of the next line.
  * `RESTORE DATA`: restore `DATA` pointer from stack.
  * `RESTORE ERROR`: restore error control status from stack.
* `RETURN`: End of subroutine.
* `RIGHT$(<string>,<number>)`: gives rightmost number of characters from string.
* `RIGHT$(<string>)`: gives rightmost character.
* `RIGHT$(<string variable>[,<count>])=<string>`: overwrite characters at end.
* `RND`: function gives a random integer.
* `RND(<n>) where <n> < 0`: initialise random number generator based on <n>.
* `RND(0)`: last `RND(1)` value.
* `RND(1)`: random real 0..1.
* `RND(<n>) where <n> > 1`: random value between 1 and `INT(<n>)`.
* `RUN`: Clear variables and start execution at beginning of program.
* `SAVE`: This command saves the current program.
* `SGN`: This function gives the values -1, 0, 1 for negative, zero, positive numbers.
* `SIN`: This function gives the sine of a number (<factor>).
* `SOUND <channel>,<amplitude>,<pitch>,<duration>[,<start beat>]`: make a sound.
* `SOUND ON|OFF`: enable|disable sounds.
* `SPC`: In `PRINT` or `INPUT` statements, prints out n spaces: `PRINT SPC(10)`.
* `SQR`: This function gives the square root of a number (<factor>).
* `STEP`: Part of the FOR..TO..`STEP` structure.
* `STEREO <channel>,<position>`: set the stereo position for a channel.
* `STOP`: Stop program.
* `STR$[~]<number>`: gives string representation [in hex] of a number (<factor>).
* `STRING$(<number>,<string>)`: gives string replicated the number of times.
* `SUM`
  * This function gives the sum of all elements in an array.
  * `SUMLEN` gives the total length of all elements of a string array.
* `SWAP <variable>,<variable>`: exchange the contents.
* `SYS`
  * The `SYS` statement calls the operating system:
  * `SYS <expression> [,<expression>]^ [TO <variable>[,<variable>]^[;<variable>]]`
* `TAB(`
  * In `PRINT` or `INPUT` statements:
  * `PRINT TAB(<n>)s$`: spaces to column n
  * `PRINT TAB(<x>,<y>)s$`: move to screen position <x>,<y>.
* `TAN`: This function gives the tangent of a number (<factor>).
* `TEMPO`
  * `TEMPO <expression>`: set the sound microbeat tempo.
  * As a function `TEMPO` gives the current microbeat tempo.
* `TEXTLOAD`: This command loads a new program, converting from text form if required.
* `TEXTSAVE`
  * This command saves the current program as text [with a `LISTO` option].
  * `TEXTSAVE[O <expression>,] <string>`
* `THEN`
  * Part of the IF..`THEN` structure. If `THEN` is followed by a newline it introduces a block structured
  * IF..`THEN`..`ELSE`..`ENDIF`.
* `TIME`
  * This pseudo-variable reads or sets the computational real time clock. `TIME$` reads or sets the
  * display version of the clock.
* `TINT`
  * `TINT <a>,<t>`: set the tint for `COLOUR`|`GCOL`|<fore>|<back> <a> to <t> in 256 colour modes.
  * Also available as a suffix to `GCOL` and `COLOUR`.
  * As a function `TINT(<x>,<y>)` gives the tint of a point in 256 colour modes.
* `TO`: part of FOR..TO..
* `TOP`: gives the address of the end of the program.
* `TRACE`
  * `TRACE [STEP] ON|OFF|PROC|<number>`: trace [in single step mode] on or off or procedure or function
  * calls or lines below the number.
  * `TRACE TO <string>`: send all output to stream <string>
  * `TRACE CLOSE`: close stream output. Expression: `TRACE` gives handle of the stream.
* `TRUE`: This function gives the logical value 'true' i.e. -1.
* `TWIN`: This command converts the program to text and calls Twin.
* `TWINO`: This command converts the program to text with a `LIST` option and calls Twin.
* `UNTIL <expression>`: end of `REPEAT`..`UNTIL` structure.
* `USR`: This function gives the value returned by a machine code routine.
* `VAL`: This function gives the numeric value of a textual string e.g. `VAL"23"`.
* `VDU`
  * `VDU <number>[;|][,<number>[;|]]`: list of values to be sent to vdu.
  * , only - 8 bits.
  * ; 16 bits.
  * | 8 bytes of zeroes.
* `VOICE <channel>,<string>`: assign a named sound algorithm to the voice channel.
* `VOICES <expression>`: set the number of sound voice channels.
* `VPOS`: This function gives the y-coordinate of the text cursor.
* `WAIT`: Wait for vertical sync.
* `WHEN`
  * `WHEN <expression>[,<expression>]^`: identifies case section at start of line.
  * See `CASE`.
* `WHILE <expression>`: start of `WHILE`..`ENDWHILE` structure.
* `WIDTH <expression>`: set width of output.
