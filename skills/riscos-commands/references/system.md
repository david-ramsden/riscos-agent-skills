# RISC OS General System Commands

* `*AIF`
  * `*AIF <file> [<arguments>]`
  * *AIF is used to execute files with the type 'absolute'.
  * The system variable AIF$Options can be set to a string of characters to override AIF checks :
  *         O Accept over-long files
  *         T Accept truncated files
  *         D Accept files with bad debug data descriptors
  *         C Accept unsuitable code bit size
  *         A Accept non-AIF files
  * 
  * Overriding these checks may compromise system stability.

* `*AMInfo`
  * `*AMInfo [-Plugins] [<file>]`
  * *AMInfo prints information about the player module status, and about the file playing, if any. When
  * -P[lugins] is present, information about the currently registered plugins is given instead.

* `*AMLocate [+|-][<hours>:<minutes>:<seconds>|<minutes>:<seconds>]`: *AMLocate locates the specified point in the file currently playing/paused.

* `*AMPause [-Off]`: *AMPause turns on and off pause mode.

* `*AMPlay`
  * `*AMPlay [-Next] [-Queue|-Cue] [-Transient] [<filename>]`
  * *AMPlay starts playing an Audio MPEG file. When -Q[ueue] is present, the file will be played after
  * the current one finishes, otherwise immediately. When -C[ue] is present, the file will be started,
  * but with the player in pause mode.When -T[ransient] is present, the file will be started in a new
  * instance which will terminate when playfile finishes.

* `*AMStop`: *AMStop stops the MPEG file playing

* `*AMVolume [+|-]<number>`: *AMVolume sets the playback volume. Range is 0 to 127, with 113 as initial value.

* `*Alphabet`
  * `*Alphabet [<country name> | <alphabet name>]`
  * *Alphabet with no parameter displays the currently selected alphabet.
  * Type *Alphabets to list available alphabets.

* `*Alphabets`: *Alphabets lists the available alphabets.

* `*AppSize <size>`: *AppSize reserves space in application workspace.

* `*Append`
  * `*Append <filename>`
  * *Append opens an existing file and subsequent lines of keyboard input are appended to it, input
  * being terminated by ESCAPE.

* `*Audio ON|OFF`: *Audio controls the sound system.

* `*BASIC [-help|-chain|-load|-quit] <filename>`: BASIC is the BASIC interpreter.

* `*BASIC [-help|-chain|-load|-quit] <filename>`: BASIC is the BASIC interpreter.

* `*BlankTime`
  * `*BlankTime [W|O] [<time>]`
  * *BlankTime sets options or BlankTime (seconds) for the Blanker.
  *         -W claims WriteCV.
  *         -O releases WriteCV.
  * If used with no parameters, it displays the current status.
  * To turn screen blanking off use '*BlankTime 0'.

* `*BreakClr`
  * `*BreakClr [<addr|reg>]`
  * *BreakClr removes the breakpoint at the specified address. If no address is given then all
  * breakpoints are removed.

* `*BreakList`: *BreakList lists all the currently set breakpoints.

* `*BreakSet <addr|reg>`: *BreakSet sets a breakpoint at the given address.

* `*Configure Cache on|off`: *Configure Cache determines the default state of the cache.

* `*ChangeDynamicArea`
  * `*ChangeDynamicArea [-FontSize <n>[K]] [-SpriteSize <n>[K]] [-RamFsSize <n>[K]] [-RMASize`
  * *ChangeDynamicArea allows the size of the font cache, system sprite area, RAM disc, module area and
  * screen memory to be set up.
  * <n>[K]] [-ScreenSize <n>[K]]

* `*ChannelVoice <channel> <voice index>|<voice name>`: *ChannelVoice attaches a Voice to a Sound Channel.

* `*ColourTransLoadings`: ColourTrans commands are for internal use only

* `*ColourTransMap`: ColourTrans commands are for internal use only

* `*ColourTransMapSize`: ColourTrans commands are for internal use only

* `*Configure`
  * *Configure <item> [<parameter>] sets the CMOS RAM options.
  * *Status displays the current options.

* `*Connect <name> <server> <dir-name> [<user-name> <password>]`: *Connect sets up a connection to an AppleTalk file server.

* `*Continue`: *Continue restarts execution from the breakpoint saved state.

* `*DPMSTime`
  * `*DPMSTime [<time>]`
  * *DPMSTime (seconds) sets the delay before an idle machine enters DPMS.
  * If used with no parameters, it displays the current status.
  * To turn auto DPMS off use '*DPMSTime 0'.

