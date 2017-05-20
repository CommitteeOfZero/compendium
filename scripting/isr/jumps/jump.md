{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'Jump',
    summary = 'Unconditional near jump',
    opcodes = ['00 07'],
    arguments = [['ushort', 'targetLabelId']]
) }}

Jump to label **`targetLabelId`** in the current script.