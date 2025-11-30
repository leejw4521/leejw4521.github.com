---
layout: post
---

Today, I will be creating a cmd that I was given as homework. CMD is the command prompt on our computer, and we're going to make it in Python.
Since I can't include all of cmd's features, I'm going to try to create just a few, a total of four.

[Order](#here)
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


```
import os
```
First, we need to **import os** to use the os module.


```
while True:
```

Then, since **cmd** needs to run in an infinite loop, set it to *True* to make it loop endlessly.

Okay, we’ve finished the initial part, so now let’s implement the functionality. Let’s start with **cd foldername**.
Looking [here](#here), there are two parts that include cd, so we’ll start by creating the part that distinguishes code with cd from code without it.
```
    if str(a[:2]) == 'cd':
```

This is it—it checks whether the first two characters are cd.



```
        else:
            b=str(a[3:])
            os.chdir(os.getcwd()+"\\"+b)
```
And if it’s not the previous code, it must be **cd foldername**, so we convert the part after *cd* into a string and use the *os* module’s **chdir** and **getcwd** to implement it by combining the current directory with that part.


And next, I’ll explain the cd.. code.
```
        if str(a[2:]) == '..':
            c = os.getcwd().split('\\') 
            c.pop()
            new_path = '\\'.join(c)
            os.chdir(new_path)
```

If the first two characters are '**cd**', we check whether the remaining characters are '**..**'.
Once it matches, it will run.

Set c as the current directory path split by \\

Then, use 'pop' on c to remove the last part.

“Next, insert '\\' between the elements of **c** and store the result in a variable named '**new_path.**'

Finally, move to the directory specified by **new_path**.

And that’s it.

Next is the dir code, which is actually pretty simple.
```
    elif a == 'dir':
        os.system('dir')
```
If it’s not any of the other cases and it’s 'dir', this is the code that runs.
If it’s **dir**, using '*os.system*' will produce the same output as in *cmd*, so that’s all we need.

The last part is cat, and it prints the contents of the file you provide.
```
    elif a[:3] == 'cat':
        o = open(os.getcwd()+"\\"+a[4:],'r')
        O=o.read()
        o.close()
        print(O)
```
If you use **cat**, it reads the contents of the file specified after it.

After reading the file, use 'read' to turn it into a str, then print it, and you’re done.

That’s it!