* `*DST`
  * `*Configure DST | NODST`
  * *Configure DST sets the clock for Daylight Saving Time
  * *Configure NODST sets the clock for Local Standard Time.

* `*Debug`: *Debug gives access to debugging facilities.

* `*DiagnosticDump`
  * The DiagnosticDump module provides a record of the output from C programs which have exited
  * abnormally. By default the behaviour is to open a filer window with the record of the backtrace.
  * This behaviour can be controlled by the system variable DiagnosticDump$Options. This is a sequence
  * of option characters followed by qualifiers. At present the following options are available:
  *         Y[<level>]      Enable module and set level of detail (0-3 in increaseding order of size)
  *         O               Automatically open dump filer window.
  * Unset the system variable to disable the module.

* `*Disconnect <name>`: *Disconnect disconnects from an AppleTalk file server.

* `*Do <command>`: *Do passes its argument to the command interpreter

* `*Echo <string>`: *Echo sends a string to the VDU, after transformation by GSRead.

* `*Error [<number>] <text>`: *Error generates an error with the given number and text.

* `*FX r0 [[,] r1 [[,] r2]]`: *FX calls OS_Byte.

* `*Configure FileSystem <fs name>|<fs number>`: *Configure FileSystem sets the default filing system.

* `*Configure Floppies <floppies>`: *Configure Floppies sets the number of floppy disc drives attached.

* `*FontCat`: *FontCat lists the fonts in <Font$Path>

* `*FontInstall`
  * `*FontInstall [<prefix>]`
  * Installs a font directory for use by the Font Manager, and also ensures that the directory is
  * rescanned.

* `*FontLibrary`
  * `*FontLibrary <directory>`
  * Temporarily installs a font directory for use by the Font Manager. There is normally only one such
  * directory at a time.

* `*FontList`: *FontList lists the fonts currently cached

* `*Configure FontMax <number>[k]`: Max font cache size (cache grows while fonts in use)

* `*Configure FontMax1 <number>`: Max point height of rescaled bitmaps (use outlines if bigger)

* `*Configure FontMax2 <number>`: Max point height of anti-aliased text (1-bpp if bigger)

* `*Configure FontMax3 <number>`: Max point height of cached bitmaps generated from outlines (draw directly if bigger)

* `*Configure FontMax4 <number>`: Max point width for horizontal subpixel adjustment

* `*Configure FontMax5 <number>`: Max point height for vertical subpixel adjustment

* `*FontRemove <prefix>`: Removes a font directory from the Font Manager's list.

* `*FreePool`: *FreePool moves all available memory except for the next slot into the free pool

* `*GO`
  * `*GO [<address>] [; environment]` - go to address (hexadecimal), default &8000. Text after ';' is
  * environment string.

* `*GOS`: *GOS enters the supervisor. Use *Quit to exit.

* `*HOff`: *HOff switches the hourglass off.

* `*HOn`: *HOn switches the hourglass on.

* `*Help`: *Help <subjects> attempts to give useful information on the selected topics.

* `*Configure IDEDiscs <IDE hard discs>`: *Configure IDEDiscs sets the number of IDE hard discs attached.

* `*Ignore [<number>]`: *Ignore sets the printer ignore character.

* `*InitStore`
  * `*InitStore [<data|reg>]`
  * *InitStore fills user memory with the specified data, or the value &E7FFFFFF (an illegal ARM
  * instruction) if no parameter is given.

* `*LoadCMOS <file>`: *LoadCMOS configures the computer from a configuration file.

* `*LoadFontCache <filename>`: Load font cache from a file (only allowed if no fonts are claimed)

* `*LoadModeFile`
  * `*LoadModeFile <filename>`
  * *LoadModeFile reads screen mode definitions from the given file and makes these the current screen
  * modes set.

* `*Lock`
  * *Lock prevents the drawer from opening.
  * *Syntax: Lock [drive]

* `*Memory [B] <addr1|reg1> [[+|-] <addr2|reg2> [+ <addr3|reg3>]]`: *Memory displays the values in the memory in ARM words.

* `*MemoryA [B] <addr|reg1> [<data|reg2>]`: *MemoryA displays and alters the memory contents in bytes or words.

* `*MemoryI [T] <addr1|reg1> [[+|-] <addr2|reg2> [+ <addr3|reg3>]]`: *MemoryI disassembles ARM or Thumb instructions.

