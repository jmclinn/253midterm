CSC253 Midterm Blog Post
==========
#### String Concatenation (+)
###### Jonathan &amp; Yennifer


Introduction and Bytecode
=========================

In this post we're going to discuss how string concatenation using the '+' operator in Python is accomplished inside the interpreter. First, let's create a few lines of Python that use string concatenation 

```python
x = 'Hello'
y = ' World'
z = x + y
print z
```

The output of that code is 

```
Hello World
```

And here is the bytecode that is produced from those lines

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

The concatenation happens on line 3, so let's start there.

Walkthrough
===========

After the variables x and y are loaded onto the stack, we move into BINARY_ADD in ceval.c

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

The path we're going to follow is shown above. (The initial if statement requires integers, so it is skipped). We move into typeobject.c to find PyString_CheckExact(). This returns true if both values are strings, and the values, the frame object, and the following bytecode instruction are sent to string_concatenate() inside of ceval.c.

Once there, the combined lengths of the strings are tested for overflow. Then the reference count to the first value is checked along with the next instruction to reduce uneeded references. In our case, STORE_NAME comes next, so the local variable dictionary is checked for the variable, and if it's included, is then cleared. The variable is unecessary because it is currently in temporary storage, and the initial value will eventually be replaced by the concatenated string.

Once the second reference is removed we move into the next if statement.

```c

if (v->ob_refcnt == 1 && !PyString_CHECK_INTERNED(v)) {

```

The reference count has been reduced to one, and our string is larger than the maximum  1 or 2 bytes required for it to be saved in the interned dictionary, which is reserved for small strings.

Next, the actual concatenation happens using the C function memcpy, shown below

```c
        memcpy(PyString_AS_STRING(v) + v_len,
               PyString_AS_STRING(w), w_len);
        return v;
```

The memcpy function takes the first value as the destination. We use our first string as the initial location, adding in an offset of its length, which is where the second string will be appended. The second value is the value to be appended, and finally it's length is included. There is no returned value from memcpy because the new combined string has been placed where the first string value was previously located.

That value is then returned back to BINARY_ADD and is pushed on to the stack where it can be accessed.

Observations
============

1. During the walkthrough you may have noticed there were multiple, and sometimes seemingly redundant, checks about what was being passed, saved, or manipulated. This includes the values being strings, the memory availability, and whether we have the ability to write to the result location. As an ever changing, open-source language, assuming the values being passed from function to function have all the necessary parameters, is an error waiting to happen. This stresses the importance of error checking when developing any program or working to potentially add to or edit the Python interpreter.

2. Py_MEMCPY vs memcpy: For most strings being concatenated memcpy is used. This takes each value and copies one to the end of the other. It has a higher setup cost however, so for cases where the two values are very small (less than 16 bytes) there is an alternate method called Py_MEMCPY that is used. This method simply adds the second value character by character to the end of the first value.

```c
  for (i_ = 0; i < n_; i_++)
    t_[i_] = s_[i_];
```

This inline concatenation is much more efficient for smaller strings, but that efficiency is lost when joining anything more than a few bytes.

Conclusions
===========

The (+) operator takes two strings, creates a new space to join them together, and places them both one at a time. This method is perfectly fine when attempting to join two strings. When multiple strings are involved however, there are more efficient methods, like the .join() function. A sample piece of Python code and it's disassembled bytecode are shown below.

```python
x = 'Hello'
y = ' World'
z = '. Goodmorning!'
a = x + y + z
b = ''.join((x,y,z))
```

```
  4          18 LOAD_NAME                0 (x)
             21 LOAD_NAME                1 (y)
             24 BINARY_ADD          
             25 LOAD_NAME                2 (z)
             28 BINARY_ADD          
             29 STORE_NAME               3 (a)

  5          32 LOAD_CONST               3 ('')
             35 LOAD_ATTR                4 (join)
             38 LOAD_NAME                0 (x)
             41 LOAD_NAME                1 (y)
             44 LOAD_NAME                2 (z)
             47 BUILD_TUPLE              3
             50 CALL_FUNCTION            1
             53 STORE_NAME               5 (b)
```

As you can see, the more strings concatenated, the more times BINARY_ADD needs to be called. The .join() function on the other hand optimizes by building a tuple to handle the multiple concatenations at the same time.

We hope this walkthrough and explanation of string concatenation is helpful!
