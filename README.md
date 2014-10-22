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
python -m dis midterm/add.py
```