* `*MiniUnzip`
  * `*MiniUnzip [-h] [-v] [-l] [-i] <input> [[-o] <output>]`
  * Command is: MiniUnzip       0.17 (09 Oct 2006)
  * 
  * *MiniUnzip will decompress Zip archives.
  * 
  *  -h     display help
  *  -v     verbose decompression
  *  -l     just list files in archive
  *  -i     specify input filename
  *  -o     specify output filename
  *  -u     Translate unix style ,xxx filenames to RISC OS types

* `*ModelList`: *ModelList lists all the loaded colour models

* `*Modules`
  * *Modules lists the modules currently loaded, giving the name and address of the module, and also the
  * address of its workspace.
  * See also *ROMModules.

* `*NODST`
  * `*Configure DST | NODST`
  * *Configure DST sets the clock for Daylight Saving Time
  * *Configure NODST sets the clock for Local Standard Time.

* `*Obey`
  * `*Obey [[-n][-v] [<filename> [<parameters>]]]`
  * *Obey executes a file of *commands, performing argument substitution on each line. Prefixing the
  * filename with -v causes each line to be echoed before execution. Prefixing the filename with -n
  * prevents the system variable Obey$Dir from being set before execution.

* `*PatchStats`
  * *PatchStats displays statistics about patches applied to applications and modules. Usually this is
  * used to correct faults in StrongARM-unaware applications, although some patches may be applied to
  * correct more common faults.

* `*PipeCopy <inputfile> <outputfile> [<outputfile>]`: *PipeCopy copies files one byte at a time

* `*PoduleLoad <expansion card number> <filename> [<offset>]`: *PoduleLoad copies a file into the RAM of a specified expansion card

* `*PoduleSave <expansion card number> <filename> <size> [<offset>]`: *PoduleSave copies the ROM of a specified expansion card into a file

* `*Podules`: *Podules gives information about expansion cards installed in the computer

* `*Pointer [0|1]`: *Pointer turns the mouse pointer on/off.

* `*Print`
  * `*Print <filename>`
  * *Print displays the contents of a file by sending each byte to the VDU.
  * See also *List and *Type.

* `*QSound <chan> <amp> <pitch> <duration> <nTicks>`: *QSound queues a sound after the specified number of tempo ticks.

* `*Quit`: *Quit leaves the current application.

* `*RMClear`: *RMClear deletes all relocatable modules from the RMA.

* `*RMEnsure`
  * `*RMEnsure <moduletitle> <version number> [<*command>]`
  * *RMEnsure checks that a module is present and is the given version, or a more modern one. The
  * command is executed if this is not the case.

* `*RMFaster <moduletitle>`: *RMFaster moves a module from ROM to RAM.

* `*RMInsert <moduletitle> [<podule number>]`: *RMInsert reverses the effect of *Unplug, but does not reinitialise the specified ROM module.

* `*RMKill <moduletitle>`: *RMKill kills and deletes a relocatable module.

* `*RMLoad <filename>`: *RMLoad loads and initialises a relocatable module.

* `*RMReInit <moduletitle>`: *RMReInit reinitialises a relocatable module, reversing the action of *Unplug if appropriate.

* `*RMRun <filename>`: *RMRun runs a relocatable module.

* `*RMTidy`: *RMTidy compacts the RMA and reinitialises all the modules.

* `*ROMModules`
  * *ROMModules lists the relocatable modules currently in ROM, along with their status.
  * See also *Modules.

* `*ReadMimeMap`: *ReadMimeMap rereads the MIME mappings file.

* `*RemoveTinyDir <pathname>`: *RemoveTinyDir removes a file, application or directory icon from the desktop icon bar.

* `*Render`
  * `*Render [-file] <filename> [<m00> <m01> <m10> <m11> <m20> <m21>] [-bbox] [-suppress] [-flatness <flatness>]`
  * *Render displays the contents of a draw file, using a transformation matrix.
  * Options:
  * -bbox           draw bounding boxes
  * -suppress       do not draw objects
  * -flatness       tolerance for curved paths

* `*Repeat`
  * `*Repeat <command> <directory> [-directories | -files | -applications | -type <type>] <tail> [-tasks] [-verbose]`
  * *Repeat iterates over a directory, performing a command for each object found.
  * Options:
  * -directories    limit search to directories
  * -files          limit search to files
  * -applications   limit search to applications
  * -type <type>    limit search to files of a given type
  * -tasks          start each command as a separate task
  * -verbose        give an indication of progress

