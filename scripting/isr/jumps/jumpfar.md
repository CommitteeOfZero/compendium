{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'JumpFar',
    summary = 'Unconditional far jump',
    opcodes = ['00 0C'],
    arguments = [['expr', 'targetScriptBufferId'], ['ushort', 'targetLabelId']]
) }}

Jump to label **`targetLabelId`** in the script currently loaded in buffer **`targetScriptBufferId`**.