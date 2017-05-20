{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'CallFar',
    summary = 'Unconditional far call',
    opcodes = ['00 0D'],
    arguments = [['expr', 'targetScriptBufferId'], ['ushort', 'targetLabelId'], ['ushort', 'returnAddressId']]
) }}

Push current script buffer ID and **`returnAddressId`** onto call stack, then jump to label **`targetLabelId`** in the script currently loaded in buffer **`targetScriptBufferId`**.