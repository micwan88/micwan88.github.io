---
title: "Undefined Reference" error when linking c function in c++
categories: c c++ linking
---

## "Undefined Reference" error when linking c function in c++

Please noted that C and C++ are different languages. Their compiler will create different symbol names on object. So C++ linker cannot find the correct symbol which is compiled by C compiler.

If you want to link the C function in C++, you have to define the function with `extern "C"` block. Here is the example:

``` c++

#ifdef __cplusplus
extern "C" {
#endif

	void printhex(const unsigned char *buf, size_t n);

#ifdef __cplusplus
}
#endif

```


References:
- [Linking static C library with C++ code](https://stackoverflow.com/questions/18877437/undefined-reference-to-errors-when-linking-static-c-library-with-c-code)
