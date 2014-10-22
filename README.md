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

...

            else if (PyString_CheckExact(v) &&
                     PyString_CheckExact(w)) {
                x = string_concatenate(v, w, f, next_instr);
                /* string_concatenate consumed the ref to v */
                goto skip_decref_vx;
            }

...

```

The path we're going to follow is shown above. (The initial if statement requires integers, so is skipped). Move into typeobject.c to find PyString_CheckExact(). This returns true if both values are strings, and the values, the frame object, and the following bytecode instruction are sent to string_concatenate() back inside ceval.c.

Once there, the combined lengths of the strings are tested for overflow. Then the reference count to the first value is checked along with the next instruction to reduce uneeded references. In our case, STORE_NAME comes next, so the local variable dictionary is checked for the variable, and if it's included is then cleared. The variable is unecessary because it is currecntly in temporary storage, and the initial value will be eventually replaced by the concatenated string.

Next, our two values are sent to PyString_Concat(), which can be found in stringobject.c.
