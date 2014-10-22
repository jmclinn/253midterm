253midterm
==========

CSC253 Midterm Blog Post (string concatenation) | Jonathan &amp; Yennifer

In this post we're going to discuss how string concatenation using the '+' operator in Python is accomplished inside the interpreter. First, let's create a few lines of Python that use string concatenation 

```python
x = 'Hello'
y = ' World'
z = x + y
print z
```

Here is the bytecode that is produced from those lines

```
  1           0 LOAD_CONST               0 ('Hello')
              3 STORE_NAME               0 (x)

  2           6 LOAD_CONST               1 (' World')
              9 STORE_NAME               1 (y)

  3          12 LOAD_NAME                0 (x)
             15 LOAD_NAME                1 (y)
             18 BINARY_ADD          
             19 STORE_NAME               2 (z)

  4          22 LOAD_NAME                2 (z)
             25 PRINT_ITEM          
             26 PRINT_NEWLINE       
             27 LOAD_CONST               2 (None)
             30 RETURN_VALUE  
```

The concatenation happens on line 4, so let's start there.

After the variables x and y are loaded onto the stack we move into BINARY_ADD in ceval.c

```c
case BINARY_ADD:
            w = POP();
            v = TOP();
            if (PyInt_CheckExact(v) && PyInt_CheckExact(w)) {
                //... skipped, because v and w are not integers
            }
            else if (PyString_CheckExact(v) &&
                     PyString_CheckExact(w)) {
                x = string_concatenate(v, w, f, next_instr);
                /* string_concatenate consumed the ref to v */
                goto skip_decref_vx;
            }
            else {
              //... never reach this point, so no need to slow_add
            }
            Py_DECREF(v);
          skip_decref_vx:
            Py_DECREF(w);
            SET_TOP(x);
            if (x != NULL) continue;
            break;
```
