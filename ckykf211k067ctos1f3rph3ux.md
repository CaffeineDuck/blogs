---
title: "Handling the fear of Asynchronous file handling in Python"
seoDescription: "Asynchronous file handling in python with aiofiles python module and asyncio as the async runtime."
datePublished: Tue Jan 18 2022 17:54:53 GMT+0000 (Coordinated Universal Time)
cuid: ckykf211k067ctos1f3rph3ux
slug: handling-the-fear-of-asynchronous-file-handling-in-python
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1642528467852/lI72XpysF.png
tags: python, async

---

# Introduction

Let's face it, the clock speed of our CPU doesn't really affect the performance of simple programs highly in the modern age. Things like `concurrency`, `multiprocessing` or `multithreading` are the leading factors affecting the performance of those programs. Luckily with most of the current programming languages, it's really easy to implement those features. In this article, we're going to focus more on `asynchronous`(concurrency) programming, and more specifically the overlooked part in asynchronous programming; `file handling`. 

[Click here](https://medium.com/analytics-vidhya/multithreading-and-multiprocessing-in-python-1f773d1d160d) to learn more about `multiprocessing` or `multithreading`

# What is Asynchronous Programming?

Asynchronous code just means that the language has a way to tell the computer/program that at some point in the code, it will have to wait for *something else* to finish somewhere else. Let's say that *something else* is called "slow-file". So, during that time, the computer can go and do some other work, while "slow-file" finishes.

In async, every function has a protocol of saying "I will take some time doing this please continue running other things'"

We can think of asynchronous programming in terms of hotels and waiters. Waiters come to you and give you a menu then they go away. They periodically come to you to check if you have completed picking the food, but while you pick the food, they go to other tables and take other orders. 

In the same way, your async program also doesn't stop doing other tasks while waiting for a function to complete, it jumps back and forth from one task to another while waiting for the tasks to complete. 


## Why would you want to use async code?

One of the main uses of asynchronous codes is while I/O (hence the name `asycnio`). It takes around **126.95 seconds** to retrieve data 500 times from an API in sync code. Whereas it takes **0.4880 seconds** to retrieve data 500 times from an API in async code (For the same API). It's even faster and more efficient than `threading` and `multiprocessing` in the case of I/O because spawning `threads` and `processes` takes more time and resources than adding another coroutine in the `event loop`.

[Click here](https://towardsdatascience.com/why-you-should-use-async-in-python-6ab53740077e) for further reading. 

## Async Code Examples

### Awaiting a function
```py
await some_function()
```

> **NOTE:** You can only await inside of async functions

### Creating an Async function
```py
async def some_function():
    ...
```

### Running async function in root python
```py
import asyncio

async def some_function():
    ...

if __name__ == '__main__':
    asyncio.run(some_function())
```

### Key Points

- Even though `multiprocessing`, `multithreading` and `concurrency` may seem similar, they are different. You can check out the link below to learn more.
- async functions are also commonly referred to as `coroutines`.
- asynchronous programming is also known as `concurrency`.

[Click here](https://fastapi.tiangolo.com/async/#technical-details) for further reading.

Credits: [Nirjal Paudel](https://www.facebook.com/n1rjal), [Mahesh C. Regmi](https://www.facebook.com/regmicmahesh)


# What is File Handling?

Simply put, file handling means `reading` and `writing` in the file through code (or specifically `python` in our case.) Through python, we can operate on the files. The concept of file handling is slightly different over various programming languages. In python's case, we can handle file handling with the help of the built-in function `open()`. 

We can open files in python in various modes such as:
- Read (r)
- Write (w)
- Append (a)

> **NOTE:** There are a lot of other modes, you can learn about them [here](https://tutorial.eyehunts.com/python/python-file-modes-open-write-append-r-r-w-w-x-etc/).

## File Handling Code Examples

Assume we have an empty file 'random.txt':

### Writing Content
```py
# Opening the file 'random.txt' in write mode
with open('random.txt', 'w') as f:
    # Writing the contents of the file
    for number in range(1, 4):
        f.write(f"hello {number}\n")
```

If you check 'random.txt' you'll see these contents there:
```
hello 1
hello 2
hello 3
```

### Reading content
```py
# Opening the file 'random.txt' in read mode
with open('random.txt', 'r') as file:
    # Reading the contents of the file
    content = file.read()
# Printing the contents
print(content)
```

OUTPUT:
```
hello 1
hello 2
hello 3
```

[Click here](https://www.geeksforgeeks.org/file-handling-python/) for further reading.

You may be confused by the use of `with` while opening the file. `with` shows the usage of `context manager` in python. 


## What are Context Managers?

Usage of resources like file operations or database connections is very common in programming. But those resources are limited in supply. If we don't release these resources after using them, it may lead to resource leaking. This is the perfect use case for context managers, which automatically set up and teardown resources.

[Click here](https://www.geeksforgeeks.org/context-manager-in-python/) for further reading.


# Asynchronous file handing in python

The default file handling in python is `synchronous` hence blocking. It stops the execution of the whole program while doing any operations related to the file. The bigger the file, the more resource and time intensive it is, which may severely degrade the performance of the whole program. This is where `asynchronous` file handling comes into play. 

We're going to use `asyncio` as the asynchronous runtime provider in python, and `aiofiles` python module for asynchronous file handling. 

> **NOTE:** You need to install `aiofiles` using `pip install aiofiles` for running the code examples below.

## Using aiofiles

### Asynchronously reading file contents
```py
# Importing required modules
import asyncio
import aiofiles

# Defining the entrypoint of our asynchronous program
async def main():
    # Opening random.txt with aiofiles in read mode
    async with aiofiles.open('random.txt') as f:
        # Asynchronously reading the files contents
        contents = await f.read()
    # Printing the file contents
    print(contents)

# Explanation: https://www.freecodecamp.org/news/if-name-main-python-example/
if __name__ == '__main__':
    # Running the async function main() using asyncio
    asyncio.run(main())
```

> **NOTE:** You might have noticed that I haven't passed `r` as the mode. That is because `r` (read-mode) is the default mode, so we don't have to explicitly pass it every time.

It's going to have the same output as the file handling example [above](#heading-file-handling-example)

### Asynchronously iterating through lines of the file
```py
# Opening 'random.txt' in read mode
async with aiofiles.open('random.txt') as f:
    # Asynchronously iterating through the lines
    async for line in f:
        # Printing each line in the file with seperator of ', '
        print(f"New Line: '{line}'", sep=', ')
```
> **NOTE:** I have removed the boilerplate code of imports, defining the main function and running of the function, if you want to run the code examples, you'll have to add that yourself.

OUTPUT:
```
New Line: 'hello 1', New Line: 'hello 2', New Line: 'hello 3'
```

### Asynchronously writing to multiple files
```py
# Importing modules
import asyncio
import aiofiles

# Creating a function that writes to files asynchronously
async def write_to_file(filename: str, content: str) -> None:
    async with aiofiles.open(filename, "w+") as f:
        await f.write(content)

# Creating an entry point for our async program
async def main():
    # Creating an empty list of tasks
    tasks = []
    # Running a loop for 100 times
    for number in range(100):
        # Appending a coroutine of write_to_file function
        # to the list of tasks
        tasks.append(
            asyncio.ensure_future(
                write_to_file(f"{number}.txt", f"File Number: {number}")
            )
        )
    # Running all the tasks using asyncio.gather
    await asyncio.gather(*tasks)

# Running the main() function
if __name__ == "__main__":
    asyncio.run(main())
```

#### What is `asyncio.ensure_future`?

The function `ensure_future` lets us execute a `coroutine` (async function) in the background, without explicitly waiting for it to finish. If we need, we can wait for it later or poll for the result. In other words, this is a way of executing code in asyncio without await. [Click here](https://www.programcreek.com/python/example/85364/asyncio.ensure_future) to learn more.


#### What is `asyncio.gather`?

The function `gather` lets you fire off a bunch of coroutines simultaneously. In other words, it returns the results of each coroutine (async function) passed The return value is a list of responses from each coroutine. [Click here](https://piccolo-orm.com/blog/asyncio-gather/) to learn more.

## Task for the reader

- The reader can write the synchronous version of the above code and find out how the time difference between the 2 versions of code. 
- The reader can write a program to read those 100 files you just created and keep the contents of every file in a list.

# Resources for Learning More

- [Aynchronous Programming in Python](https://medium.com/velotio-perspectives/an-introduction-to-asynchronous-programming-in-python-af0189a88bbb)
- [Asynchronous file handling in Python](https://www.twilio.com/blog/working-with-files-asynchronously-in-python-using-aiofiles-and-asyncio)
- [Python asyncio Documentation](https://docs.python.org/3/library/asyncio.html)
- [Awesome Python async modules](https://pythonawesome.com/a-curated-list-of-awesome-python-asyncio-frameworks-libraries-software-and-resources/)


# Conclusion

File handling is often overlooked while doing asynchronous programming which leads to unintended slow ups as file handling is very resource and time-intensive for big files. Using `aiofiles` for file handling drastically decreases the execution time of the program due to its asynchronous nature. It does not have to block the whole program while reading a file, hence other files can be read/ write and modified in the meantime, so it's a lot faster than the default way of file handling in python.
