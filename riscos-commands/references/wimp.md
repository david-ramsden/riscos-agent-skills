# RISC OS Wimp Operations

* `*AddApp <application>`: *AddApp creates a link from the Resources icon to an application.

* `*AddTinyDir [<pathname>]`: *AddTinyDir adds a file, application or directory to the desktop icon bar.

* `*BackDrop`
  * `*BackDrop [-Colour <&BBGGRR00>] [-TopColour <&BBGGRR00> [-hsv]] [-TextColour <&BBGGRR00>] [-Centre | -Tile | -Scale | -Remove] [<pathname>]`
  * *BackDrop puts a sprite or JPEG on the desktop background, or selects the colour of the background.
  * Use BackDrop -Remove to clear the background.
  * -Colour         Selects the colour for the backdrop (or bottom colour if
  *                 top colour is given)
  * -TopColour      Selects the colour for the top of the backdrop to fade to.
  * -HSV            Perform fade within the HSV colour space (multi-coloured)
  * -TextColour     Selects the colour for text to be displayed in.
  * -Centre         Centres a sprite or JPEG.
  * -Tile           Tiles a sprite or JPEG.
  * -Scale          Scales a sprite or JPEG to the size of the background.
  * -Remove         Remove any sprite or JPEG.

* `*Desktop`
  * `*Desktop [<*command> | -File <filename>]`
  * *Desktop starts up any dormant Wimp modules, and also passes an optional *command or file of
  * *commands to Wimp_StartTask.

* `*Desktop_ADFSFiler`
  * The ADFSFiler provides the ADFS icons on the icon bar, and uses the Filer to display ADFS
  * directories.
  * Do not use *Desktop_ADFSFiler, use *Desktop instead.

* `*Desktop_CDFSFiler`
  * The CDFSFiler provides the CDFS icons on the icon bar, and uses the Filer to display CDFS
  * directories.
  * Do not use *Desktop_CDFSFiler, use *Desktop instead.

* `*Desktop_ConfirmShutdown ON|OFF`: Allows configuration of the shutdown warning.

* `*Desktop_DisplayManager`
  * The Display Manager controls aspects of the screen display.
  * Do not use *Desktop_DisplayManager, use *Desktop instead.

* `*Desktop_Filer`
  * The Filer is the Desktop file management tool.
  * Do not use *Desktop_Filer, use *Desktop instead.

* `*Desktop_Free`
  * The Free utility displays free space information for the desktop filers
  * Do not use *Desktop_Free use *Desktop instead.

* `*Desktop_HostFSFiler`
  * The HostFSFiler provides the HostFS icons on the icon bar, and uses the Filer to display HostFS directories.
  * Do not use *Desktop_HostFSFiler, use *Desktop instead.

* `*Desktop_NetFiler`
  * The NetFiler provides the Net icons on the icon bar, and uses the Filer to display Net directories.
  * Do not use *Desktop_NetFiler, use *Desktop instead.

* `*Desktop_Pinboard`
  * The Pinboard utility allows directories and files to appear on the desktop background.
  * It also provides the TinyDirs utility which allows directories and files to appear on the icon bar.
  * Do not use *Desktop_Pinboard, use *Desktop instead.

* `*Desktop_RAMFSFiler`
  * The RAMFSFiler provides the RAMFS icon on the icon bar, and uses the Filer to display RAMFS directories.
  * Do not use *Desktop_RAMFSFiler, use *Desktop instead.

* `*Desktop_ResourceFiler`
  * The Resource Filer provides the Apps icon on the icon bar, and uses the Filer to display Resource directories.
  * Do not use *Desktop_ResourceFiler, use *Desktop instead.

* `*Desktop_ShareFSFiler`
  * The ShareFSFiler provides the ShareFS icon on the icon bar, and uses the Filer to display ShareFS directories.
  * Do not use *Desktop_ShareFSFiler, use *Desktop instead.

* `*Desktop_TaskManager`
  * The Task Manager module provides task management under the desktop.
  * Do not use *Desktop_TaskManager, use *Desktop instead.

* `*Filer_Action`: The Filer_Action module runs the background Filer operations

* `*Filer_Boot`
  * `*Filer_Boot [-d] <application>`
  * *Filer_Boot boots the application specified. If -d is specified then booting of the application is
  * delayed until the desktop is active.

