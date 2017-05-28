{% import "/macros.html" as macros %}
{{ macros.instruction_header(
    name = 'Return',
    summary = 'Pop address off call stack and jump there',
    opcodes = ['00 0E']
) }}

Pop script buffer ID and return address ID off call stack, then jump there.

Returning when the call stack is empty is undefined.