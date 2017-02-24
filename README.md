# bagit-python

[![Build Status](https://travis-ci.org/LibraryOfCongress/bagit-python.svg)](http://travis-ci.org/LibraryOfCongress/bagit-python)

bagit is a Python library and command line utility for working with  [BagIt](http://purl.org/net/bagit) style packages.

## Installation

bagit.py is a single-file python module that you can drop into your project as
needed or you can install globally with:

    pip install bagit

Python v2.6+ is required.

## Command Line Usage

When you install bagit you should get a command-line program called bagit.py
which you can use to turn an existing directory into a bag:

    bagit.py --contact-name 'John Kunze' /directory/to/bag

### Finding Bagit on your system

The `bagit.py` program should be available in your normal command-line window
(Terminal on OS X, Command Prompt or Powershell on Windows, etc.). If you are
unsure where it was installed you can also request that Python search for
`bagit` as a Python module: simply replace `bagit.py` with `python -m bagit`:

    python -m bagit --help

On some systems Python may have been installed as `python3`, `py`, etc. –
simply use the same name you use to start an interactive Python shell:

    py -m bagit --help
    python3 -m bagit --help

### Configuring BagIt

You can pass in key/value metadata for the bag using options like
`--contact-name` above, which get persisted to the bag-info.txt. For a
complete list of bag-info.txt properties you can use as commmand line
arguments see `--help`.

Since calculating checksums can take a while when creating a bag, you may want
to calculate them in parallel if you are on a multicore machine. You can do
that with the `--processes` option:

    bagit.py --processes 4 /directory/to/bag

To specify which checksum algorithm(s) to use when generating the manifest,
use the --md5, --sha1, --sha256 and/or --sha512 flags (MD5 is generated by default).

    bagit.py --sha1 /path/to/bag
    bagit.py --sha256 /path/to/bag
    bagit.py --sha512 /path/to/bag

If you would like to validate a bag you can use the --validate flag.

    bagit.py --validate /path/to/bag

If you would like to take a quick look at the bag to see if it seems valid
by just examining the structure of the bag, and comparing its payload-oxum (byte
count and number of files) then use the `--fast` flag.

    bagit.py --validate --fast /path/to/bag

And finally, if you'd like to parallelize validation to take advantage of
multiple CPUs you can:

    bagit.py --validate --processes 4 /path/to/bag

## Using BagIt in your programs

You can also use BagIt programatically in your own Python programs by importing
the `bagit` module.

### Create

To create a bag you would do this:

```python
bag = bagit.make_bag('mydir', {'Contact-Name': 'John Kunze'})
```

`make_bag` returns a Bag instance. If you have a bag already on disk and would
like to create a Bag instance for it, simply call the constructor directly:

```python
bag = bagit.Bag('/path/to/bag')
```

### Update Bag Metadata

You can change the metadata persisted to the bag-info.txt by using the `info`
property on a `Bag`.

```python
# load the bag
bag = bagit.Bag('/path/to/bag')

# update bag info metadata
bag.info['Internal-Sender-Description'] = 'Updated on 2014-06-28.'
bag.info['Authors'] = ['John Kunze', 'Andy Boyko']
bag.save()
```

### Update Bag Manifests

By default `save` will not update manifests. This guards against a situation
where a call to `save` to persist bag metadata accidentally regenerates
manifests for an invalid bag. If you have modified the payload of a bag by
adding, modifying or deleting files in the data directory, and wish to
regenerate the manifests set the `manifests` parameter to True when calling
`save`.

```python

import shutil, os

# add a file
shutil.copyfile('newfile', '/path/to/bag/data/newfile')

# remove a file
os.remove('/path/to/bag/data/file')

# persist changes
bag.save(manifests=True)
```

The save method takes an optional processes parameter which will
determine how many processes are used to regenerate the checksums.
This can be handy on multicore machines.

### Validation

If you would like to see if a bag is valid, use its `is_valid` method:

```python
bag = bagit.Bag('/path/to/bag')
if bag.is_valid():
    print "yay :)"
else:
    print "boo :("
```

If you'd like to get a detailed list of validation errors,
execute the `validate` method and catch the `BagValidationError`
exception. If the bag's manifest was invalid (and it wasn't caught by the
payload oxum) the exception's `details` property will contain a list of
`ManifestError`s that you can introspect on. Each ManifestError, will be of
type `ChecksumMismatch`, `FileMissing`, `UnexpectedFile`.

So for example if you want to print out checksums that failed to validate
you can do this:

```python

bag = bagit.Bag("/path/to/bag")

try:
  bag.validate()

except bagit.BagValidationError, e:
  for d in e.details:
    if isinstance(d, bag.ChecksumMismatch):
      print "expected %s to have %s checksum of %s but found %s" % \
        (e.path, e.algorithm, e.expected, e.found)
```

To iterate through a bag's manifest and retrieve checksums for the payload
files use the bag's entries dictionary:

```python
bag = bagit.Bag("/path/to/bag")

for path, fixity in bag.entries.items():
  print "path:%s md5:%s" % (path, fixity["md5"])
```

## Contributing to bagit-python development

    % git clone git://github.com/LibraryOfCongress/bagit-python.git
    % cd bagit-python
    # MAKE CHANGES
    % python test.py

### Running the tests

You can quickly run the tests by having setuptools install dependencies:

    python setup.py test

Once your code is working, you can use [Tox](https://tox.readthedocs.io/) to run
the tests with every supported version of Python which you have installed on
the local system:

    tox

If you have Docker installed, you can run the tests under Linux inside a
container:

    % docker build -t bagit:latest . && docker run -it bagit:latest

## Benchmarks

If you'd like to see how increasing parallelization of bag creation on
your system effects the time to create a bag try using the included bench
utility:

    % ./bench.py

License
-------

[![cc0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

Note: By contributing to this project, you agree to license your work under the
same terms as those that govern this project's distribution.