* `*Filer_CloseDir <full dirname>`: *Filer_CloseDir may be used in the Desktop to close a directory viewer.

* `*Filer_Layout`
  * `*Filer_Layout [-LargeIcons | -SmallIcons | -FullInfo | -Thumbnails] [-SortByName | -SortByType | -SortBySize | -SortByDate]`
  * *Filer_Layout sets the default layout for filer viewers.

* `*Filer_OpenDir`
  * `*Filer_OpenDir <full dirname> [<x> <y> [<width> <height>]] [<switches>]`
  * *Filer_OpenDir may be used in the Desktop to open a directory viewer.
  * Options are taken from the Filer's template and the user's selections in the Filer's menu.
  * Switches:
  * -SmallIcons     Display small icons
  * -LargeIcons     Display large icons
  * -FullInfo       Display full information
  * -SortbyName     Display sorted by name
  * -SortbyType     Display sorted by type
  * -SortbyDate     Display sorted by date
  * -SortbySize     Display sorted by size
  * Field names:
  * -DIRectory      The full dirname
  * -X0, -topleftx  The x-coordinate of the top left of directory viewer
  * -Y1, -toplefty  The y-coordinate of the top left of directory viewer
  * -Width          The width of the viewer
  * -Height         The height of the viewer
  * All numeric quantities are in OS units. X0 and Y1 may not be supplied independently. Width and
  * Height may not be supplied independantly and may be ignored if X0 and Y1 are not supplied.

* `*Filer_Options`
  * `*Filer_Options <switches>`
  * *Filer_Options sets the default options for Filer operations, and other configuration switches.
  * Switches:
  * -ConfirmAll     Prompt for confirmation of all operations
  * -ConfirmDeletes Prompt for confirmation of deletes only
  * -Verbose        Provide an information window during operations
  * -Force          Force overwrites of existing objects
  * -Newer          Copy only if the source is more recent than the destination
  * -Faster         Perform operation faster
  * -Query          List the currently selected options
  * -TransientSelections    Enable the transient selection model
  * -LowerCase      Ensure that filenames that are solely capitals are lower cased
  * -SelectRenames  Allow files to be renamed by selecting their text

* `*Filer_Run [-noshift] [-type <filetype>] <file>|<application>`: *Filer_Run is equivalent of double clicking on an object.

* `*Filer_Thumbnails`
  * `*Filer_Thumbnails [-Width <OS units>] [-Height <OS units>] [-Depth <depth type>] [-MaximumFileSize <kilobytes>]`
  * *Filer_Thumbnails sets the width and height of icons in the thumbnail display. The 'Depth' can be
  * one of :
  * 0       Use the current depth
  * 1       Use 16-bit colour
  * 2       Use 24-bit colour
  * The maximum file size declares the largest file size which will be loaded during thumbnailing. Files
  * larger than this will never be thumbnailed.

* `*Filer_Truncation`
  * `*Filer_Truncation [-LargeIconDisplay <OS units>] [-SmallIconDisplay <OS units>] [-FullInfoDisplay <OS units>] [-ThumbnailsIconDisplay <OS units>`
  * *Filer_Truncation sets the width that long filenames are truncated to.

* `*Filters`: *Filters displays all Wimp filters currently active.

* `*IconSprites`
  * `*IconSprites [-priorityclear] [[-priority|-file] <filename>]`
  * *IconSprites loads a sprite file into the Wimp's common sprite pool. If -priority is specified, the
  * sprite file will be loaded into a high priority sprite pool which over-rides all normal *IconSprites
  * commands. The priority sprite pool can be used to ensure that the users chosen 'style' of graphics
  * are not overridden by applications. The -priorityclear switch can be used to remove all sprites from
  * the priority pool, as might be performed when changing 'styles'.

* `*Pin <pathname> <x> <y>`: *Pin adds a file, application or directory to the desktop pinboard.

* `*Pinboard`: *Pinboard clears the icons from the pinboard.

