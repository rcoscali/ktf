# Kernel Test Framework (KTF)

KTF is a Google Test-like environment for writing C unit tests for
kernel code.  Tests are implemented as kernel modules which declare
each test as part of a test case.  The body of each test case
consists of assertions.  Tests look like this:

	TEST(examples, hello_ok)
	{
		EXPECT_TRUE(true);
	}

"examples" is the test case name, "hello_ok" the test.
KTF provides many different types of assertions, see
kernel/ktf.h for the complete list.

Usually tests are added on test module init via

	ADD_TEST(test_name);

This registers the test with the KTF framework for later
execution.  There are many examples in the examples/
directory.

"ktfrun" is provided to execute tests, it communicates
with the KTF kernel module via netlink socket to query
available tests and trigger test execution.

The design priorities for KTF are to make it

 * easy to run tests.  Just ensure the ktf module is loaded,
   then load your test module and execute "ktfrun".

 * easy to interpret results.  Output from ktfrun is clear
   and can be filtered easily.  Assertion failures indicate
   the line of code where the failure occurred.  Results of
   the last test run are always available from
   /sys/kernel/debug/ktf/results/<test case name>

 * easy to add tests.  Adding a test takes a few lines of code.
   Just (re)build the test module, unload/reload and KTF can
   run the test.  See the examples/ directory for some hints.

 * easy to analyse test behaviour (code coverage, memory utilization
   during test execution).  We provide "ktfcov" to support enabling
   coverage support on a per-module basis.  Coverage data is
   available in /sys/kernel/debug/ktf/coverage, showing how often
   functions were called during the coverage period, and optionally
   any outstanding memory allocations originating from functions
   that were subject to coverage.

All of the above will hopefully help Linux kernel engineers
practice continuous integration and more thoroughly unit test
their code.

## User Documentation

See [./doc](./doc)


## build

## Building googletest

```
# apt install cmake
# mkdir ~/Sources
# cd Sources
# mkdir -p ktf-src
# cd ktf-src/
# git clone --recurse-submodules https://github.com/google/googletest.git
# mkdir -p build/`uname -r`
# cd build/`uname -r`
# mkdir googletest
# cd googletest/
# cmake -DBUILD_SHARED_LIBS=ON ~/Sources/ktf-src/googletest
# make
# sudo make install
```

## Building KTF

We need kernel headers and cpp packages to build. Finally once we have built ktf, we insert the kernel module.

```
# sudo apt install autoconf libnl-3-dev libnl-genl-3-dev libnl-cli-3-dev libnl-route-3-dev libnl-nf-3-dev libnl-idiag-3-dev libnl-xfrm-3-dev
# sudo apt install cpp linux-headers-`uname -r`
# cd ~/Sources/ktf-src
# git clone https://gitlab.com/rcoscali/ktf
# cd ktf
# autoreconf
# cd ~/Sources/ktf-src/build/`uname -r`
# mkdir ktf
# cd ktf
# PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ~/Sources/ktf-src/ktf/configure KVER=`uname -r`
# make
# sudo make install
# sudo mkdir /usr/local/kernel
# sudo cp -f kernel/ktf.ko /usr/local/kernel
# sudo insmod /usr/local/kernel/ktf.ko
```
