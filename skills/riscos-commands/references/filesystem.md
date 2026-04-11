# RISC OS Filesystem Operations

* `*ADFS`: *ADFS selects ADFS as the current filing system.

* `*ADFSDirCache`
  * `*Configure ADFSDirCache <size>[K]`
  * *Configure ADFSDirCache sets the size of the directory cache (in Kilobytes) used by ADFS. A value of
  * 0 selects a default value which depends on RAM size.

* `*ADFSbuffers`
  * `*Configure ADFSbuffers <buffers>`
  * *Configure ADFSbuffers sets the number of extra 1024 byte file buffers taken by ADFS to speed up
  * operations on open files. A value of 1 selects the default number of buffers for the RAM size, and 0
  * disables fast buffering.

* `*Access`
  * `*Access <object> [<attributes>]`
  * *Access changes the attributes of all objects matching the wildcarded specification.
  * Attributes:
  * L(ock)          Lock object against deletion
  * R(ead)          Read permission
  * W(rite)         Write permission
  * /R,/W,/RW       Public read and write permission

* `*AppPath <path-variable> <path-component>`: Append a given path component onto a path variable, ensuring it only appears once.

* `*Back`: *Back swaps the current and previous directories.

* `*Backup <source drive> <dest. drive> [Q]`: *Backup copies one whole floppy disc, (except free space) to another.

* `*Build`
  * `*Build <filename>`
  * *Build opens a new file and subsequent lines of keyboard input are directed to it, input being
  * terminated by ESCAPE.

* `*Bye`
  * `*Bye [[:]<file server name>|<station number>]`
  * *Bye terminates your use of the current (or given) file server. Closing all open files and
  * directories.

* `*Cat [<directory>]`: *Cat lists all the objects in a directory (default is current directory).

* `*CDDevices`: *CDDevices displays all CD devices attached.

* `*Configure CDROMBuffers <buffersize>`: *Configure CDROMBuffers sets the buffer size used by CDFS.

* `*Configure CDROMDrives <drives>`: *Configure CDROMDrives sets the number of CD-ROM drives attached.

* `*CDFS`: *CDFS selects CDFS as the current filing system.

* `*CDSpeed [drive] [speed]`: *CDSpeed displays or sets the CD-ROM read speed.

* `*CDir <directory> [<size in entries>]`: *CDir creates a directory of given name (and size on Net only).

* `*CheckMap`
  * `*CheckMap [<disc spec.>]`
  * *CheckMap checks that the map of a new map disc has the correct checksums, and is consistent with
  * the directory tree. If only one copy is good it allows you to rewrite the other.

* `*Close`: *Close closes all files on the current filing system.

* `*Compact [<disc spec.>]`: *Compact tries to collect free spaces together by moving files.

* `*Copy`
  * `*Copy <source spec> <destination spec> [<options>]`
  * *Copy copies one or more objects that match the given wildcarded specification between directories.
  * Options are taken from the system variable Copy$Options and those given to the command.
  * Options: (use ~ to force off,eg. ~C)
  * A(ccess)        Force destination access to same as source {on}
  * C(onfirm)       Prompt for confirmation of each copy {on}
  * D(elete)        Delete the source object after copy {off}
  * F(orce)         Force overwriting of existing objects {off}
  * L(ook)          Look at destination before loading source file {off}
  * N(ewer)         Copy only if source is more recent than destination {off}
  * P(rompt)        Prompt for the disc to be changed as needed in copy {off}
  * Q(uick)         Use application workspace to speed file transfer {off}
  * R(ecurse)       Copy subdirectories and contents {off}
  * S(tamp)         Restamp datestamped files after copying {off}
  * sT(ructure)     Copy only the directory structure {off}
  * V(erbose)       Print information on each object copied {on}

* `*CopyBoot <src-drive> <dest-drive>`: *CopyBoot allows the MS-DOS BOOT BLOCK from one floppy to be copied over the BOOT BLOCK of another.

* `*Count`
  * `*Count <pattern> [<options>]`
  * *Count adds up the size of one or more files that match the given wildcarded specification. Options
  * are taken from the system variable Count$Options and those given to the command.
  * Options: (use ~ to force off,eg. ~R)
  * C(onfirm)       Prompt for confirmation of each count {off}
  * R(ecurse)       Count subdirectories and contents {on}
  * V(erbose)       Print information on each file counted {off}

* `*Countries`: *Countries lists the available countries.