* `*PinboardOptions`
  * `*PinboardOptions <switches>`
  * *PinboardOptions allows you to set the options used by Pinboard.
  * Switches:
  * -Grid                   Turn on grid lock
  *                         If switch not specified, grid lock is off.
  * -IconiseTo<Location>    Specify location windows are iconised and tidied to
  *                         (IconBar, TopLeft, BottomLeft, TopRight, BottomRight)
  *                         Without switch, icons are iconised at pointer.
  * -IconiseStackVertical   Specify vertical stacking of iconised window icons
  *                         If switch not specified, icons are stacked horizontally.
  * -TidyTo<Location>       Specify location file icons are tidied to
  *                         (TopLeft, BottomLeft, TopRight, BottomRight)
  *                         Without switch, icons are tidied to top left.
  * -TidyStackVertical      Specify vertical stacking of file icons
  *                         If switch not specified, icons are stacked horizontally.

* `*Ping`
  * `*Ping [-Rdfnqrv] [-c <count>] [-i <wait>] [-l <preload>]`
  * Command is: Ping            1.06 (11 Dec 2004)
  * 
  * *Ping sends ICMP echo requests to a host to calculate a round-trip time. This is usually used to 
  * identify network problems and conjestion.
  *               [-p <pattern>] [-s <packetsize>] <host>
  * 
  * Switches:
  *   -R  Set 'Record Route' option
  *   -d  Set 'Debug' option
  *   -f  Flood ping (handle with care)
  *   -n  Display numeric addresses
  *   -q  Quiet (just display results)
  *   -r  Don't route
  *   -v  Verbose (display additional information)
  *   -c  Specify number of packets to send
  *   -i  Specify interval between packets
  *   -l  Specify number of initial packets to preload network with
  *   -s  Specify packet size
  *   -p  Specify pattern for data in packet

* `*ShellCLI`: ShellCLI - used by a Wimp Program to create a CLI shell

* `*ShellCLI_Task`
  * `*ShellCLI_Task XXXXXXXX XXXXXXXX`
  * *ShellCLI_Task runs an application in a window. The first argument is an 8 digit hex. number giving
  * the task handle of the parent task. The second argument is an 8 digit hex. number giving a handle
  * which may be used by the parent task to identify the task.
  * This command is intended for use only within applications.

* `*ShellCLI_TaskQuit`: *ShellCLI_TaskQuit quits the current task window

* `*TaskWindow`
  * `*TaskWindow [<command>] [[-wimpslot] <n>K] [[-name] <taskname>] [-ctrl] [-display] [-quit]`
  * The *TaskWindow command allows a background task to be started, which will obtain a task window if
  * it needs to do any screen I/O.
  *         <command> is the command to be executed
  *         -wimpslot sets the memory to be allocated
  *         -name sets the task name
  *         -ctrl allows control characters through
  *         -display opens the task window immediately, rather than waiting for a character to be printed
  *         -quit makes the task quit after the command even if the task window has been opened
  * Note that fields must be in " " if they comprise more than one word

* `*ToolSprites`
  * `*ToolSprites [<filename>]`
  * *ToolSprites loads a sprite file to use as window borders. If no filename is given it restores the default tools.

* `*WimpSlot`
  * `*WimpSlot [[-min] <size>[K|M|G]] [[-max] <size>[K|M|G]] [[-next] <size>[K|M|G]]`
  * Change the size of application space, or the amount of application space allocated to the next task to run.

* `*WimpAutoFrontDelay`
  * `*Configure WimpAutoFrontDelay <delay>`
  * *Configure WimpAutoFrontDelay sets the time in 1/10 second units that the pointer has to stay at the
  * bottom of the screen before the icon bar is brought to the front.

* `*WimpAutoFrontIconBar`
  * `*Configure WimpAutoFrontIconBar On|Off`
  * *Configure WimpAutoFrontIconBar sets whether the icon bar is brought to the front when the pointer
  * is held at the bottom of the screen.

* `*WimpAutoMenuDelay`
  * `*Configure WimpAutoMenuDelay <delay>`
  * *Configure WimpAutoMenuDelay sets the time in 1/10 second units that the pointer has to stay over a
  * non-leaf menu entry before the submenu is opened automatically if WimpFlags bit 7 is set.

* `*WimpAutoScrollDelay`
  * `*Configure WimpAutoScrollDelay <delay>`
  * *Configure WimpAutoScrollDelay sets the time in 1/10 second units that the pointer has to stay over
  * the edge of a window before it starts scrolling. This only applies in certain circumstances.

