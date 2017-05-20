{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'JumpTable',
    summary = 'Tabular unconditional near jump',
    opcodes = ['00 08'],
    arguments = [['expr', 'index'], ['ushort', 'jumpTableLabelId']]
) }}

Jump to **`LabelTable[jumpTableLabelId][index]`** in the current script. The jump table is an array of `ushort`.