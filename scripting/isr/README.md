# Instruction set reference

Instructions are **variable length**. Each instruction starts with an **opcode** of one or more bytes, followed by zero or more **arguments**. Some instructions have a *variable number of arguments*, that is, their argument count depends on the value of an earlier argument. Usually this is only used to form **instruction groups**. In the reference implementation, each one- or two-byte opcode corresponds to one function. We may treat instruction groups with two-byte fixed opcodes and a function selection argument as multiple instructions, depending on how much functionality/code is shared.

The following **argument types** are supported:

* `uchar`: An unsigned 8-bit integer. Usually used for selecting one instruction from an instruction group.
* `ushort`: An unsigned 16-bit integer. Usually used to reference [string, label or return address IDs](/scripting/scx_file_format.md).
* `expr`: An [expression](/scripting/expressions.md).

## How to read this

Each instruction definition in this section contains a quick-reference header providing:

* The **instruction name**, in the upper left.
* The **instruction opcode(s)**, in the upper right.
* A short **summary** of the instruction's function, in the middle.
* A list of **arguments**, at the bottom. Note that, as argument count is variable, this only includes the *required* arguments.

**Note:** Instruction names are defined by us and likely do not match official names. While we have looked at *SRC*/*SCS* source code, and our naming scheme is inspired by it, we cannot make every name match (as the instruction set differs between games), and so we do not try.