* `*WimpButtonType`
  * `*Configure WimpButtonType Click|Release`
  * *Configure WimpButtonType sets whether the back, close, iconise and toggle-size icons act instantly
  * when you click on them, or when you release the mouse button afterwards.

* `*WimpClickSubmenu`
  * `*Configure WimpClickSubmenu On|Off`
  * *Configure WimpClickSubmenu sets whether clicking on a menu item that has an attached submenu will
  * cause the attached submenu to be opened.

* `*WimpDoubleClickDelay`
  * `*Configure WimpDoubleClickDelay <delay>`
  * *Configure WimpDoubleClickDelay sets the time in 1/10 second units after a single click during which
  * a double click is accepted.

* `*WimpDoubleClickMove`
  * `*Configure WimpDoubleClickMove <distance>`
  * *Configure WimpDoubleClickMove sets the distance in OS units that the pointer has to move after a
  * single click for a double click to be cancelled.

* `*WimpDragDelay`
  * `*Configure WimpDragDelay <delay>`
  * *Configure WimpDragDelay sets the delay in 1/10 second units after a single click after which a drag
  * is started.

* `*WimpDragMove`
  * `*Configure WimpDragMove <distance>`
  * *Configure WimpDragMove sets the distance in OS units that the pointer has to move after a single
  * click for a drag to be started.

* `*WimpFlags`
  * `*Configure WimpFlags <number>`
  * *Configure WimpFlags sets the default actions when dragging windows, as follows:
  * bit 0 set: continuous window movement
  * bit 1 set: continuous window resizing
  * bit 2 set: continuous horizontal scroll
  * bit 3 set: continuous vertical scroll
  * bit 4 set: don't beep when error box appears
  * bit 5 set: allow windows to go partly off screen
  * bit 6 set: allow windows to go partly off screen in all directions
  * bit 7 set: open submenus automatically

* `*WimpFont`
  * `*Configure WimpFont <font number>`
  * *Configure WimpFont sets the font to be used within the desktop for icons and menus. 0 means use `Wimp$*` and 1 means use system font. 2-15 refer to a ROM font.

* `*WimpIconBarAcceleration`
  * `*Configure WimpIconBarAcceleration <rate>`
  * *Configure WimpIconBarAcceleration sets the acceleration rate of an icon bar scroll in OS units per
  * second per second.

* `*Configure WimpIconBarSpeed <speed>`: *Configure WimpIconBarSpeed sets the initial scrolling speed of the icon bar in OS units per second.

* `*WimpIconBorder`
  * `*WimpIconBorder [-arcsize small | medium | normal | large] [-buttonround] [-buttonfade] [-buttonrim] [-button3dbg] [-sunkround] [-sunkfade] [-sunkrim] [-sunk3dbg] [-writeableround] [-groupround] [-thin] [-special [-specialbg] [-specialactionfg &RRGGBB] [-specialdefaultfg &RRGGBB] [-specialaction &RRGGBB] [-specialdefault &RRGGBB]] [-colourgroup &RRGGBB] [-groupoutline] [-sprites] `
  * *WimpIconBorder is used to configure the icon borders which will be used for buttons and borders.

* `*Configure WimpIconiseButton On|Off`: *Configure WimpIconiseButton sets whether an iconise button is added to top-level windows.

* `*WimpKillSprite <spritename>`: Remove a sprite from the Wimp sprite pool.

* `*WimpMenuDragDelay`
  * `*Configure WimpMenuDragDelay <delay>`
  * *Configure WimpMenuDragDelay sets the time in 1/10 second units for which menu activity is disabled
  * after a menu has been opened automatically. This enables the pointer to move over other menu entries
  * without cancelling the submenu.

* `*WimpMode <number> | <specifier string>`: Change the current Wimp screen mode without affecting the configured value.

* `*WimpPalette <filename>`: Used to activate a Wimp palette file.

* `*WimpScroll`
  * `*WimpScroll [-Off] [-Type Pointer | Focus | FocusOrPointer | FavourHigher] [-LineScroll] [-Speed <value>]`
  * *WimpScroll controls the scrolling of windows using the alternate positioning device (usually 'scroll wheels').

* `*WimpSpritePrecedence`
  * `*Configure WimpSpritePrecedence RAM|ROM`
  * *Configure WimpSpritePrecedence sets whether ROM sprites have priority over RAM sprites or vice
  * versa.

