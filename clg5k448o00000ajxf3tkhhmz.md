---
title: "Data Race in Python despite GIL?"
seoDescription: "It might even seem silly even to assume that GIL would let you have a condition like data race. But to understand that we'll need to understand GIL."
datePublished: Thu Apr 06 2023 20:14:38 GMT+0000 (Coordinated Universal Time)
cuid: clg5k448o00000ajxf3tkhhmz
slug: data-race-in-python-despite-gil
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680810580963/b66efd22-8c92-4ef9-a59b-fa25ed96778c.png
tags: python, gil

---

# Data race

While the definite definition of **data race** differs with the concurrency model and the language, it's safe to assume that data race is when 2 or more threads try to access the same memory where at least one of them is a write operation.

Seeing the above requirements, it might even seem silly even to assume that GIL (*Global Interpreter Lock*) would let you have a condition like data race. But there are nuances to it and to understand that we'll need to understand what GIL is and how it works.

***Note:*** *Python's semantics aren't tight or specific enough to definitively call something a data race vs. a race condition, so I'll avoid going too deep into its classification.*

# Global Interpreter Lock (GIL)

As the name suggests, GIL locks the *interpreter* to running only 1 thread at a time. This means it limits one thread to execute bytecode at a time or it runs *one bytecode instruction* at a time.

The GIL also guarantees lock is released every `sys.getswitchinterval()` second, at which point a thread switch can take place. This means that for Python code, a thread switch can still take place, but only between byte code instructions.

But what does *"one bytecode instruction at a time"* even mean?

# Bytecode

Bytecode is the low-level representation of the Python code which is platform-independent. In simpler words, it is the *compiled* version of the code you wrote which is run by the Python interpreter. You may be confused and thinking "Isn't Python an interpreted language?", while yes you're right, that is outside the scope of this article so you can read more about it [here](https://stackoverflow.com/questions/6889747/is-python-interpreted-or-compiled-or-both).

Now to understand the statement "one bytecode instruction at a time", we'll need to look at the statement `x=x+1.` While it seems like a simple statement where `x`'s value is incremented by one, but at the bytecode level a simple statement is turned into multiple instructions, such as;

1. Load `x` into the register
    
2. Increment `1` to the value of `x`
    
3. Write the value in the register back to `x`
    

***Note:*** *We'll call this order of operations the "*`load-modify-store` *statement' from now on.*

A simple operation like incrementing a variable has to use multiple instructions. So, if threads are switched in the middle of executing an instruction, it might lead to what we know as a *data race.*

# How does data race occur?

Going back to the `load-store-modify` statement, we discussed that it might lead to a race condition. That is because, at any point in the `load-store-modify` statement, another thread might take over and run its own operation before the 3-part operation is complete, which will affect the correctness of the output of the program. Hence, it may lead to a race condition.

As a simple example, assume that **GIL doesn't exist** in this version of Python's interpreter.

```python
x = 10

def increment():
    global x
    x += 1

t1 = Thread(target=increment)
t2 = Thread(target=increment)

t1.start()
t2.start()

t1.join()
t2.join()
```

Now let's try to dry-run the following code; you need to keep in mind that, in data races, the code will not be deterministic hence we'll dry-run a possibility which might happen.

1. `t1` reads the value of `x` and stores it in a temporary variable `t1_x`,
    
2. `t1` increments the value of `t1_x` by `1`,
    
3. `t2` reads the value of `x` and stores it in a temporary variable `t2_x`,
    
4. `t1` writes the value of `t1_x` back to `x`,
    
5. `t2` increments the value of `t2_x`,
    
6. `t2` writes the value of `t2_x` back to `x`
    

If you don't look at the dry run, you might assume the value of `x` to be `12`, but due to a possible data race condition, the value of `x` will be `11`. The value of `t1_x` will be overwritten as `t2_x` never knew about the `t1_x`'s value being written to `x` and only cares about the initial value of `x`.

You also have to remember that, the following condition isn't possible in current Python (with GIL) due to it having locks on thread switches which lasts `sys.getswitchinterval() seconds` (*5 milliseconds on average*). This means that Python won't switch threads unless an operation takes more than 5 milliseconds and as `x+=1` requires less than that to complete, so threads won't switch context and a data race won't happen.

## Data race example

While the above example can't practically demonstrate data race in Python, the following example can practically demonstrate data race on Python versions *below 3.10*.

```python
from threading import Thread
from time import sleep

counter = 0

def increase():
    global counter
    for _ in range(0, 100000):
        counter = counter + 1

threads = []

for i in range(0, 400):
    threads.append(Thread(target=increase))
for thread in threads:
    thread.start()
for thread in threads:
    thread.join()

print(f'Final counter: {counter}')

# Final counter: 31735072
# Final counter: 32829326
# Final counter: 31496003
```

The above example works because the loop of 100,000 takes more than 5 milliseconds to complete, hence data race can be seen practically.

# Relation between race condition and data race

A race condition is a flaw that occurs when a program's correctness depends on the order and/or timing of the events executed. A race condition is a higher-level error than a data race. Many race conditions can be caused by data races, but this is not necessary. In summary, a data race is a specific thing that a program does, and a race condition is a flaw in the program's behaviour.

It's very important to understand that race condition and data race aren't the same things. They are not a subset of one another. They are also neither necessary nor sufficient conditions for one another. But in this context `load-modify-store` statement causes a data race which might lead to a race condition as well.

To clarify, the `load-modify-store` statement causes a data race which leads to a race condition, and that is the relationship between data race and race condition.

***Note:*** *Because access to data structures is not atomic, race conditions can take place there as well.*

# Conclusion

The data race condition mentioned before is also known as `load-modify-store` data race. A true **data race** condition (same memory location being accessed and modified by our processor at the same time) isn't technically possible, most of the **data race** conditions happen due to cache maintenance problems.

Data race can be classified as a cache maintenance problem as well. If you load a value from a memory location, modify it, and then at some point write it back to the original memory location then you realize the value at the shared location prior to write-back is no longer the value you originally copied then information is potentially lost and that's where things start to go bad.

Data race in Python is definitely possible and GIL doesn't prevent data race as GIL only makes sure that only one thread executes bytecode at a time, which doesn't guarantee higher-level operations will be atomic, hence data race and race conditions can happen in Python.