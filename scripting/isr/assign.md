{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'Assign',
    summary = 'Evaluate expression for (assignment) side-effects',
    opcodes = ['FE'],
    arguments = [['expr', 'expression']]
) }}

Evaluate **`expression`**, throw away the result, and return. This is not a no-op; expressions may have side effects such as assignments, which this instruction is mostly used for.