* `*Country`
  * `*Configure Country <country name>`
  * *Configure Country controls which country setting the computer will use on a hard reset, which in
  * turn determines which alphabet and keyboard driver is used.
  * Type *Countries to list available countries.

* `*Create`
  * `*Create <filename> [<length> [<exec addr> [<load addr>]]]`
  * *Create reserves space for the named file, optionally giving it load and execution addresses. No data is transferred to the file. Length and addresses are in hexadecimal.

* `*DOSMap`
  * `*DOSMap [<MSDOS extension> [<RISC OS filetype>]]`
  * *DOSMap specifies an MSDOS extension to RISC OS filetype mapping. The RISC OS filetype can be given
  * as either a 12bit HEX ASCII number or as the text identifier defined by a suitable File$Type_XXX
  * variable. If no RISC OS filetype is given, then the existing MSDOS extension (if present) will be
  * removed. If no parameters are given then the current mappings are displayed.

* `*Defect`
  * `*Defect <disc spec.> <disc add.>`
  * *Defect maps out a defect from a new map disc if it lies in an unallocated part of the disc.
  * Otherwise it searches for the object containing the defect.

* `*Delete`
  * `*Delete <filename>`
  * *Delete tries to delete the named file or directory, returning an error if the object does not exist. Directories can only be deleted when they contain no files.
  * See also *Wipe, and *Remove.

* `*Dir [<directory>]`: *Dir selects a directory as the current directory (default is user root directory).

* `*Dismount <:discname>`: *Dismount closes files on, and discards local caches for, a remote shared disc.

* `*Drive <drive>`: *Drive sets the current drive.

* `*Dump`
  * `*Dump <filename> [<file offset> [<start address>]]`
  * *Dump displays the contents of the file as a hex and ASCII dump. The file offset and start address
  * are in hexadecimal.

* `*Eject [drive]`: *Eject opens the drawer.

* `*EnumDir`
  * `*EnumDir <directory> <output file> [<pattern>]`
  * *EnumDir creates a file of filenames from a directory that match the supplied wildcarded
  * specification (default is *).

* `*Ex`
  * `*Ex [<directory>]`
  * *Ex lists all the objects in a directory together with their file information (default is current
  * directory).

* `*Exec`
  * `*Exec [<filename>]`
  * *Exec <filename> directs the operating system to take further input from the given file.
  * *Exec with no filename causes the exec file to be closed.

* `*FileInfo <object>`: *FileInfo yields the full file information of an object.

* `*Format`
  * `*Format <drive> [<format> [<disc name>]] [Y]`
  * Format prepares a floppy disc for use with the ADFS.
  * F - 1600K, 77 entry directories, new map, Archimedes ADFS 2.50 and above.
  * E - 800K, 77 entry directories, new map, Archimedes ADFS 2.00 and above.
  * D - 800K, 77 entry directories, old map, Archimedes ADFS.
  * L - 640K, 47 entry directories, old map, all ADFS.
  * E+ - 800K, variable directory entries, RISC OS 4 and above.
  * F+ - 1600K, variable directory entries, RISC OS 4 and above.
  * DOS/Q - 1440K, DOS 3.5" high density disc
  * DOS/M -  720K, DOS 3.5" disc
  * DOS/H - 1200K, DOS 5.25" high density disc
  * DOS/N -  360K, DOS 5.25" disc
  * DOS/P -  180K, DOS 5.25" disc
  * DOS/T -  320K, DOS 5.25" disc
  * DOS/U -  160K, DOS 5.25" disc
  * Atari/M - 720K, Atari ST (DS), 3.5" disc
  * Atari/N - 360K, Atari ST (SS), 3.5" disc
  * PCMCIA - PCMCIA 2.0 SRAM card
  * The default is E.

* `*Free [<mount spec.>]`: *Free displays the total free space on a mounted volume.

* `*FS`: *Configure FS <name> sets the file server or domain name from which LanManFS will attempt to boot.

* `*FSLock_ChangePassword <FSName> [New [New [Old]]]`: *FSLock_ChangePassword changes the password used to unlock the system.

* `*FSLock_Lock`: *FSLock_Lock locks machine against modification.

* `*FSLock_Status`: *FSLock_Status reports on whether a filing system is currently locked or not.

* `*FSLock_Unlock`
  * `*FSLock_Unlock [-full] [Password]`
  * *FSLock_Unlock unlocks the currently locked filing system when given the right password. If the
  * -full flag is not given then the filing system is only unlocked till the next reset of the machine.

* `*HostFS`: *HostFS selects the HostFS filing system

