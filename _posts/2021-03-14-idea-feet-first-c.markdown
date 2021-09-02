---
layout: post
title: "Project idea: 'Feet First C: The worse guide ever'"
lang: english
---

Lately I've been thinking about writing a C guide of sorts. Perhaps the worse C
guide ever, because I'd like to start from a really crude and unconventional
point: barely compilable C code with plenty of `goto`s.

I can already imagine the reaction of some people when they see something like:

```c
main() {
    int input = read_int();
    int result = 0;

    if (input == 0)
        goto input_0;
    if (input == 1)
        goto input_1;
    
    /* bad input */
    result = -1;
    goto end;

input_1:
    result = 1;
    goto end;
input_2:
    result = 2;
    goto end;

end:
    return result;
}
```

From that the reader would be guided into higher and higher level constructs
like if/else blocks, loops and all:

```c
/* this: */
    int i = 0;
start:
    if (i >= 10)
        goto end;
    /* ... */
next:
    i = i + 1;

/* becomes this: */

for (int i = 0; i < 10; i = i + 1) {
    /* ... */
}
```

The idea is to not only teach C but also teach a general idea of how the
computer runs our code. My hope is that this somehow smoothens the learning
curve as the reader will be presented with very few language constructs
(variables, goto, if, functions). And yes, I intend to present functions before
`while`, `for` etc...