* `*WimpTask <*command>`: Start up a new task (from within a task).

* `*WimpTextSelection`
  * `*WimpTextSelection [-inserteffect <effect>] [-deleteeffect <effect>] [-moveeffect <effect>] [-fg &RRGGBB] [-bg &RRGGBB] [-autoselect] [-disable]`
  * *WimpTextSelection changes the manner in which the selection of text in writeable icons functions.
  * The selection colours can be modified using the -bg and -fg options. The effect of inserting or
  * deleting text can be modified using the -inserteffect and -deleteeffect options. If the colours are
  * not specified, the icons own colours will be inverted. If the selection controls interfere with
  * commonly used applications, the selection handling can be disabled through this command. Effects
  * which can be used when text is inserted or deleted are :
  * 
  * none    leave selection in place, insert within or around the selection
  * delete  delete the selected text
  * clear   clear the selection, leaving text in place
  * cut     copy the selected text to the clipboard, then delete it from the icon

* `*WimpToolOrder`
  * `*WimpToolOrder <location>=<toollist> <location>=<toollist> <location>=<toollist>`
  * *WimpToolOrder changes the way in which windows tools are rendered around windows. The settings are
  * described by the positions that the tools are located, listing the tools from left (or top) to right
  * (or bottom). The format that should be used to describe tool locations is a space separated list of
  * components :
  *         <location>=<toollist>
  * 
  * where <location> may be 'T', 'B', 'L' or 'R' for the 'Top', 'Bottom', 'Left' or 'Right' edges of the
  * window. The tools which may be listed in a single string are :
  * b       Back icon.
  * c       Close icon
  * t       Title text
  * i       Iconise icon
  * s       Toggle size icon
  * v       Vertical scrollbar
  * h       Horizontal scrollbar
  * r       Resize icon
  * 
  * Certain combinations are non-sensical, and extreme care should be observed when selecting settings.
  * Typical examples might be :
  * T=bctis R=tvr B=hr
  *         Standard RISC OS 4 style
  * B=bctir R=hvr T=hs
  *         Upside down RISC OS 4 style
  * T=btisc R=tvr B=hr
  *         Typical of another well known OS
  * 
  * In particular, it should be noted that the toggle size icon and the resize icon may not function as
  * desired if they move away from their more common locations.

* `*WimpVisualFlags`
  * `*WimpVisualFlags <options>`
  * *WimpVisualFlags changes some aspects of the visual appearance of the desktop.
  * -3DWindowBorders
  *         Give all menus and dialogue boxes a 3D border.
  * -TexturedMenus
  *         Give all menus a textured background.
  * -UseAlternateMenuBg
  *         Use a different background tile for menus.
  * -RemoveIconBoxes
  *         Remove the filled box from behind the text in text+sprite icons.
  * -NoIconBoxesInTransWindows
  *         Remove the filled box from icons on windows similar to the pinboard.
  * -Fully3DIconbar
  *         Make the iconbar have a full 3D border.
  * -NoFontBlending
  *         Don't use font blending in icons.
  * -WindowBorderFaceColour <&RRGGBB>
  *         Set the colour of the top left portion of the window border.
  * -WindowBorderOppColour <&RRGGBB>
  *         Set the colour of the bottom right portion of the window border.
  * -MenuBorderFaceColour <&RRGGBB>
  *         Set the colour of the top left portion of the menu border.
  * -MenuBorderOppColour <&RRGGBB>
  *         Set the colour of the bottom right portion of the menu border.
  * -SpecialHighlightBackColour <&RRGGBB>
  *         Set the background colour of the 'normal' selected icons.
  * -SpecialHighlightForeColour <&RRGGBB>
  *         Set the foreground colour of the 'normal' selected icons (must set background first).
  * -CaretColour <&RRGGBB>
  *         Set the default colour to be used for the caret.

* `*WimpWriteDir`
  * `*WimpWriteDir 0|1`
  * Change the direction for writable icons.
  * 0 - Same direction as the configured territory.
  * 1 - Opposite direction to the configured territory.

* `*Window_Gadgets`
  * Display a list of the gadgets registered with the Window module. The list is sorted by gadget type
  * (first number) and gadget size (second number).
