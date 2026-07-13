# Week 7 of #buildinpublic

This week is all about OOP. I started my lecture on OOP on Monday, and by evening I had covered a lot — the basics of OOP, Classes and Objects, the constructor method, and the `self` keyword in detail. `self` is the thing that took most of my time to understand — what it is, and why we write it with every function in a class. But then my instructor said something very beautiful that made it all make total sense:

There's a golden rule in Python: **"Only objects can access the functions of a class — only an object can call and access a method in a class."**

So the reason we're able to communicate between functions in our class is this `self` keyword, because it's nothing special — it's just the object itself.

Here are my raw thoughts and notes on this specific thing:

```python
''' This rule is saying nothing else can call or access a method/function of
a class other than the object. Then the question is, in the code below, how can we
call menu inside the constructor when only the object has access to the function? '''

class Atm:
    def __init__(self):    
        self.menu()      
        
    def menu(self):       
        print("Hello!")
```

Here in this code, the instructor is calling and using the `menu` function without any error, but the rule was saying only an object can access that. This is the main point — we are not just calling the function like `menu()`, we are using `self.menu()`, and that `self` is actually the object of the class. Here's how:

I will create an object of this class and print its memory address, then we will match it against the memory address of `self`, and they will match.

```python
class Atm:
    def __init__(self):    
        print(id(self))
        # self.menu()      
        
    def menu(self):       
        print("Hello!")
        
obj = Atm()
print(id(obj))
```

The id of both the `self` keyword and the object is the same. This means that `self` is actually the object, and because the object can access any function inside the class, we get no error.

When we create an object and call a method like `obj.methodname()`, and that method has a `self` parameter, what happens is the object itself is given to that method as the `self` parameter. So we aren't passing any argument explicitly — it's going secretly, the object itself — and since the object can access functions, it can call different functions and variables.

Another thing is Python stores an empty dict at the address of the object when it's created, and when the class stores something in it, like:

```python
''' self.name = "Ahmad" is creating an entry in the dict at the memory address of obj,
with key = name and value = Ahmad '''
```

All this behavior in-depth is explained in this DeepSeek chat — it explained it very easily with a proper program flow, step by step: https://chat.deepseek.com/share/ek8h4uo81je7i67y9d

To see all my notes for Monday on this, go here on Notion: https://app.notion.com/p/Class-and-Objects-39c7554446558031bab6e0ff79e84196?source=copy_link