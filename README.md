# IEC61850 C/C++ South plugin

A simple asynchronous IEC61850 plugin that pulls data from a server and sends it to Fledge.

## OrxaGrid Fork — What's Different From Upstream

This repository is OrxaGrid's fork of [mz-automation/fledge-south-iec61850](https://bitbucket.org/mz-automation/fledge-south-iec61850) (the `fledge-power`/mz-automation south plugin for IEC 61850), maintained here as `fledge-south-iec61850-1` for RDSS, OrxaGrid's SCADA/DMS platform, where it is the south-side integration for IEC 61850 protection relays (GE, ABB, SEL, and others).

Everything below this section is the generic upstream build/usage documentation and is still accurate. On top of upstream, OrxaGrid has fixed around ten real bugs found through live integration testing against real IEDs, not synthetic test models. In short, these fixes cover:

- Multi-DA-per-DO reports (e.g. ACT/ACD `Op`/`Str` with `general`/`phsA`/`phsB`/`phsC`) resolving to the wrong point or failing to decode
- DO-level dataset entries (no DA suffix) misreading the attribute or crashing the plugin on load
- Boolean-type (SPS/SPC) decoding that ignored the configured DA name and only matched the CDC-generic `stVal` default
- DA-suffixed objref reports/polling reading the wrong spec level or attribute, so data was never delivered for that documented config style
- A null/out-of-bounds crash (segfault) when value processing failed for a dataset member
- An `out_of_range` crash on DO-level dataset entries with neither a DA suffix nor an `[FC]` bracket, which took down the whole south service before it could connect
- A polling-cycle bug where one failed point silently shifted every subsequent point's label, so real data streamed in under the wrong asset name

See `git log` for the individual commits and full technical detail on each fix.

To build this plugin, you will need the lib61850 library installed on your environment as described below.

You also need to have Fledge installed from the source code, not from the package repository.

## Building lib61850

To install the dependencies you can run the requirements.sh script.

If you want to install them manually :

To build IEC61850 C/C++ Southth plugin, you need to download lib61850 at:
https://github.com/mz-automation/libiec61850

```bash
$ git clone https://github.com/mz-automation/libiec61850.git
$ cd libiec61850
$ export LIB_IEC61850=`pwd`
```
As shown above, you need a $LIB_IEC61850 env var set to the source tree of the
library.

Then, you can build libiec61850 with (note mbedtls installation):

```bash
$ cd libiec61850
$ cd third-party/mbedtls
$ wget -c https://github.com/Mbed-TLS/mbedtls/archive/refs/tags/v2.28.3.tar.gz -O - | tar -xz
$ mv mbedtls-2.28.3 mbedtls-2.28
$ cd ../../
$ cmake -DBUILD_TESTS=NO -DBUILD_EXAMPLES=NO ..
$ make
$ sudo make install
$ sudo ldconfig
```

Build
-----


To build the iec61850 plugin, once you are in the plugin source tree you need to run:

To build a release:

```bash 
$ mkdir build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Release ..
$ make
```

To build with unit tests and code coverage:

```bash
$ mkdir build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Coverage ..
$ make
```

- By default the Fledge develop package header files and libraries
  are expected to be located in /usr/include/fledge and /usr/lib/fledge
- If **FLEDGE_ROOT** env var is set and no -D options are set,
  the header files and libraries paths are pulled from the ones under the
  FLEDGE_ROOT directory.
  Please note that you must first run 'make' in the FLEDGE_ROOT directory.

You may also pass one or more of the following options to cmake to override
this default behaviour:

- **FLEDGE_SRC** sets the path of a Fledge source tree
- **FLEDGE_INCLUDE** sets the path to Fledge header files
- **FLEDGE_LIB sets** the path to Fledge libraries
- **FLEDGE_INSTALL** sets the installation path of Random plugin

NOTE:
- The **FLEDGE_INCLUDE** option should point to a location where all the Fledge
  header files have been installed in a single directory.
- The **FLEDGE_LIB** option should point to a location where all the Fledge
  libraries have been installed in a single directory.
- 'make install' target is defined only when **FLEDGE_INSTALL** is set

Examples:

- no options

  $ cmake ..

- no options and FLEDGE_ROOT set

  $ export FLEDGE_ROOT=/some_fledge_setup

  $ cmake ..

- set FLEDGE_SRC

  $ cmake -DFLEDGE_SRC=/home/source/develop/Fledge  ..

- set FLEDGE_INCLUDE

  $ cmake -DFLEDGE_INCLUDE=/dev-package/include ..
- set FLEDGE_LIB

  $ cmake -DFLEDGE_LIB=/home/dev/package/lib ..
- set FLEDGE_INSTALL

  $ cmake -DFLEDGE_INSTALL=/home/source/develop/Fledge ..

  $ cmake -DFLEDGE_INSTALL=/usr/local/fledge ..


Using the plugin
----------------

As described in the Fledge documentation, you can use the plugin by adding
a service from a terminal, or from the web API.C

1 - Add the service from a terminal:

.. code-block:: console

$ curl -sX POST http://localhost:8081/fledge/scheduled/task -d '{"name": "iec61850","plugin": "iec61850","type": "south","schedule_type": 3,"schedule_day": 0,"schedule_time": 0,"schedule_repeat": 30,"schedule_enabled": true}' ; echo

Or

2) Add the service from the web GUI:

- On the web GUI, go to the South tab
- Click on "Add +"
- Select iec61850 and give it a name, then click on "Next"
- Change the default settings to your settings, then click on "Next"
- Let the "Enabled" option checked, then click on "Done"