# SCX file format

All numbers are *32-bit little-endian integers*, unless otherwise noted.

SCX script files begin with the following **header:**

* **Magic number:** `SC3\0`
* **String table offset:** Offset of a string table, which is made up of offsets of embedded strings. The encoding of these strings is non-standard and will be explained in later sections.
* **Return address table offset:** Offset of a return address table, which is made up of addresses relative to the file's beginning (i.e. offsets) of following instructions for each *call*.[^1]
* **Label table:** A table (directly included, not another offset!) of addresses of instructions or local data blocks. These can be used as e.g. jump targets or arrays. The first entry is the script's entry point.

These tables are plain arrays with no length specification or other metadata. SCX files are likely not intended to be read top-to-bottom, only interpreted just-in-time.

However, in our experience, the end of the code always coincides with the start of the string table, the end of the string table with the start of the return address table, the end of the return address table with the start of the first string, and the end of the label table with the first label.

[^1]: Don't blame us, we didn't design this engine, we're just documenting it. In fact, we're just gonna stop commenting on this stuff from now on, because otherwise this guide would be nothing but footnotes.