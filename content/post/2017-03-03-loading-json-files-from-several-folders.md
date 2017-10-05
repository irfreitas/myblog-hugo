---
title: "Loading JSON files from several folders"
author: "Isaac Freitas"
date: "2017-03-03T21:07:54-06:00"
---

For my dissertation research, I used the [twarc](https://github.com/DocNow/twarc) command line and python package to collect tweets related to the flooding in Louisiana during August of 2016.  A useful utility which comes with this project is twarc-archive.py which allows passing keywords in which are then used to download all tweets (within the API limits) which mention those words.  Each time twarc-archive.py is run, it keeps a running log file of the progress and status of tweets being collected and creates a new .json file.
<!--more-->

For my data collection, I ran the program every day for four months for several different searches.  This resulted in multiple folders (one for every search) and dozens of json files in each folder.  Long story short, to do any further analysis I had to learn how to load all that json data into a usable format.  I found inspiration from a few stackoverflow posts such as [this](http://stackoverflow.com/questions/38511163/parsing-multiple-json-files) and [this](http://stackoverflow.com/questions/21533894/how-to-read-line-delimited-json-from-large-file-line-by-line) and [this](http://stackoverflow.com/questions/1744989/read-from-file-or-stdin), but ended up making an amalgamation with the best parts of each.

The three pip-installable packages that I used are **glob**, **json**, and **fileinput**. Another utility from within the twarc is the **json2csv.py** file.  **Glob** enables reading in both lists of folders along a path as well as lists of folders meeting certain criteria.  The **json** package allows load and dumping of json data, and **fileinput** is useful for reading in data from a file or from command line arguments.   The json2csv utility in twarc has functions to allow easily reading in the twitter API json results into a list of lists in addition to outputting csv files.

To get started, we import the packages and setup a path to where our folders are.

``` python 
import glob
import json
import fileinput
import /path/to/twarc/utils/json2csv.py as j2c

path = "/path/to/folders/"
```

To create a list of the folders along my path, you can pass your path into glob.glob() with a list comprehension and store the results in the jsonfolders variable.  To get all of the json files within all of the folders, you can either do a double for loop or you can combine two list comprehensions to iterate through the folders and glob up all the files.  This gives a final list of every json file.

``` python
jsonfolders = [x for x in glob.glob(jfolder + "/*")]
alljsonfiles = [y for x in range(len(jsonfolders))
                for y in glob.glob(jsonfolders[x] + "/*.json")]
```

Once you have the list of json files, you can then iterate to read over each json file.  Twarc outputs line-delimited json so you want to read in each line as a json object.  The **fileinput.input(file)** allows running a for loop over each line to read in the line-delimited json. Finally, we use the **get_row()** function from the json2csv package to flatten the data to create a list of lists in our variable **empty_lst**.

``` python
empty_lst = []
for file in alljsonfiles:
    for line in fileinput.input(file):
        tweet = json.loads(line)
        empty_lst.append(j2c.get_row(tweet))
```

And to wrap everything up, I tied everything together into a function for reading every line of every json file in every folder. The resulting list of lists can be turned into a **pandas** dataframe easily for further analysis or written to csv for simple storage.  With this function, I was able to load about eight hundred thousand tweets in a relatively short time.  I hope this walkthrough will be useful for others!

``` python
def read_json_data(jfolder):
    empty_lst = []
    jsonfolders = [x for x in glob.glob(jfolder + "/*")]
    alljsonfiles = [y for x in range(len(jsonfolders))
                    for y in glob.glob(jsonfolders[x] + "/*.json")]
    for file in alljsonfiles:
        for line in fileinput.input(file):
            tweet = json.loads(line)
            d_lst.append(j2c.get_row(tweet))
    return d_lst
```

If you like the walkthrough, send me a tweet about it.  See the links at the bottom of the page.
