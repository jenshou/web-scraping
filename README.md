# web-scraping

Examples of scraping data from websites in Python in various form: 

* Open API, clean REST API(OMDB, openpayments) - in [openpayments]

* Registered API - in [twitter]
Use tweepy to fetch tweets.

* Complicated HTML with JavaScript - in [amazon]
Created a function that returns a list of tuples, one per book. Inspected with Inspection mode in <span> tags and scraped with user-agent. Output form:
  ```python
books = parseAmazonBestSellers()
for price, title, author, href in books:
    print(title, author, price)
    print(href)
```

looks like:

```
Born to Run by Bruce Springsteen $19.50
https://www.amazon.com/Born-Run-Bruce-Springsteen/dp/1501141511/ref=zg_bs_books_1/158-6476629-1853566

The Girl on the Train by Paula Hawkins $9.60
https://www.amazon.com/Girl-Train-Paula-Hawkins/dp/1594634025/ref=zg_bs_books_2/158-6476629-1853566

Cooking for Jeffrey: A Barefoot Conte... by Ina Garten $21.00
https://www.amazon.com/Cooking-Jeffrey-Barefoot-Contessa-Cookbook/dp/030746489X/ref=zg_bs_books_3/158-6476629-1853566
...
```