* `*Info <object>`: *Info lists the file information of all objects matching the given wildcarded specification.

* `*LanMan`: *LanMan selects Lan Manager as the current filing system.

* `*LCat [<directory>]`: *LCat lists all the objects in a subdirectory relative to the library (default is current library).

* `*LEx`
  * `*LEx [<directory>]`
  * *LEx lists all the objects in a subdirectory of the library together with their file information
  * (default is current library).

* `*Lib`
  * `*Configure Lib <0|1>`
  * *Configure Lib 0 will mean that logon will select the default library, if it exists. *Configure Lib
  * 1 means that the library 'ArthurLib' will be selected at logon.

* `*List`
  * `*List [-File] <filename> [-TabExpand]`
  * *List displays the contents of the file in the configured GSRead format. Each line is preceded with
  * a line number.
  * See also *Print and *Type.

* `*ListFS`
  * `*ListFS [-force]`
  * *ListFS shows those file servers that the NetFS currently knows about. If the optional argument is
  * supplied then the list will be refreshed before it is displayed.

* `*ListPS`
  * `*ListPS [-full]`
  * *ListPS shows the names of all the currently available printer servers. If the optional argument is
  * supplied then the status of each server will also be displayed.

* `*Load`
  * `*Load <filename> [<load addr>]`
  * *Load with no specified address loads the named file at its own load address. If a load address
  * (hexadecimal) is specified, it will be used instead.

* `*LMInfo`: *LMInfo displays debugging information.

* `*LMLogoff`: *LMLogoff clears the workgroup and default user settings and disables network browsing.

* `*LMLogon <workgroup> <user-name> [<password>]`: *LMLogon sets default information about the network.

* `*LMNameMode 0 | 1 | 2`: *LMNameMode sets the way LanManFS capitalises names.

* `*LMPrinters <server> [<printername>] [<printername>] ...`: *LMPrinters adds a server name and list of printers.

* `*LMServer <server> [<sharename>] [<sharename>] ...`: *LMserver adds a server name and list of shared drives.

* `*LMStats`: *LMStats shows network statistics.

* `*LMTransport`: *Configure LMTransport sets whether LanManFS should use NetBEUI or TCP/IP as transport protocol.

* `*Map [<disc spec.>]`: *Map displays a disc's free space map.

* `*MimeMap`
  * *MimeMap [&xxx | .ext | mime/type | Filetype | "Mac_Type"] Returns information on the file type
  * specified.

* `*Mount [:]<disc name>`: *Mount reselects your user root as well as your currently selected directory and library.

* `*NameDisc <disc spec.> <disc name>`: *NameDisc alters a disc's name.

* `*NameDisc <disc spec.> <disc name>`: *NameDisc alters a disc's name.

* `*NoDir`: *NoDir unsets the currently selected directory on the temporary filing system.

* `*NoLib`: *NoLib unsets the library directory on the temporary filing system.

* `*NoURD`: *NoURD unsets the user root directory on the temporary filing system.

* `*Opt`
  * `*Opt [<x> [[,] <y>]]`
  * *Opt controls various filing system actions.
  * Opt 1 <n> Set the filing system message level (for Load/Save/Create):
  *         0       No filing system messages
  *         1       Filename printed
  *         2       Filename,hexadecimal addresses and length printed
  *         3       Filename,datestamp and length printed
  * Opt 4 <n> Set the filing system boot option:
  *         0       No boot action
  *         1       *Load boot file
  *         2       *Run boot file
  *         3       *Exec boot file

* `*Pass [<Old password> [<New password>]]`: *Pass changes your password on the file server.

* `*Play <track> [drive]`: *Play will play from <track> to the end of the disc.

* `*PlayList [drive]`: *PlayList lists the tracks on the compact disc.

* `*PlayMSF <from> <to> [drive]`: *PlayMSF <MM:SS:FF> <MM:SS:FF> plays a given piece of audio.

* `*Prefix`
  * `*Prefix [<directory>]`
  * *Prefix selects a directory as the current directory unique to the currently executing task. *Prefix
  * with no arguments sets the current directory back to the systemwide default (as set with *Dir).

* `*PrepPath <path-variable> <path-component>`: Prepend a given path component onto a path variable, ensuring it only appears once.

* `*PS`
  * `*Configure PS <printer server name>|<station number>`
  * *Configure PS sets the default name or number for the printer server. This name or number will be used if the first open (or save) does not explicitly quote either a name or a number

* `*Ram`: *Ram selects the Ram filing system as the current filing system.

