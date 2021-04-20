Collect Data About Builds
=========================

We may want to take some decisions in the future about deprecations and supported versions.
Right now we don't have data about the usage of packages and their versions on Read the Docs.

Configuration file
------------------

We are saving the config file in our database,
but to save some space we are saving it only if it's different than the one from a previous build
(if it's the same we save a reference to it).

The config file being saved isn't the original one used by the user,
but the result of merging all defaults.
Is saved using a _fake_ ``JSONField``
(charfield that is transformed to json when creating the model object).
For this reasons we can't query them or download them in bulk without iterating over all objects.

We may also want to have the original config file,
so we know what users are really setting.

Python
------

We can get the Python version from the config file when using a Python environment,
and from the ``conda list`` output when using a Conda environment.

PIP packages
------------

We can get a json with all root dependencies with ``pip list --not-required --format json``.

Conda packages
--------------

We can get a json with all dependencies with ``conda list --json``.
That command gets all the root dependencies and their dependencies,
so we may getting some noise, we can use ``pip list --not-required --format json``
as secondary source.

APT packages
------------

This isn't implemented yet,
but we can get the list from the config file or using ``apt list --installed`` or similar.

Format
------

.. code-block:: jsonc

  {
    "project": "docs",
    "version": "latest",
    "build": 12,
    // full timestamp or just year/month/day?
    "date": "2021-04-20-...",
    "user_config": {},
    "final_cofig": {},
    "packages": {
      "pip": [{
         "name": "sphinx",
         "version": "3.4.5"
      }],
      "conda": [{
         "name": "sphinx",
         "channel": "conda-forge",
         "version": "0.1"
      }],
    },
    "python": "3.7",
    // This may be better in the build object itself?
    "trigger": "api | manual | webhook | automation rule",
  }

Storage
-------

We can save all this information in json files in storage.
Then we could use a tool to import all this data into.

We can try to calculate a hash of the file and not upload duplicates that happen on the same day.

I think we are fine aggregating this data per year/month.

.. TODO: decide in an structure to save this files in storage.
   /telemetry/builds/{year}/{month}/{year}-{month}-{day}-{timestamp|pk}.json

Analyzing the data
------------------

Decide for a tool or service where to fed all this data.
Maybe just fed the data there and don't use storage? Vendor locking?

Changes
-------

- Allow to ignore stdout (already done in https://github.com/readthedocs/readthedocs.org/pull/8123/)
- Setup new bucket in storage (easy).
