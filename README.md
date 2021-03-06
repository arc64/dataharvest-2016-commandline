
# Your data is too big for excel? Command line tools for journalists

## Oh no why?
What is this terminal thing you ask? And why do we want to use it?

* What if the CSV file we want to open is too big for your spreadsheet program?
* Feel like having a quick look at what the data is without opening the file and needing to take a smoke break?
* An inexplicable reason your file does not open in your chosen spreadsheet program...
* There is power here. Grab bits of the internet, manipulate them quickly, and pop them into files or databases without needing to learn the complexities of coding.

The terminal is a text based interface that allows you to run text commands and interact with your file system.

There are alternate names for the terminal including; console, shell, command line, command prompt.

##The beginning

The following tutorial is tailored to the Mac and Linux terminals.

You will need to open up the terminal;
* On a Mac or Linux machine just use the standard terminal program.
* If you have a windows machine please install Cygwin (a Unix terminal emulator) in order use the same commands as this tutorial - [Cygwin (Unix emulator)](https://cygwin.com/install.html) - https://cygwin.com/install.html


For those on Linux or Mac go right ahead and install csvkit for the last part of the tutorial:
```sh
$ easy_install pip
$ pip install csvkit
```

### What are those commands? Package management

Quick note easy_install, pip, and brew are all package managers that allow you to install what we need from the command line. Rather than downloading and trying to install and configure them yourself.

* [easy_install](https://en.wikipedia.org/wiki/Setuptools) is a package manager to install Python packages, however it isn't a fully fledged one. Pip however is, so first we install easy_install and then pip.
* [pip](https://en.wikipedia.org/wiki/Pip_(package_manager)) is another Python package manager. A lot of these packages are on the [Python Package Index](https://en.wikipedia.org/wiki/Python_Package_Index) (PyPI)

### When you have trouble installing

There could be a number of issues;
* dependencies
* permissions

With dependencies, you should 'oogle the specific error. Perhaps you already have somethings installed, perhaps you need to upgrade, perhaps your operating system has a specific issue. [Stackoverflow](http://stackoverflow.com/) is a good place to get answers.

Permissions can be a bit more confusing. All the commands we have shown so far are run with permissions that you (the default user) have. Sometimes this isn't enough and you need to run with bigger, grander permissions. Like a super hero, or in this case a super user (also known as root).

``` $ sudo <your command> ```

Be careful though, there are some ramifications to doing this.

TODO: explain this

##Getting all the data
There have been a lot of tutorials on scraping, getting data out of documents and freedom of information requests.

For this exercise I grabbed all the tweets relating to the #dataharvest16 twitter handle and popped them into a CSV file using a tool called [T](https://github.com/sferik/t). [Which has it's own tutorial we don't have time for](https://gist.github.com/dannguyen/7c592c4559ee64f753e5)

```sh
$ t search all 'EIJC16' --csv -n 3200 > Dataharvest16tweets.csv
```

This is how I'll write commands in this tutorial. These are the commands we will pop into the terminal in order to do stuff, like processing our data.

Don't worry if you don't have a CSV, you can convert to one

```sh
in2csv input_file.xls output_file.csv
```

## Moving around

When you open your terminal you'll be somewhere - but where is that?

```sh
$ pwd
```

Will show you the path that you are in right now.

TODO: mkdir, cd, navigation

List the shizzle in your directory. Command first and options after. List. All. With readable file sizes.

```sh
$ ls -alk
```

Notice that there is your username or perhaps other usernames (like root) listed. These are the owners of the files, and the letters on the very left are [permissions](https://en.wikipedia.org/wiki/File_system_permissions) (i.e -rw-r--r--).

Don't know what a command does?

```sh
$ man ls
```

## How will you get the data?

You will use wget to the file with out data (or [whole sites](http://www.linuxjournal.com/content/downloading-entire-web-site-wget))

```sh
$ wget https://github.com/arc64/dataharvest-2016-commandline/blob/master/dataharvest16tweets.csv
```
You may find you don't have wget installed. To install it, the easiest way is with another package manager called [Homebrew](http://brew.sh/). And then run ``` $ brew install wget ```

If you don't feel like installing the fabulous tool wget, just grab the file (click raw in github), and put it into the directory where you want to use it.


## So what does the file look like?

Count the words

```sh
wc -w dataharvest16tweets.csv
    3739 dataharvest16tweets.csv
```

Or lines

```sh
wc -l README.md
```

In a CSV though we want to count the number of rows

```sh
csvstat --count dataharvest16tweets.csv
    Row count: 197
```

Looking at the beginning and end of the file

```sh
head dataharvest16tweets.csv
```

```sh
tail dataharvest16tweets.csv
```

And searching for specific text

```sh
grep "@nrecherch" dataharvest16tweets.csv
```

Return only the top 10

```sh
grep "@nrecherch" dataharvest16tweets.csv | head
```

Get all the headers for the file

```sh
$ csvcut -n dataharvest16tweets.csv
  1: ID
  2: Posted at
  3: Screen name
  4: Text
```

Grab some columns

```sh
$ csvcut -c 3, 4 dataharvest16tweets.csv
```

Print it pretty

```sh
head dataharvest16tweets.csv | csvcut -c 3,4 dataharvest16tweets.csv | csvlook
```

Get some stats about your tweeters

```sh
csvcut -c 3 dataharvest16tweets.csv | csvstat
```

csvgrep -c 3 -m "Hackette7" | csvlook

```sh
$ csvcut -c 3 dataharvest16tweets.csv
```


Getting all the unique handles

```sh
$ csvcut -c 'Screen name' dataharvest16tweets.csv | sort | uniq -c | sort -rn
```

Getting just tweets by Hackette7

```sh
csvgrep -c 3 -m "Hackette7" dataharvest16tweets.csv | csvlook
```

What about just getting the RT

```sh
csvgrep -c 4 -r "^RT" dataharvest16tweets.csv
```

You can run sql queries on it, which if your file is very big will run slow (it is in memory!)

```sh
csvsql --query "SELECT * FROM dataharvest16tweets" dataharvest16tweets.csv > dataharvest16tweets_nrecherch.csv

```


If you really want to you can push this all to a sqllite database

```sh
$ cat dataharvest16tweets.csv | csvsql --table tweets --db sqlite:///tweets.sqlite --insert

$ sqlite3 tweets.sqlite

    sqlite> SELECT * FROM tweets;

    sqlite> .exit
```

What can CSVkit do?
* clean
* do sql
* fuzzy matching
* pop into a database


## CHEATSHEET

### Keyboard shortcuts

* tab completion: use to finish command, directory or file names, or to see directories or files that match
* ctrl + a: move to the beginning of a line
* ctrl + e: move to the end of a line
* ctrl + c: terminate a command that is running
* up arrow: cycle back through previous commands

### UNIX features you should know

* redirection - [\>](http://cli.learncodethehardway.org/book/ex15.html)
* append - \>>
* piping - [|](http://cli.learncodethehardway.org/book/ex15.html)

### Commands you have have heard in this tutorial
* [wc](http://www.compciv.org/unix-tools/#wc)
* [mv](http://www.compciv.org/unix-tools/#mv)


## Additional resources

* [Commandline basics and further reading](https://github.com/chrislkeller/nicar15-command-line-basics)

* [A murder mystery tour of the command line](http://veltman.tumblr.com/post/65613277843/the-command-line-murders-teaching-the-terminal)

* [A complete guide to the command line](http://cli.learncodethehardway.org/book/) - http://cli.learncodethehardway.org/book/

* [CSVkit](https://csvkit.readthedocs.io/en/0.9.1/)

* [Bad data guide](https://github.com/Quartz/bad-data-guide) - https://github.com/Quartz/bad-data-guide

in the beginning was the command line

```wget http://www.cryptonomicon.com/command.zip ; unzip command.zip ; nano command.txt```


For a workshop that [Annabel Church](https://twitter.com/annabelchurch) and [Sandhya Kambhampati](https://twitter.com/sandhya__k) did at Dataharvest in 2016.

With specific thanks to these tutorials and authors for their inspiraton
* Dan Nguyen - [Using the t and csvkit](https://gist.github.com/dannguyen/7c592c4559ee64f753e5)
* Jue Yang, AJ Vincens, Chris Keller - [Commandline basics and further reading](https://github.com/chrislkeller/nicar15-command-line-basics/blob/master/NICAR_15/README.md)
* Noah Veltman - [Learning how to solve murder](https://github.com/veltman/clmystery/blob/master/cheatsheet.md)


