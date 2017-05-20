# Expression encoding

SC3 includes a **binary infix encoding** for mathematical expressions, used in instruction arguments and string escape sequences.

* An **expression** is a **stream of tokens**.
* A **token** is either a **immediate value** or an **operator**.
* The first byte of the token, treated as a signed 8-bit integer, indicates its **type**:
  * `token[0] > 0`: The token is an operator.
  * `token[0] < 0`: The token is an immediate value.
  * `token[0] == 0`: End of expression.
* The last byte of the token indicates **precedence**. Operators with highest precedence must be evaluated first. For immediate values, precedence is meaningless, but the byte is still present. "End of expression" tokens do not conclude with a precedence byte. Operator types do not have
* Operators only consist of the token type and precedence byte, immediate values may have more bytes in between, see below.

Expressions (and all included terms) evaluate to **32-bit signed integers**.

## Immediate values

Immediate values are parsed as such:
```C
uint8_t *token;
int32_t result;

switch (token[0] & 0x60) {
  case 0:  // Token length: 2 bytes
    result = (token[0] & 0x1F);
    if (token[0] & 0x10)  // negative
      result |= 0xFFFFFFE0;
    break;
  case 0x20:  // Token length: 3 bytes
    result = ((token[0] & 0x1F) << 8) + token[1];
    if (token[0] & 0x10)  // negative
      result |= 0xFFFFE000;
    break;
  case 0x40:  // Token length: 4 bytes
    result = ((token[0] & 0x1F) << 16) + (token[2] << 8) + token[1];
    if (token[0] & 0x10)  // negative
      result |= 0xFFE00000;
    break;
  case 0x60:  // Token length: 6(!) bytes
    // token[1]..token[4] is a little-endian signed int32
    result = READ_LITTLE_ENDIAN_INT32(token + 1);
    break;
}
```

**Reminder:** Even immediate values end with a precedence byte.

## Operators

There are several categories of operators:
* **Prefix unary**
* **Postfix unary**
* **Infix binary**
  * **Assignment**
* **Function** (prefix, operand count varies)

An operator's first byte indicates its type.

| **Type** | **# Op. before** | **# Op. after** | **Category**  | **Function**             | **Name**            |
|----------|------------------|-----------------|---------------|--------------------------|---------------------|
| **0x01** | 1                | 1               | Infix binary  | `{left} * {right}`       | Multiply            |
| **0x02** | 1                | 1               | Infix binary  | `{left} / {right}`[^1]   | Divide              |
| **0x03** | 1                | 1               | Infix binary  | `{left} + {right}`       | Add                 |
| **0x04** | 1                | 1               | Infix binary  | `{left} - {right}`       | Subtract            |
| **0x05** | 1                | 1               | Infix binary  | `{left} % {right}`[^1]   | Modulo              |
| **0x06** | 1                | 1               | Infix binary  | `{left} << {right}`      | LeftShift           |
| **0x07** | 1                | 1               | Infix binary  | `{left} >> {right}`      | RightShift          |
| **0x08** | 1                | 1               | Infix binary  | `{left} & {right}`       | BitwiseAnd          |
| **0x09** | 1                | 1               | Infix binary  | `{left} ^ {right}`       | BitwiseXor          |
| **0x0A** | 1                | 1               | Infix binary  | `{left} or {right}`      | BitwiseOr           |
| **0x0B** | 0                | 1               | Prefix unary  | `~{operand}`             | Negation            |
| **0x0C** | 1                | 1               | Infix binary  | `{left} == {right}`      | Equal               |
| **0x0D** | 1                | 1               | Infix binary  | `{left} != {right}`      | NotEqual            |
| **0x0E** | 1                | 1               | Infix binary  | `{left} <= {right}`      | LessThanEqual       |
| **0x0F** | 1                | 1               | Infix binary  | `{left} >= {right}`      | GreaterThanEqual    |
| **0x10** | 1                | 1               | Infix binary  | `{left} < {right}`       | LessThan            |
| **0x11** | 1                | 1               | Infix binary  | `{left} > {right}`       | GreaterThan         |
| **0x14** | 1                | 1               | Assignment    | `{left} = {right}`       | Assign              |
| **0x15** | 1                | 1               | Assignment    | `{left} *= {right}`      | MultiplyAssign      |
| **0x16** | 1                | 1               | Assignment    | `{left} /= {right}`[^1]  | DivideAssign        |
| **0x17** | 1                | 1               | Assignment    | `{left} += {right}`      | AddAssign           |
| **0x18** | 1                | 1               | Assignment    | `{left} -= {right}`      | SubtractAssign      |
| **0x19** | 1                | 1               | Assignment    | `{left} %= {right}`[^1]  | ModuloAssign        |
| **0x1A** | 1                | 1               | Assignment    | `{left} <<= {right}`     | LeftShiftAssign     |
| **0x1B** | 1                | 1               | Assignment    | `{left} >>= {right}`     | RightShiftAssign    |
| **0x1C** | 1                | 1               | Assignment    | `{left} &= {right}`      | BitwiseAndAssign    |
| **0x1D** | 1                | 1               | Assignment    | `{left} or= {right}`     | BitwiseOrAssign[^2] |
| **0x1E** | 1                | 1               | Assignment    | `{left} ^= {right}`      | BitwiseXorAssign    |
| **0x20** | 1                | 0               | Postfix unary | `{operand}++`            | Increment           |
| **0x21** | 1                | 0               | Postfix unary | `{operand}--`            | Decrement           |
| **0x28** | 0                | 1               | Function      | `GlobalVars[{operand}]`  | FuncGlobalVars      |
| **0x29** | 0                | 1               | Function      | `Flags[{operand}]`       | FuncFlags           |
| **0x2A** | 0                | 2               | Function      | `DataAccess({a},{b})`    | FuncDataAccess      |
| **0x2B** | 0                | 1               | Function      | `LabelTable[{a}]`        | FuncLabelTable      |
| **0x2C** | 0                | 2               | Function      | `FarLabelTable({a},{b})` | FuncFarLabelTable   |
| **0x2D** | 0                | 1               | Function      | `ThreadVars[{operand}]`  | FuncThreadVars      |
| **0x2E** | 0                | 2               | Function      | `DMA({a},{b})`           | FuncDMA             |
| **0x2F** | 0                | 0               | Function      | `GetUnk2F()`             |                     |
| **0x30** | 0                | 0               | Function      | `GetUnk30()`             |                     |
| **0x31** | 0                | 0               | Function      | n/a[^3]                  | n/a                 |
| **0x32** | 0                | 0               | Function      | n/a[^3]                  | n/a                 |
| **0x33** | 0                | 1               | Function      | `Random({operand})`      | FuncRandom          |