* `*RemPath <path-variable> <path-component>`: Remove a given path component from a path variable.

* `*Remove`
  * `*Remove <filename>`
  * *Remove tries to delete the named file or directory without returning an error if the file does not
  * exist. Directories may only be removed if they contain no files.
  * See also *Wipe, and *Delete.

* `*Rename`
  * `*Rename <object> <new name>`
  * *Rename changes the name of an object. It cannot be used to move objects between filing systems or
  * between discs on the same filing system; *Copy with the D(elete) option must be used instead.

* `*ResourceFS`: *ResourceFS selects the ResourceFS filing system.

* `*Run <filename> [<parameters>]`: *Run loads and executes the named file, passing optional parameters to it.

* `*SafeLogon`
  * `*SafeLogon [[:]<station number>|:<File server name>] <user name> [[:<CR>]<Password>]`
  * *SafeLogon initialises the current (or given) file server for your use, except that if you are already logged on, it does nothing.

* `*Save <filename> <start addr> [<end addr> | +<length>] [<exec addr> [<load addr>]]`: *Save copies the given area of memory to the named file. Length and addresses are in hexadecimal.

* `*Configure ShareBoot <discname>`: *Configure ShareBoot sets a discname for remote booting.

* `*Share`
  * `*Share <pathname> [<discname>] [-protected] [-readonly] [-cdrom] [-subdir] [-noicon] [-auth <key>]`
  * *Share allows a local directory to be seen as a shared disc.

* `*Shares`: *Shares lists the local directories currently being seen as shared discs.

* `*ShareFS`: *ShareFS selects ShareFS as the current filing system.

* `*ShareFSCache [on|off]`: *ShareFSCache enables or disables CD cacheing

* `*ShareFSCacheType [<type> [S][P]]`: *ShareFSCacheType disables primary or secondary cacheing for a specified filetype.

* `*ShareFSIcon <discname>`: *ShareFSIcon adds an icon to the icon bar for a remote shared disc.

* `*ShareFSLogoff <name>`: *ShareFSLogoff logs off from Access+.

* `*ShareFSLogon <name> <key>`: *ShareFSLogon logs on to Access+.

* `*ShareFSWindow [<size>]`: *ShareFSWindow changes the size for the ShareFS transmission window.

* `*Shut`: *Shut closes all open files on all filing systems.

* `*ShutDown`
  * *ShutDown closes all open files on all filing systems, logs off file servers and causes hard discs to be parked.

* `*Spool`
  * `*Spool [<filename>]`
  * *Spool <filename> opens a new file and causes subsequent VDU output to be directed to it, subject to
  * the current *fx 3 status. *Spool with no filename causes the spool file to be closed.

* `*SpoolOn`
  * `*SpoolOn [<filename>]`
  * *SpoolOn <filename> opens an existing file and causes subsequent VDU output to be appended to it,
  * subject to the current *fx 3 status. *SpoolOn with no filename causes the spool file to be closed.

* `*Stamp`
  * `*Stamp <filename>`
  * *Stamp sets the datestamp on a file to the current time and date. If the file is not already datestamped then it is given file type Data (&FFD).

* `*Stop [drive]`: *Stop ceases audio play.

* `*Supported`: *Supported displays the drive types recognised by CDFS.

* `*Type`
  * `*Type [-File] <filename> [-TabExpand]`
  * *Type displays the contents of the file in the configured GSRead format.
  * See also *List and *Print.

* `*URD [<directory>]`: *URD selects a directory as the user root directory (default restores the URD to & or $ as appropriate).

* `*UnShare <discname>`: *UnShare stops sharing a local shared disc.

* `*Unlock [drive]`: *Unlock will unlock the drawer.

* `*Up [<levels>]`: *Up moves the current directory up the directory structure by the specified number of levels.

* `*Verify`
  * `*Verify [<disc spec.>]`
  * *Verify checks the whole disc is readable.
  * The default is the current disc.

* `*WhichDisc`: *WhichDisc displays a number for the current disc.

* `*Wipe`
  * `*Wipe <file spec> [<options>]`
  * *Wipe deletes one or more objects that match the given wildcard specification. Options are taken
  * from the system variable Wipe$Options and those given to the command.
  * Options: (use ~ to force off,eg. ~V)
  * C(onfirm)       Prompt for confirmation of each deletion {on}
  * F(orce)         Force deletion of locked objects {off}
  * R(ecurse)       Delete subdirectories and contents {off}
  * V(erbose)       Print information on each object deleted {on}
  * See also *Delete, and *Remove.
