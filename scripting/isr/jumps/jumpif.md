{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'JumpIf',
    summary = 'Conditional near jump',
    opcodes = ['00 0A'],
    arguments = [['uchar', 'conditionExpectedTrue'], ['expr', 'condition'], ['ushort', 'targetLabelId']]
) }}

Jump to label **`targetLabelId`** in the current script if value of **`condition`** has same truthiness as **`conditionExpectedTrue`** (i.e. `(condition == 0 && conditionExpectedTrue == 0) || (condition != 0 && conditionExpectedTrue != 0)`).