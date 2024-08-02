---
title: "[Agon Light] Calling Assembly from C in AgDev Environment"
date: 2024-07-30T15:45:08+02:00
draft: false
toc: false
images:
tags: 
  - agon
  - c
---

Combining C and assembly allows you to leverage the strengths of both languages: the high-level structure and portability of C, and the low-level control and efficiency of assembly. This can be particularly useful for performance-critical code, hardware manipulation, or when dealing with legacy code.

## Example code

Here's a step-by-step guide on how to call assembly from C using the AgDev environment. You can find the complete example code [here on GitHub](https://github.com/mikolajmikolajczyk/agdev-examples/tree/master/02-callasmfromc).

## How arguments and return values are passed

[AgDev README](https://github.com/pcawte/AgDev/blob/main/README.md) explains briefly how arguments and return values are handled by AgDev but lets try to figure it out from more practical perspective.

Lets assume we have function `uint16_t add(uint8_t a, uint8_t b)`. We implement this function in assembly.

### Arguments

You may wonder how to get `a` and `b`. This snippet shows how:

```asm
_add:
    push ix             ; Save the current value of index register IX on the stack
    ld   ix, 0          ; Initialize IX to 0
    add  ix, sp         ; Adjust IX to point to the current stack frame

    push bc             ; Save the current value of BC on the stack

    ; Load the first 8-bit integer (function argument) into HL register pair
    ld h, 0             ; Clear H register (upper byte of HL)
    ld l, (ix+6)        ; Load the first argument from stack offset 6 into L register (lower byte of HL)

    ; Load the second 8-bit integer (function argument) into BC register pair
    ld b, 0             ; Clear B register (upper byte of BC)
    ld c, (ix+9)        ; Load the second argument from stack offset 9 into C register (lower byte of BC)
```

But why?

To understand this, we need more data:

1. Arguments are passed on the stack (sp)
2. Return address is also on stack sp + [0,2]
3. In adl mode push and pull instructions always use 3 bytes

> Look at how we don't care about `bc` on the stack! `ix` contains stack pointer from before we pushed `bc`.

Lets create a table with offsets:

| ix offset  | Contents |
|------------|----------|
| ix + [0,2] | Function return address|
| ix + [3,5] | IX value we pushed|
| ix + [6,8] | First argument|
| ix + [9,11]| Second argument|

Now you can see that it is quite easy to predict that `a` is in `(ix + 6)` and `b` is in `(ix + 9)`.

### Return values

[AgDev README](https://github.com/pcawte/AgDev/blob/main/README.md)  explains that return values are always passed through one of registers - depending on the size.

Second part of the example function:

```asm
    add hl, bc          ; Add BC to HL, result is stored in HL

    pop bc              ; Restore the original value of BC from the stack
    pop ix              ; Restore the original value of IX from the stack

    ret                 ; Return to caller, result is in HL
```

In our example we have `uint16_t` as function output - which is 16 bit and based on documentation it will be stored in `hl` register. Luckily `add hl, bc` will store its result in `hl` register and it can automatically become a return value for this function.

## Glue code

Previous section told you how to write assembly compatible with C, but how to actually tell C about this function we wrote in assembly?

### Assembly file

There is a small template you will use for each file you write in assembly for AgDev

```asm
assume adl=1 ; turn on 24 bit addressing mode (Agon uses ez80 and ez80 have this feature)

section .text ; make sure executable instructions you write are in correct memory section
```

You will also need to expose functions you want to be visible in C. You can do so by adding `public` declaration.

```asm
public _add

_add:
  function_body
  ret
```

### C function definition

You also need to define function in C. You can create header for this:

```C
#ifndef MATH_H
#define MATH_H

#include <stdint.h>

extern uint16_t add(uint8_t a, uint8_t b);

#endif
```

That's it! You can call `add` function written in assembly via `C` now! You can look at example provided [here](https://github.com/mikolajmikolajczyk/agdev-examples/tree/master/02-callasmfromc) for visual aid.

#### Happy coding!