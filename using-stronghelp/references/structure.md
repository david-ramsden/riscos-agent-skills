# StrongHelp Manual Structure

## Container Directory
A StrongHelp manual is built from a directory containing files for each page.

- **!Root**: Required. Entry point of the manual.
- **!Configure**: Optional. Custom fonts/styles for the manual.
- **!Pre**: Optional. Utility code (RISC OS machine code) for lookups.
- **!Menu**: Optional. (Main manual only) Iconbar menu.
- **NotFound_{manual}**: Optional. Page shown for missing external manual links.

## File Organization
To manage large manuals, group pages:

- **Flat**: All files in the manual's root directory.
- **Prefix Directory**: Group pages with a common prefix in a directory named with that prefix (e.g., `Wimp_Poll` and `Wimp_PollIdle` in `Wimp_`).
- **First-Letter Range**: Use `[a-z]` or similar directories to automatically group pages by their first letter.
- **!root**: In any directory, a `!root` file is the default page.

## Redirections
Files named `OLDNAME>NEWNAME` redirect lookups for `OLDNAME` to the page `NEWNAME`.

## Naming Constraints
RISC OS filenames have restrictions. StrongHelp additionally reserves `! [ ] \ < > . $ % & ? # :`. Use `!xx` (hex) for special characters in filenames.

## Images
- **#Sprite**: References a sprite from the Wimp pool or a specified sprite file.
- **#SpriteFile**: Defines the path to a sprite file (e.g., `!Sprites`).
- **#Draw**: Places a RISC OS Drawfile.
