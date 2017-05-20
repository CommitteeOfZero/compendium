{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'Call',
    summary = 'Unconditional near call',
    opcodes = ['00 0B'],
    arguments = [['ushort', 'targetLabelId'], ['ushort', 'returnAddressId']]
) }}

Push current script buffer ID and **`returnAddressId`** onto call stack, then jump to label **`targetLabelId`** in the current script.