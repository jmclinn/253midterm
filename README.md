253midterm
==========

CSC253 Midterm Blog Post (string concatenation) | Jonathan &amp; Yennifer

In this post we're going to discuss how string concatenation using the '+' operator in Python is accomplished inside the interpreter. First, let's create a few lines of Python that use string concatenation, and the bytecode it produces. 

```python
x = 'Hello'
y = ' World'
z = x + y
print z
```

```
  4           0 LOAD_CONST               0 ('Hello')
              3 STORE_NAME               0 (x)

  5           6 LOAD_CONST               1 (' World')
              9 STORE_NAME               1 (y)

  6          12 LOAD_NAME                0 (x)
             15 LOAD_NAME                1 (y)
             18 BINARY_ADD          
             19 STORE_NAME               2 (z)

  7          22 LOAD_NAME                2 (z)
             25 PRINT_ITEM          
             26 PRINT_NEWLINE       
             27 LOAD_CONST               2 (None)
             30 RETURN_VALUE  
```
