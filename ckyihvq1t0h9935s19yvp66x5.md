---
title: "How to save Python Objects in Redis?"
seoDescription: "Saving complex python objects in redis using aioredis as the redis client and pickle for seralizing and deserializing."
datePublished: Mon Jan 17 2022 09:38:25 GMT+0000 (Coordinated Universal Time)
cuid: ckyihvq1t0h9935s19yvp66x5
slug: how-to-save-python-objects-in-redis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1642407096360/dOKm_8tjI.jpeg
tags: redis, python, howtos

---

# Introduction

In this article, you'll learn about `pickle`, `Redis`, why you would want to save `python objects` in Redis, and how you would be able to achieve that with the **pros**, **cons**, and **alternatives** of doing so.

# What is Redis?

 [Redis](https://redis.io/topics/introduction)  (which stands for REmote DIctionary Server) is an open-source in-memory data structure store that is commonly used to implement non-relational key-value databases and caches. It is used as a caching database where the data is saved in key-value pairs. You can check  [this out](https://aioredis.readthedocs.io/en/latest/getting-started/) to learn more.

# What are Python Objects?

Python has been an object-oriented language since it existed. Unlike procedure-oriented programming, where the main emphasis is on functions, object-oriented programming stresses on objects.

An object is simply a collection of data (variables) and methods (functions) that act on those data. Similarly, a class is a blueprint for that object.

We can think of a class as a sketch (prototype) of a house. It contains all the details about the floors, doors, windows, etc. Based on these descriptions we build the house. House is the object.

As many houses can be made from a house's blueprint, we can create many objects from a class. An object is also called an instance of a class and the process of creating this object is called instantiation.

[Click here](https://www.geeksforgeeks.org/python-classes-and-objects/) if you want to learn more.



# Why would you want to save Python Objects in Redis?

Redis being a simple `caching server`, it doesn't come with many of the data types available in relational databases such as  [Postgres](https://www.postgresql.org/about/). Due to the lack of variety in data types, we may sometimes need to save `Python Objects` in the Redis server itself. 

You may sometimes want to save nested and complex python objects in the cache, which may be hard to `serialize` and `deserialize` manually into default data types provided by `Redis`.  That is the perfect use case for directly saving python objects in redis cache.


# How to save Python Objects in Redis? 

Redis stores everything as a string, hence we can save the python object as a binary string. One of the easiest ways to dump and load binary in Python for Python Objects is using the python module [pickle](https://docs.python.org/3/library/pickle.html).

# What is Pickle?

Python pickle module is used for serializing and de-serializing python object structures. The process to convert any kind of python object (list, dict, etc.) into byte streams (0s and 1s) is called `pickling` or serialization or flattening or marshaling. We can convert the byte stream (generated through pickling) back into python objects by a process called `unpickling`.

## How to use pickle?

```py
import pickle

class Person:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    def __str__(self) -> str:
        return f'{self.name} {self.age}'

def main() -> None:
    person_a = Person('John', 20)

    with open('test.pkl', 'wb+') as f:
        pickle.dump(person_a, f)

    with open('test.pkl', 'rb') as f:
        person_b = pickle.load(f)

    print(str(person_a), str(person_b), sep='\n')
    # OUTPUT:
    # John 20
    # John 20


if __name__ == '__main__':
    main()
```

#### Explanation of above code

1. Pickle is imported using `import pickle`
2. A class named `Person` is created.
3. `person_a` is created as an instance of the class `Person`.
4. 'test.pkl' file is opened with `write-binary`(wb) mode.
5. `person_a` object is dumped into 'test.pkl' after `pickling`.
6. 'test.pkl' file is opened with `read-binary`(rb) mode.
7. Data `unpickled` from the file is loaded into the variable `person_b`.
8. `person_a` and `person_b` variables are printed as strings.

> **Warning:** [The pickle module is not secure](https://davidhamann.de/2020/04/05/exploiting-python-pickle/). Only unpickle data you trust.

## How to use Pickle with Redis?

We're going to use a python module called [aioredis](https://aioredis.readthedocs.io/en/latest/getting-started/) as our redis client. You can install it using `pip install aioredis`. 

### Pure Redis

```py
import asyncio

import aioredis


async def main():
    redis = aioredis.from_url("redis://localhost")
    await redis.set("my-key", "value")
    value = await redis.get("my-key")
    print(value)
    # OUTPUT:
    # value


if __name__ == "__main__":
    asyncio.run(main())
```
#### Explanation of Above Code
1. Redis client is initialized by connecting to the `redis server` after providing the connection `URI`.
2. Value of "*my-key*" is set as "*value*" in the `redis server`.
3. Value of the key "*my-key*" is retrived from the `redis server`.

### Redis with Pickle

```py
import asyncio
import pickle

import aioredis


class Person:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    def __str__(self) -> str:
        return f'{self.name} {self.age}'


async def main():
    person_a = Person('John', 20)
    pickled_person_a = pickle.dumps(person_a)

    redis = aioredis.from_url("redis://localhost")
    await redis.set("person-a", pickled_person_a)
    value = await redis.get("person-a")

    unpickled_person_a = pickle.loads(value)

    print(str(person_a), str(unpickled_person_a), sep='\n')
    # OUTPUT:
    # John 20
    # John 20


if __name__ == "__main__":
    asyncio.run(main())
```

### Key Points
- The above code is `asynchronous` if you don't know about asynchronous programming. Please check [this out](https://medium.com/velotio-perspectives/an-introduction-to-asynchronous-programming-in-python-af0189a88bbb).
- Don't set the `decode_responses` keyword-argument `True` while initializing `aioredis` client. I will write a detailed blog explaining why you shouldn't do this soon enough.
- In the Redis example, we're not saving the `pickled` objects in file, rather we're saving it in the redis-server so we don't have to open some file and write/ read from them. 

# Alternatives to Pickle

If you can cache your object in other ways while implementing the data structures and types available in Redis, you should use it instead of `pickle`.

One alternative is to convert the python object into JSON and save it. It can be done by using the `json` module. You'd have to convert the python object into a dictionary using the `vars` built-in method. After that you can use the `json.loads` and `json.dumps` functions to dump and load the dictionary. [Click here](https://pythonexamples.org/convert-python-class-object-to-json/) to learn more.

You can check this [thread](https://stackoverflow.com/questions/15219858/how-to-store-a-complex-object-in-redis-using-redis-py) on various alternatives for storing python objects in Redis.

# Vulnerabilities that come with using Pickle and Redis

What is actually happening behind the scenes while `unpickling` is that the byte-stream created by `pickle.dumps` contains opcodes that are then one-by-one executed as soon as we load the pickle back in. For this reason, it's really easy to create a vulnerable app. `unpickling` is almost equivalent to using `eval()` so if your Redis server is compromised, your whole app would be compromised. You can check [this out](https://davidhamann.de/2020/04/05/exploiting-python-pickle/) to learn more.

Credits: [Regmi C. Mahesh](https://www.facebook.com/regmicmahesh)

# Conclusion

`pickle` is a great tool for storing python objects in `Redis`, but it comes with its own drawbacks. Pickle would be perfect if you're only looking to store python objects in Redis quick and easy. You're recommended to not use it in a production environment unless you have checked out other alternatives and weighed the pros and cons of using pickle. In conclusion, I would recommend you check out [other alternatives](https://www.reddit.com/r/Python/comments/2gptxg/saving_a_python_object_in_redis/) and decide which would be perfect for your use case.