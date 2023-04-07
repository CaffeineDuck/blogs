---
title: "Creating a Python Module for Nepal Stock Exchange API"
seoDescription: "Nepal stock exchange has been gaining a lot of traction lately. Seeing that, I was interested so I created my API Wrapper for NEPSE API."
datePublished: Mon Jun 14 2021 09:51:26 GMT+0000 (Coordinated Universal Time)
cuid: ckpwfslb40fobzls18hbrcsrf
slug: creating-a-python-module-for-nepal-stock-exchange-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1623662058604/cRm10gmen.png
tags: api, python

---

# Introduction

Nepal stock exchange has been gaining a lot of traction lately. I was interested so I wondered if I could get data for my [discord bot](https://github.com/samrid-pandit/peacebot). I found a project [nepse](https://github.com/pyFrappe/nepse/), but I was not fond of that module. 


### Why I disliked [nepse](https://github.com/pyFrappe/nepse/) module

- The module was `synchronous` hence blocking
- The module returned data in `dict`
- The module returned data in `camelCase`
- The codebase was not properly managed
- There was no proper documentation

Hence, I decided to create my own **NEPSE API** [Wrapper](https://github.com/samrid-pandit/nepse-api).


### How [nepse-api](https://github.com/samrid-pandit/nepse-api) will be better than [nepse](https://github.com/pyFrappe/nepse/)

- It will be `asynchronous`
- It will return data as an `object`
- It will return the data in `snake_case`
- Its codebase will be properly managed
- It will have proper documentation
- It will fetch data from API rather than scraping,
- It will be type-hinted.


# My Experience

## How I started

I got started on my journey of getting **Github Stars**. I thought about how I was going to code it, and about my directory structure. I looked at other `API Wrappers` and got the basic idea about my code/ directory structure from there. 

**The roadmap I created:**


### 1. Creating python class for storing the data returned by API

It's not the best way, but I just simply started creating `Python Classes` with the `@dataclass` decorator by looking at what the API returned. Some of the fields were `null` so it was a hassle to determine their data type; in the end, I just used the `Any` data type from the `typing` module. The attributes weren't that consistent so it was a hassle overall.


### 2. Creating various handlers for getting various types of data like company data, broker data, market data, etc

I got the inspiration from other API wrappers that I had seen in general. I created a `MarketClient`, `SecurityClient`, and `BrokerClient` to get the market data, company data, and broker data respectively. Doing this was a good decision in my opinion because this made my codebase a bit more organized and easier to maintain.


### 3. Creating a caching system so that the user has the ability to either cache or fetch real-time data

I got the inspiration of doing this from [discord.py](https://github.com/rapptz/discord.py). I had recently figured out proper ways of caching. I used [cachetools](https://pypi.org/project/cachetools/) for caching. I was very excited and made caching non-customizable. But After talking with `@Santosh#2138` I figured out it was a bad idea to not provide the user with the choice to control caching. I fixed it and gave the user the choice to change the `cache_retain_time` and `cache_size` during initialization.


### 4. Making sure that the code is fully asynchronous and is not blocking

Just using `async/ await` doesn't mean your code is automatically asynchronous. The code still might be blocking, so I had to make sure everything was asynchronous and nothing was blocking, because if that were the case; my main motive for making this **API Wrapper** would have failed. I accomplished it by using `httpx.AsyncClient` instead of `requests` for fetching the data from an API. 

### 5. Converting the `camelCase` to `snake_case`

One of my motives for creating this **API Wrapper** was to make it more pythonic by having the data returned in `snake_case` rather than `camelCase`. I had to think of something creative for this because API returned the data in `camelCase`. I thought of using `regex` but I thought it would be too slow. I created a *monstrous* converter but later I figured out that using `regex` would make my code more maintainable and the performance gain won't be that noticeable, so I used the python module `pyhumps` (which uses regex) in the end for converting `camelCase` to `snake_case`.

### 6. Creating tests for testing my module

I had heard about `unit testing`, but I wasn't that familiar with it. I installed `pytest` to get started with it. But shortly I figured out that you need to do a bit of extra work to run pytest in `async code`. You can learn more about `async code` unit testing [here](https://promity.com/2020/06/03/testing-asynchronous-code-in-python/). 

Basically you need to `pip install pytest-asyncio` and your code should be like this: 
```py
@pytest.mark.asyncio
async def test_coroutine():
    res = await long_computation()
    assert res == "DONE"
```

### 7. Formatting the code

I used `black` for code formatting and `isort` for import sorting. These tools are very good and these make your code much more readable and elegant. I have been in love with these tools ever since I started using them. I would highly recommend you to use it as well.


### 8. Creating Documentation

As this was my first time creating a python package, I had no idea at all about creating documentation, but google and youtube had my back. I wrote doc-strings in [Google Docstring Fomat](https://stackoverflow.com/questions/3898572/what-is-the-standard-python-docstring-format). I used [sphinx](https://www.sphinx-doc.org/) to generate `auto-docs` from my code and used `restructured text` for the documentation. You can check out the live version of my documentation [here](https://nepse-api.readthedocs.io/)


### 9. Publishing module to **PyPi**

[PyPi](https://pypi.org/) is a repository of python modules for the Python programming language. If you publish your package here as `something-you-made`, people all over the world can install your package by just doing `pip install something-you-made`. I had firstly used `pipenv` for dependencies management, and `setup.py` for publishing the package, but after I found out about [Python Poetry](https://python-poetry.org/); my life has been so much easier. It makes it very easy to manage dependencies for your package and publish it in **PyPi** as well.


### 10. Managing open source project

Even though I am a huge advocate of open source projects, there are problems that you face while maintaining these projects. But now there are many tools in place to help you manage open source projects easily. [Github Actions](https://github.com/features/actions) is one of them. You can have various checks, tests that run automatically when an event is triggered such as `pull request` or `code push`. You can create a `CODE_OF_CONDUCT`, and proper contributing guides, issue labels, and projects to efficiently manage your open source project.


## Problems I faced along the way

### API Not Found

I started working but immediately after starting, I faced my very first hurdle. I was not able to find an API for getting the data about the Nepal stock exchange. But after struggling for some time, I figured out the API by looking at the `network` tab of **dev tools**. 


### Asynchronous code

I got started on my journey to getting Github stars. But the hurdles wouldn't stop. I had boldly claimed about making an `asynchronous` module. But I didn't have much experience with asynchronous coding. But countless hours of StackOverflow, Googling, and Python discord server surfing really did help me. If you want a full guide on this, **I may create a blog related to this.**

### Caching

I used [cachetools](https://pypi.org/project/cachetools/) for caching, but I was not sure which type of caching would be the best. I played around with `LRU` (Least Recently Used) caching method. But `LRU` had a fatal flaw for my use case. I wanted the cache to expire because the data was very volatile, but `LRU` wouldn't be able to suffice that. So, I decided to use `TTL` (Time To Live) caching method. Using `TTL` meant that the cache would expire so which was perfect for my use case.


### Legal issues

I don't know much about the IT policy or cyber laws of Nepal. So, when I heard that legal action could be taken against me just for making that python module, I was scared. But later I found out that, I won't be in any trouble because the API was public, and I had given reference and copyright to NEPSE for their data, and have never claimed their data as mine. A lesson for everyone to **fact-check** everything before believing anything blindly.



## Problems I haven't been able to solve 

### Getting live market data

Even though I was able to get the market data for a company from [New NepalStock Site](https://newweb.nepalstock.com/api/), it just wasn't possible to get real-time market data from that site. The data there updated very slowly and real-time data was not much found there. While searching for the solution for this problem, I discovered [Paid Nepse API](https://www.fiscalnepal.com/2020/12/28/2881/nepse-brings-data-api-real-time-data-within-30-seconds/) which returned real-time data, but 10k-60k per month was too costly for a school student like me. 


### NEPSE changing their API

This has been one of the biggest problems for me while making this API wrapper. Within a week of making this API wrapper, NEPSE did some breaking changes to their API. They changed the request method from `GET` to `POST` and added a payload of `id` is uniform across every request, but changes every time the `id` in response of `/market-open` endpoint changes. I created a [mapping](https://docs.google.com/spreadsheets/d/1wKMa4ai4OcXmzuDCLlxtoxje_my6pfbz8i36XgTxSBY/edit?usp=sharing) of `\market-open`'s `id` to `id` sent in the payload of the `POST` request. But it didn't work out much because the numbers went up to 70 so it was not feasible to create a mapping for such huge numbers. I tried to figure out a mathematical relationship between those, but that didn't work as well. You can learn about the issue in detail [here](https://github.com/Samrid-Pandit/nepse-api/issues/27).



# Conclusion

All of this was a **bitter-sweet** experience. I learned a lot of new things, I learned about properly working with `asynchronous` code in python, creating `context managers`, `creating async generators`, `creating documentation`, `github actions`, and many more. I faced a ton of problems along the way, but those problems helped me grow even more towards being a better developer. In the end, I would just like to say, if you have any questions you can always DM in discord at `sussy#0002`, and it would be very helpful of you if you could contribute in 
[nepse-api](https://github.com/Samrid-Pandit/nepse-api).

%[https://github.com/Samrid-Pandit/nepse-api]
