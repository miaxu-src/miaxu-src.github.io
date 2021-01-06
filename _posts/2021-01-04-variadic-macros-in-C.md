---
category: program
---

If a macro expects to accept a variable number of arguments, one can use the syntax below:

```c
#define DBG_PRINT(...) fprintf(stderr, __VA_ARGS__)
DBG_PRINT("This is my %d-th birthday!", 20);
```

This kind of macro is called <em>variadic</em>. What happens above is that during the pre-processing, the identifier `__VA_ARGS__` in the macro body will be replaced by all the tokens, <em>including any commas</em>, inside the `DBG_PRINT`'s curly brackets. One can also use a name for the variable argument instead of `__VA_ARGS__`. For example,

```c
#define DBG_PRINT(args...) fprintf(stderr, args)
```

Pay attention to the `args...`, there is no comma between `args` and `...`.

If you need both named arguments and variable arguments, then you can use the syntax like this:

```c
#define DBG_PRINT(fp, ...) fprintf(fp, __VA_ARGS__)
```

where the argument `fp` specifies a `FILE*`, and everything else after the comma is treated as a variable number of arguments and will be used to replace `__VA_ARGS__`. In this case, you will need commas to separate named arguments and variable arguments.
