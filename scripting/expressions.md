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
    // token[1]..token[4] is a big-endian signed int32
    result = (token[4] << 24) + (token[3] << 16) + (token[2] << 8) + token[1];
    break;
}
```

**Reminder:** Even immediate values end with a precedence byte.

# Operators

There are several categories of operators:
* **Prefix unary**
* **Postfix unary**
* **Infix binary**
  * **Assignment**
* **Function** (prefix, operand count varies)

An operator's first byte indicates its type.

| **Type** | **# op. before** | **# op. after** | **Category**  | **Function**            | **Name**            |
|----------|-----------------------|----------------------|---------------|-------------------------|---------------------|
| **0x01** | 1                     | 1                    | Infix binary  | `{left} * {right}`      | Multiply            |
| **0x02** | 1                     | 1                    | Infix binary  | `{left} / {right}`[^1]  | Divide              |
| **0x03** | 1                     | 1                    | Infix binary  | `{left} + {right}`      | Add                 |
| **0x04** | 1                     | 1                    | Infix binary  | `{left} - {right}`      | Subtract            |
| **0x05** | 1                     | 1                    | Infix binary  | `{left} % {right}`[^1]  | Modulo              |
| **0x06** | 1                     | 1                    | Infix binary  | `{left} << {right}`     | LeftShift           |
| **0x07** | 1                     | 1                    | Infix binary  | `{left} >> {right}`     | RightShift          |
| **0x08** | 1                     | 1                    | Infix binary  | `{left} & {right}`      | BitwiseAnd          |
| **0x09** | 1                     | 1                    | Infix binary  | `{left} ^ {right}`      | BitwiseXor          |
| **0x0A** | 1                     | 1                    | Infix binary  | `{left} or {right}`      | BitwiseOr           |
| **0x0B** | 0                     | 1                    | Prefix unary  | `~{operand}`            | Negation            |
| **0x0C** | 1                     | 1                    | Infix binary  | `{left} == {right}`     | Equal               |
| **0x0D** | 1                     | 1                    | Infix binary  | `{left} != {right}`     | NotEqual            |
| **0x0E** | 1                     | 1                    | Infix binary  | `{left} <= {right}`     | LessThanEqual       |
| **0x0F** | 1                     | 1                    | Infix binary  | `{left} >= {right}`     | GreaterThanEqual    |
| **0x10** | 1                     | 1                    | Infix binary  | `{left} < {right}`      | LessThan            |
| **0x11** | 1                     | 1                    | Infix binary  | `{left} > {right}`      | GreaterThan         |
| **0x14** | 1                     | 1                    | Assignment    | `{left} = {right}`      | Assign              |
| **0x15** | 1                     | 1                    | Assignment    | `{left} *= {right}`     | MultiplyAssign      |
| **0x16** | 1                     | 1                    | Assignment    | `{left} /= {right}`[^1] | DivideAssign        |
| **0x17** | 1                     | 1                    | Assignment    | `{left} += {right}`     | AddAssign           |
| **0x18** | 1                     | 1                    | Assignment    | `{left} -= {right}`     | SubtractAssign      |
| **0x19** | 1                     | 1                    | Assignment    | `{left} %= {right}`[^1] | ModuloAssign        |
| **0x1A** | 1                     | 1                    | Assignment    | `{left} <<= {right}`    | LeftShiftAssign     |
| **0x1B** | 1                     | 1                    | Assignment    | `{left} >>= {right}`    | RightShiftAssign    |
| **0x1C** | 1                     | 1                    | Assignment    | `{left} &= {right}`     | BitwiseAndAssign    |
| **0x1D** | 1                     | 1                    | Assignment    | `{left} or= {right}`     | BitwiseOrAssign[^2] |
| **0x1E** | 1                     | 1                    | Assignment    | `{left} ^= {right}`     | BitwiseXorAssign    |
| **0x20** | 1                     | 0                    | Postfix unary | `{operand}++`           | Increment           |
| **0x21** | 1                     | 0                    | Postfix unary | `{operand}--`           | Decrement           |
| **0x28** | 0                     | 1                    | Function      | `GlobalVars[{operand}]` | FuncGlobalVars      |
| **0x29** | 0                     | 1                    | Function      | `Flags[{operand}]`      | FuncFlags           |
| **0x2A** | 0                     | 2                    | Function      | `DMA1({a},{b})`         | FuncDMA1            |
| **0x2B** | 0                     | 1                    | Function      |                         |                     |
| **0x2C** | 0                     | 2                    | Function      |                         |                     |
| **0x2D** | 0                     | 1                    | Function      | `ThreadVars[{operand}]` | FuncThreadVars      |
| **0x2E** | 0                     | 2                    | Function      |                         |                     |
| **0x2F** | 0                     | 0                    | Function      | `GetUnk2F()`            |                     |
| **0x30** | 0                     | 0                    | Function      | `GetUnk30()`            |                     |
| **0x31** | 0                     | 0                    | Function      | n/a[^3]                 | n/a                 |
| **0x32** | 0                     | 0                    | Function      | n/a[^3]                 | n/a                 |
| **0x33** | 0                     | 1                    | Function      | `Random({operand})`     | FuncRandom          |

[^1]: Division by zero yields 0x7FFFFFFF.
[^2]: Attention: The order for assignment is And-Or-Xor, not And-Xor-Or as for the assignmentless operators.
[^3]: There is code to handle these in the reference implementation, but it just skips them.