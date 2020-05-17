# wikipedia-revisions
download every wikipedia edit

[![status](https://github.com/DominicBurkart/wikipedia-revisions/workflows/Python%20application/badge.svg)](https://github.com/DominicBurkart/wikipedia-revisions/actions?query=is%3Acompleted+branch%3Amaster)


This script downloads every revision to wikipedia. It can output the revisions into a [sqlalchemy-supported database](https://docs.sqlalchemy.org/en/13/dialects/) or into single bz2-zipped csv file.

Revisions are output with the following fields:
- `id`: the revision id, numeric
- `parent_id`: parent revision id (if it exists), numeric
- `page_id`: id of page, numeric
- `page_title`: name of page
- `page_ns`: page namespace, numeric
- `timestamp`: timestamp with timezone
- `contributor_id`: id of contributor, numeric
- `contributor_name`: screen name of contributor
- `contributor_ip`: ip addresss of contributor
- `text`: complete article text after the revision is applied, string in wikipedia markdown
- `comment`: comment

The id, timestamp, page_id, page_title, and page_ns cannot be null. All other fields may be null.

System requirements:
- 4gb memory
- ~3tb storage
- python 3 & pip pre-installed

I wrote a [blog post](https://dominicburkart.com/blog/2020/big_data_and_small_computers.html) on some of the project 
goals and technical choices.

## Install

### CPython

CPython is the standard Python distribution.

For writing to a bz2-compressed csv file:
```sh
git clone https://github.com/DominicBurkart/wikipedia-revisions
cd wikipedia-revisions
pip3 install -r requirements.txt
```

For writing to a database:
```sh
git clone https://github.com/DominicBurkart/wikipedia-revisions
cd wikipedia-revisions
pip3 install -r database_requirements.txt
```

### PyPy

PyPy is an alternate, faster Python distribution. Note that setting 
up PyPy may require external dependencies.

For writing to a bz2-compressed csv file:
```sh
git clone https://github.com/DominicBurkart/wikipedia-revisions
cd wikipedia-revisions
pypy3 -m pip install -r requirements.txt
```

For writing to a postgres database (no other dialects supported in PyPy):
```sh
git clone https://github.com/DominicBurkart/wikipedia-revisions
cd wikipedia-revisions
pypy3 -m pip install -r database_requirements.txt
```
__Note__: writing out to a database using PyPy may be slower than CPython.

## Use

Use `--help` to see the available options:
```sh
python3 wikipedia_download.py --help
```

Output all revisions into a giant bz2-zipped csv:
```sh 
python3 wikipedia_download.py
```

Use a wikipedia dump from a specific date:
```sh
python3 wikipedia_download.py --date 20200101
```

Output to postgres database named "wikipedia_revisions" waiting at localhost port 5432:
```sh
python3 wikipedia_download.py --database
```

To set the database url:
```sh
python3 wikipedia_download.py --database --database-url postgres://postgres@localhost:5432/wikipedia_revisions
```

Note: PyPy is only supported for outputting to a CSV file or to a 
postgres database (using the driver `psycopg2cffi`). If using PyPy, a 
custom database url must point to a postgres database and start with 
`postgresql+psycopg2cffi`, as in 
`postgresql+psycopg2cffi:///wikipedia_revisions`.

## Configuration Notes
The above information is sufficient for you to run the program. The information below is useful for optimization.

- this program is I/O heavy and relies on the OS's [page cache](https://en.wikipedia.org/wiki/Page_cache). Having a few gigabytes of free memory for the cache to use will improve I/O throughput.
- using an SSD provides substantial benefits for this program, by increasing I/O speed and eliminating needle-moving cost.
- the `--low-memory` option more closely couples file reading and database I/O. It also limits the number of files actively processed to 2, which might be valuable if you are hitting your I/O constraints.
- if writing to a database stored on an external drive, run the program in a directory on a different drive than the database (and ideally the OS/swap). The wikidump is downloaded into the current directory, so putting them on a different disk than the output database avoids throughput and needle-moving issues. As an example configuration, here is the command that I used to process the revisions into a local postgres database using two external drives (an SSD, and a larger HDD that holds the output database). The `nohup` command prevents the command from stopping if the terminal process that spawned it is closed, and the output is saved in nohup.out. The tail program outputs the contents of nohup.out to the screen for monitoring. 
```sh
cd /path/to/ssd/without/db && > nohup.out && nohup time pypy3 -u  /path/to/wikipedia_download.py --database --date 20200401 --delete-database & tail -f nohup.out 
```
