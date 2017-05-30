# String encoding

MAGES. engine uses a **custom string encoding** for storing text.

* A string is a **stream of tokens**.
* A token is either a **character** or a **command**.
* The first byte of the token, treated as an unsigned 8-bit integer, indicates its **type**:
  * `token[0] >= 0x80`: The token is a character.
  * `token[0] < 0x80`: The token is a command.


## Characters

Characters are stored as unsigned 16-bit *big-endian* integers, from which 0x8000 must be subtracted to obtain the character index (for the font texture and width table) - i.e. 0x8000 is character 0.

Most MAGES. engine game executables contain a *mostly but not entirely* complete and accurate **UTF-16 conversion table**, likely a debug leftover.

Note that not all characters necessarily correspond to a Unicode codepoint - most character sets contain some *compound characters* made up of several Unicode codepoints.

*In our tools*, we have decided to map these characters to *Private Use Area* codepoints in the conversion tables, accompanied by codepages mapping those codepoints to (Unicode) strings.

A common problem with conversion tables found in game executables is that while the font contains separate segments for full-width and variable-width Western characters, these tables represent both using variable-width Unicode characters. For manually fixed-up tables addressing the issues mentioned above, [refer to SciAdv.Net](https://github.com/CommitteeOfZero/SciAdv.Net/tree/master/src/SciAdvNet.SC3/Data). (TODO: update with more tables)

## Commands

TODO