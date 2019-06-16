# Pulling data from (open) REST APIs

[Lists of source of public APIs](https://www.publicapis.com/)

This is how we use `requests` to fetch a webpage in Python:

```python
r = requests.get('http://www.cnn.com')
print(r.text)
```

If the URL of that request returns HTML, this is simply fetching a webpage. On the other hand, if the URL is returning data in some form, we would say that we are accessing a *REST* api. 

Let's take a look at some examples:

## JSON from openpayments.us

[www.openpayments.us](http://www.openpayments.us).
 
There is a REST data API available at URL template:

```
URL = "http://openpayments.us/data?query=%s"
```


```python
>>> import urllib
>>> quote("john chan")
'john%20chan'
>>> quote("john&chan")
'john%26chan'
```

The conversion uses the ASCII character code (in 2-digit hexadecimal) for space and ampersand. Sometimes you will see the space converted to a `+`, which also works: `John+Chan`.

This website gives you JSON, which is very easy to load in using the default `json` package:

```python
data = json.loads(jsondata)
```

Dump the JSON using `json.dumps()`.

**Exercise**: Change `csv` into `json` in the URL and see that you get JSON back now.

## Pulling movie data from IMDB

Let's try to pull down some more interesting data using the [OMDb API](http://www.omdbapi.com/):

> The OMDb API is a free web service to obtain movie information, all content and images on the site are contributed and maintained by our users.

Hmmm...as of 2018, you have to [get an API key](http://www.omdbapi.com/apikey.aspx). They will send you a key by email that you must activate. [Never store your API key in your code](https://www.zdnet.com/article/over-100000-github-repos-have-leaked-api-or-cryptographic-keys/).

Let's also learn a more convenient way to specify URL parameters (with a dictionary).

```python
URL = "http://www.omdbapi.com/?"

args = {
	's' : 'cats',
	'r' : 'json',
   'apikey' : YOUR_OMDB_API_KEY
}

r = requests.get(URL, params=args)
```

**Exercise**: Use this code as a starting point and extract data from the movie database. You can change the search string to anything you want.  My output looks like:

```bash
$ python omdb.py YOUR_OMDB_API_KEY | jq
{
  "Search": [
    {
      "Title": "Cats & Dogs",
      "Year": "2001",
      "imdbID": "tt0239395",
      "Type": "movie",
      "Poster": "https://m.media-amazon.com/images/M/MV5BY2JmMDJlMmEtYTk4OS00YWQ5LTk2NzMtM2M3NzhkMjI4MGJkL2ltYWdlL2ltYWdlXkEyXkFqcGdeQXVyMjUzOTY1NTc@._V1_SX300.jpg"
    },
    {
      "Title": "The Truth About Cats & Dogs",
      "Year": "1996",
      "imdbID": "tt0117979",
      "Type": "movie",
      "Poster": "https://ia.media-imdb.com/images/M/MV5BOWM0MTA4NjItMzM3ZS00NDJmLTg3NWItNGE5ODIyOGJhNzQ0L2ltYWdlL2ltYWdlXkEyXkFqcGdeQXVyMTQxNzMzNDI@._V1_SX300.jpg"
    },
    ...
```

Look at `r.url` so you can see how it encodes the dictionary as GET arguments on the URL. [Solutions](https://github.com/parrt/msds692/tree/master/notes/code/omdb)

**Exercise**:  Write a small Python script using base URL `http://www.omdbapi.com` and parameters `t` (movie title) and `y` (movie year) to look up some of your favorite movies. The default output should come back in JSON.  Here is the structure of the argument dictionary:

```python
args = {
	't' : movie_title,
	'y' : movie_year,
   'apikey' : YOUR_OMDB_API_KEY
}
```

Now, change your program so that it requests data back in XML format:

```python
args = {
	't' : movie_title,
	'y' : movie_year,
	'r' : 'xml',
   'apikey' : YOUR_OMDB_API_KEY
}
```

Replace the json conversion code with code that [untangle](https://untangle.readthedocs.io/en/latest/)'s the XML to print out the title and the plot. All of the REST parameters are explained in the [API document](http://www.omdbapi.com/).  Recall that untangle lets you refer to children of `x` with `x.childname`. Parse with `x = untangle.parse(testxml)`. Attributes of a specific node are stored in a dictionary so `x`'s attributes are `x['attributename']`. You will have to look at the structure of the XML to figure out how to dig down into the tree.

[Solutions](https://github.com/parrt/msds692/tree/master/notes/code/omdb)

## CURLing for it

Now that we've done it the hard way in Python, let's repeat the exercises from above using one-liners on the shell. The `curl` program is your friend and can do all sorts of amazing things. Here are the 4 GETs using curl:

```bash
curl "http://ichart.finance.yahoo.com/table.csv?s=TSLA"
curl "http://openpayments.us/data?query=John+Chan"
curl "http://www.omdbapi.com/?s=cats&r=json"
curl "http://www.omdbapi.com/?t=Star+Wars"
```

`curl` writes the output to standard out so you can redirect into a file.

Notice that we have to url encode the arguments, so ` ` becomes `+`.

Next, let's look at how we might process JSON using the command line. First, install [jq](https://stedolan.github.io/jq/), which is a very handy JSON formatting and querying tool:

```bash
$ brew install jq  # for macs
```

```bash
$ curl -s "http://www.omdbapi.com/?s=cats" | jq
{
  "Search": [
    {
      "Title": "Cats & Dogs",
      "Year": "2001",
      "imdbID": "tt0239395",
      "Type": "movie",
      "Poster": "http://ia.media-imdb.com/images/M/MV5BMjExMjIwNzE4OV5BMl5BanBnXkFtZTYwNTY0MDI5._V1_SX300.jpg"
    },
...
```

Given that output, let's use `jq` to extract the list of IDs:

```bash
$ curl -s "http://www.omdbapi.com/?s=cats" | jq '.Search[].imdbID'
"tt0239395"
"tt0117979"
"tt1287468"
"tt1426378"
"tt0118829"
"tt1223236"
"tt0465315"
"tt0217990"
"tt2345459"
"tt0285371"
```

We can get rid of those pesky quotes with `tr`:
 
```bash
curl -s "http://www.omdbapi.com/?s=cats" | jq '.Search[].imdbID' | tr -d '"'
tt0239395
tt0117979
tt1287468
tt1426378
tt0118829
tt1223236
tt0465315
tt0217990
tt2345459
tt0285371
```

Why would I want to do that? so I can use a for loop in the shell to go fetch all of those. First, let's get those into a variable:

```bash
$ ids=$(curl -s "http://www.omdbapi.com/?s=cats" | jq '.Search[].imdbID' | tr -d '"')
```

Then we can loop over these IDs to individually get their records and write them to files:

```bash
$ for id in $ids
do
	curl -s "http://www.omdbapi.com/?i=$id" > /tmp/$id.json
done
```

That writes out a bunch of files, which we can look at again with `jq` four nice formatting:

```bash
$ jq < /tmp/tt0117979.json 
{
  "Title": "The Truth About Cats & Dogs",
  "Year": "1996",
  "Rated": "PG-13",
  "Released": "26 Apr 1996",
  "Runtime": "97 min",
  "Genre": "Comedy, Romance",
  "Director": "Michael Lehmann",
  "Writer": "Audrey Wells",
  "Actors": "Uma Thurman, Janeane Garofalo, Ben Chaplin, Jamie Foxx",
  "Plot": "A successful veternarian & radio show host with low self-esteem asks her model friend to impersonate her when a handsome man wants to see her.",
  "Language": "English",
  "Country": "USA",
  "Awards": "1 nomination.",
  "Poster": "http://ia.media-imdb.com/images/M/MV5BMTYxNjIyMjkxNl5BMl5BanBnXkFtZTYwMDAzOTg4._V1_SX300.jpg",
  "Metascore": "64",
  "imdbRating": "6.3",
  "imdbVotes": "22,358",
  "imdbID": "tt0117979",
  "Type": "movie",
  "Response": "True"
}
```
