# extern usage

if you want to use a global variable in another source file in the same project,
Declare it with the extern keyword where you are using that variable

When using the extern keyword, be careful not to be a general global variable

```
/* test.h */

int g_data1 = 0;
int g_data2 = 0;
int g_data3 = 0;

```

```
/* test.c */
int g_data1; // (1)
extern int g_data2; // (2)
extern int g_data3 = 0; // (3)
extern int g_data4; // (4)
extern int g_data5 = 0; // (5)
```

* #1 throw link error with duplicate declarations.
* #2 work well
* #3 throw link error caused by duplicate declarations because the variable is treated as a general global variable when initialized at the same time as declaration.
* #4 throw link error because there is not a variable referenced.
* #5 is treated as a general global variable. there is no error, there is no meaning that add the extern keyword.
