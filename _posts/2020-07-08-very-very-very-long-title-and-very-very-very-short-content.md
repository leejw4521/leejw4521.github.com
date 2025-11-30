---
layout: post
---

Today, I will be creating a cmd that I was given as homework. CMD is the command prompt on our computer, and we're going to make it in Python.
Since I can't include all of cmd's features, I'm going to try to create just a few, a total of four.
1. cd **foldername**
2. cd..
3. dir
4. cat **filename**


So if I try to write code with these features...
```
import os

while True:
    a=input(os.getcwd()+">")
    if str(a[:2]) == 'cd':
        if str(a[2:]) == '..':
            c = os.getcwd().split('\\') 
            c.pop()
            new_path = '\\'.join(c)
            os.chdir(new_path)
        else:
            os.chdir
            b=str(a[3:])
            os.chdir(os.getcwd()+"\\"+b)
    elif a[:3] == 'cat':
        o = open(os.getcwd()+"\\"+a[4:],'r')
        O=o.read()
        o.close()
        print(O)
    elif a == 'dir':
        os.system('dir')
```

I got code that looks like this. 

But if you only write code like this, it’ll be hard to understand. So let me explain the code.
