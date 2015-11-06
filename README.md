# wsperf

**wsperf** is a WebSocket load testing probe that can be used for load testing and performance benchmarking of WebSocket servers:

 * comes as a command line tool that builds on [WebSocket++](http://www.zaphoyd.com/websocketpp), a high-performance C++/ASIO based WebSocket library.
 * designed to work stand-alone or being controlled from **wstest**, which comes as part of the [Autobahn WebSocket testsuite](http://autobahn.ws/testsuite/).

## Description

The first test mode implemented is **WebSocket Handshaking**.

In this mode, **wsperf** will:

  1. open a TCP connection to a testee
  2. perform a WebSocket opening handshake
  3. perform a WebSocket closing handshake
  4. close the TCP connection

It can do so in parallel and using multiple threads. You can fine control exactly how it operates (see usage section below).

It will create a detailed log file (JSON format) with high-precision timestamps for all states a connection goes through (and also of course if the connection was successful at all).

The generated log file can then be post-processed with **wstest** to obtain statistics like in the following:

	Aggregate results (WebSocket Opening+Closing Handshake)
	
	          Duration:      13.4 s
	             Total:    200000
	           Success:    200000
	              Fail:         0
	            Fail %:      0.00
	    Handshakes/sec:     14883
	
	     Min:       2.0 ms
	      SD:      10.7 ms
	     Avg:      67.6 ms
	  Median:      67.3 ms
	  q90   :      81.2 ms
	  q95   :      85.2 ms
	  q99   :      93.2 ms
	  q99.9 :     104.9 ms
	  q99.99:     108.3 ms
	     Max:     109.2 ms
	
	
	Analyze done.
 
## Building

**wsperf** is currently developed and tested on Unix like systems (I use Ubuntu 14.04 LTS x64).

You will need a *decent* C++ compiler, currently at least *GCC 4.6* or *clang X.X*.

The only dependencies of **wsperf** are:

  * [Boost](http://boost.org/)
  * [WebSocket++](https://github.com/zaphoyd/websocketpp)

### Boost

Don't waste time on your distro's packaged Boost - likely too old. Build from the source, Luke;)

Also see the [Boost Getting Started](http://www.boost.org/doc/libs/1_59_0/more/getting_started/unix-variants.html).

Get Boost from [here](http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.bz2/).

	cd ~/build
	tar xvjf ../tarballs/boost_1_59_0.tar.bz2
	cd boost_1_59_0

Configure to build using *Clang*

	./bootstrap.sh --prefix=$HOME/boost_1_59_0 --with-toolset=clang

Configure to build using *GCC*

	./bootstrap.sh --prefix=$HOME/boost_1_59_0 --with-toolset=gcc

Build the thing (using 8 CPU cores)

	./b2 -j8 install

Build on FreeBSD 9 with "NewStack":

   	./b2 -j8 toolset=clang cxxflags="-Wall -O3 -march=native -I/usr/local/include -I/usr/include -std=c++0x -stdlib=libc++" linkflags="-L/usr/local/lib -L/usr/lib -L/lib -lcxxrt -stdlib=libc++"

Building will take a some time.

### WebSocket++

WebSocket++ is a header-only library. So all you need is:

	cd ~/scm/3rdparty
	git clone git@github.com:zaphoyd/websocketpp.git

### SCons

**wsperf** is built using [SCons](http://scons.org/), a Python based build tool.

So if you have Python installed, all you need is:

	easy_install scons

### wsperf

To build **wsperf**, you will need to have 2 environment variables set:

  * `BOOST_ROOT` pointing to a Boost installation
  * `WSPP_ROOT` pointing to a WebSocket++ source distribution

Like add the following to your `.bashrc`:

	export BOOST_ROOT=${HOME}/boost_1_59_0
	export WSPP_ROOT=${HOME}/scm/3rdparty/websocketpp

Now get the source and build

	cd ~
	git clone git@github.com:crossbario/wsperf.git
	cd wsperf
	scons

When successful, this should produce a `wsperf` executable (optimized, statically linked and unstripped).

To cleanup

	scons -uc

## Usage

Basic usage of **wsperf**:

	wsperf <wsuri> <threads> <connections> <low_watermark> <high_watermark> <result_file>

like e.g.

	wsperf ws://127.0.0.1:9000 4 200000 1000 2000 results.json

The `wsuri` the the WebSocket address of the testee.

The `threads` parameter controls the number of background worker threads to be spawned.

> It can also be `0` in which case the load is processed on the main thread. Note that ASIO will nevertheless create a background thread of asynchronous name resolution. So you see 2 threads for **wsperf** even if run with `threads==0`.
> 

The `connections` is the total number of WebSocket connections that are opened to the testee - not concurrently, but in total. Also note that **wsperf** will currently not retry a failing connection.

The `result_file` is the name of the log file to produce.

The `low_watermark` and `high_watermark` control how many parallel connections will be in flight as follows:

**wsperf** will open new TCP connections to the testee and perform WebSocket opening handshakes on those as fast as it can up till the `high_watermark` connections is reached. That is connections which have not yet again been closed.

When that happens, it will stop trying to connect more. After some time, more WebSocket connections will get closed again (by performing a closing handshakes), and the outstanding number reaches the `low_watermark`, **wsperf** will start again connecting as fast as it can.

So the watermarks limit the number of WebSocket connections that haven't yet reached the "open" state .. hence are still in flight.

## Postprocessing

Writeme.
