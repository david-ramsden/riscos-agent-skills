# StrongHelp Page Syntax

## Page Structure
- **First Line**: Always the page title.
- **Commands**: Preceded by `#` at the start of a line or enclosed in `{}` within text. Multiple commands can be separated by `;`.
- **Text**: Standard text. `TAB` characters are used for column alignment.
- **Escaping**: Use `\` for `{`, `}`, `<`, `#`.

## Links
- `<PageName>`: Simple link.
- `<Display Text=>PageName>`: Explicit link.
- `<Display Text=>Manual:PageName>`: External link.
- `<Display Text=>*OSCommand>`: Runs a command.
- `<Display Text=>#URL [url]>`: Opens a URL.
- `<Display Text=>#TYPE [text]>`: Types text into the keyboard buffer.
- `<Display Text=>PageName#tagname>`: Links to a tag.

## Formatting Commands
- **#f**: Restore previous font/style.
- **#fname**: Set current style (e.g., `#Code`, `#Strong`, `#H1`).
- **#fname:Text**: Apply style to `Text`.
- **#fno = fontname xsize [ysize] [Bold no] [Italic no] [Both no]**: Define physical font.
- **#fname = [f{*/_}[no]] [RGB r,g,b] [Align Left|Right|Centre]**: Define style.
- **#RGB r,g,b**: Set text color (0-255).
- **#Indent [[+]n]**: Indent by `n` OS units. Empty `Indent` restores level.
- **#Line [width%]**: Draw a horizontal line.
- **#Align [Left|Right|Centre]**: Text alignment.
- **#Wrap [On|Off|Nojoin]**: Wrapping control.
- **#Tab [format]**: Column alignment with optional styles per column.
- **#Table [Columns|Lines] n**: Organize text into columns. End with `#Endtable`.

## Structure and Navigation
- **#Parent PageName**: Mandatory for navigation.
- **#Subpage**: Defines a sub-page within the same file.
- **#TAG tagname**: Defines a linkable tag.

## Preprocessing
- **{$varname}**: System variable expansion.
- **#Include filename**: Include contents of another file.

## Standard Styles
- `Std`: Body text.
- `Link`: Link text.
- `Strong`: Bold.
- `Emphasis`: Italic.
- `Underline`: Underlined.
- `Code`: Monospaced.
- `H1`-`H6`: Headings.
- `Cite`: Citations.