[^1]: Division by zero yields 0x7FFFFFFF.
[^2]: Attention: The order for assignment is And-Or-Xor, not And-Xor-Or as for the assignmentless operators.
[^3]: There is code to handle these in the reference implementation, but it just skips them.

TODO: figure out return value of assignment, especially whether increment/decrement is pre or post

### Functions

* **GlobalVars**: Accesses a [global variable](/scripting/virtual_machine.md). This is supported as an assignment lvalue.
* **ThreadVars**: Accesses a [thread-local variable](/scripting/virtual_machine.md). This is supported as an assignment lvalue. Note the details of thread-local variable indexing.
* **Flags**: Accesses a [flag](/scripting/virtual_machine.md). This is supported as an assignment lvalue.
* **GetUnk2F**, **GetUnk30**: These functions read state related to input handling or state transitions (likely gamepad/keyboard button press masks).
* **Random**: `return operand * (rand() & 0x7FFFu) >> 15;` in the reference implementation on Win32 (MSVC, `RAND_MAX = 0x7FFF`).
* **DataAccess**: Does not appear to be supported as an assignment lvalue.
  ```C
  if (a >= 0) {
    uint8_t *script = getScriptInBuffer(threadContext.buffer_id);
    uint8_t *array = script + a;
    int32_t result = READ_LITTLE_ENDIAN_INT32(array + (4 * b));
  } else {
    uint8_t *array = (uint8_t *)getSC3ThreadContext(-a); // parameter is thread ID
    int32_t result = READ_LITTLE_ENDIAN_INT32(array + (4 * b));
  }
  ```
* **LabelTable**: Gets the offset of a [label](/scripting/scx_file_format.md) in the calling script. Generally used in conjunction with DataAccess to read embedded arrays.
* **FarLabelTable**: Gets the offset of label `b` in the script loaded in buffer `a`.
* **DMA**: Case `a < 0` supported as assignment lvalue.
  ```C
  if (a >= 0) {
    uint8_t *data = a + 16*b;
    return *(int *)data;
  } else {
    // Same as with data access, except this time, assignment is supported as well.
    uint8_t *array = (uint8_t *)getSC3ThreadContext(-a); // parameter is thread ID
    int32_t result = READ_LITTLE_ENDIAN_INT32(array + (4 * b));
  }
  ```