* `*SaveFontCache <filename>`: Save font cache to a file

* `*SChoose <name>`: *SChoose selects a sprite.

* `*SCopy <name> <new name>`: *SCopy makes a copy of a sprite.

* `*SDelete <name> [<name>]`: *SDelete deletes sprites.

* `*SDisc [:]<disc name>`: *SDisc is synonymous with *Mount.

* `*SFlipX <name>`: *SFlipX reflects the sprite about the X axis.

* `*SFlipY <name>`: *SFlipY reflects the sprite about the Y axis.

* `*SGet <name>`: *SGet picks up an area of the screen as a sprite.

* `*ShowRegs`: *ShowRegs displays the stored ARM registers.

* `*SInfo`: *SInfo prints the size of the sprite memory.

* `*SList`: *SList lists all sprites.

* `*SLoad <filename>`: *SLoad loads a sprite file into memory.

* `*SMerge <filename>`: *SMerge appends a sprite file to those in memory.

* `*SNew`: *SNew clears all sprite definitions.

* `*SRename <old name> <new name>`: *SRename renames a sprite.

* `*SSave <filename>`: *SSave saves the sprite memory.

* `*ScreenLoad <filename>`: *ScreenLoad loads into the graphics window.

* `*ScreenSave [-native] <filename>`: *ScreenSave saves the graphics window (or screen). With `-native` saves the graphics window as a native file format (usually PNG).

* `*Shadow`: *Shadow makes subsequent mode changes use the other screen bank.

* `*Sound <chan> <amp> <pitch> <duration>`: *Sound makes a foreground (immediate) sound.

* `*Configure SoundDefault <0|1> <0-7> <1-16> (speaker, volume, voice)`: *Configure SoundDefault sets default sound channel one parameters.

* `*SoundGain <gain> where <gain> is 0-7 for 0dB (default) to +21dB gain, in 3dB steps`: *SoundGain sets the gain for 8-bit mu-law to 16-bit linear sound conversion.

* `*Configure SoundSystem 8bit | 16bit [Oversampled] | <0-7>`: *Configure SoundSystem sets the default sound system.

* `*Speaker ON|OFF`: *Speaker controls the loudspeaker.

* `*StartDesktopTask <*command>`: Cause a task to start next time the desktop environment is entered.

* `*Status`: *Status shows the selected CMOS RAM options. Use *Configure to set the options.

* `*Configure Step <step delay> [<drive>]`: *Configure Step sets the step rate of one or all floppy disc drives.

* `*Stereo <chan> <pos> where <chan> is 1-8, <pos> is -127(L) to 127(R) (0 for centre)`: *Stereo sets the stereo position of a sound channel.

* `*TV needs 0 to 2 parameters.`: *TV [<vertical position> [[,] <interlace>]] sets the position of the display on the screen.

* `*Tempo <n> (0 - &FFFF, default is &1000)`: *Tempo sets the system tempo.

* `*Territories`: *Territories lists the currently loaded territory modules.

* `*Configure Territory <Territory Number>`: *Configure Territory sets the default territory for the machine.

* `*Time`: *Time displays the time and date.

* `*Configure TimeZone [+/-]<Hours>[:<Minutes>]`: *Configure TimeZone sets the time zone as an offset from UTC

* `*Toolbox_Objects`: Display a list of the Objects registered with the Toolbox module. The list is sorted by class identifier.

* `*Configure Truncate on|off`: *Configure Truncate sets whether filenames should be truncated when too long.

* `*Tuning -&offf to &offf (-16383 to 16383) (where 'o' is octave, 'fff' is fraction of octave)`: *Tuning alters the relative system tuning. *Tuning 0 resets tuning to default.

* `*Unplug`
  * `*Unplug [<moduletitle> [<podule number>]]`
  * *Unplug stops the given ROM module being initialised.
  * *Unplug with no argument lists the unplugged ROM modules.

* `*VIDCBandwidthLimit <bandwidth> <bandwidth> <bandwidth>`: *VIDCBandwidthLimit is for internal system use only.

* `*Voices`: *Voices lists the installed voices and channel allocation.

* `*Volume <n>`: *Volume sets the audio channel loudness; range 1-127.

* `*X <command>`: *X passes its argument to the command interpreter, storing any error in a system variable.

* `*XPin <pathname> <x> <y>`: *XPin adds a file, application or directory to the desktop pinboard ignoring any errors.

