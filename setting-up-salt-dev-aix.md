# Setting up a salt development environment on AIX #

I recently got salt up and running on an aix server, so here's a description of
how I did it. I'm not a build engineer or a rpm master, so there are probably
better ways of doing it than like this. Your mileage will vary.

RPM sources:
 
* [IBM Linux Toolbox](http://www-03.ibm.com/systems/power/software/aix/linux/toolbox/alpha.html)
* [Perzl.org's AIX Open Source Packages](http://www.perzl.org/aix/index.php?n=Main.HomePage)

## GCC ##

* gcc-4.8.1-1.aix7.1.ppc.rpm
* gcc-c++-4.8.1-1.aix7.1.ppc.rpm
* gcc-cpp-4.8.1-1.aix7.1.ppc.rpm
* gmp-5.0.5-1.aix5.1.ppc.rpm
* gmp-devel-5.0.5-1.aix5.1.ppc.rpm
* libgcc-4.8.1-1.aix7.1.ppc.rpm
* libiconv-1.14-2.aix5.1.ppc.rpm
* libmpc-1.0.1-2.aix5.1.ppc.rpm
* libstdc++-4.8.1-1.aix7.1.ppc.rpm
* libstdc++-devel-4.8.1-1.aix7.1.ppc.rpm
* mpfr-3.1.2-1.aix5.1.ppc.rpm

## Building Python with GCC ##

* autoconf-2.69-1.aix5.1.ppc.rpm
* bzip2-devel-1.0.6-1.aix5.1.ppc.rpm
* db4-4.7.25-2.aix5.1.ppc.rpm
* db4-devel-4.7.25-2.aix5.1.ppc.rpm
* expat-2.1.0-1.aix5.1.ppc.rpm
* expat-devel-2.1.0-1.aix5.1.ppc.rpm
* fontconfig-2.8.0-2.aix5.1.ppc.rpm
* freetype2-2.5.0-1.aix5.1.ppc.rpm
* gdbm-1.10-1.aix5.1.ppc.rpm
* gdbm-1.8.3-5.aix5.2.ppc.rpm
* gdbm-devel-1.10-1.aix5.1.ppc.rpm
* glib2-2.34.3-1.aix5.1.ppc.rpm
* libffi-3.0.13-1.aix5.1.ppc.rpm
* libffi-devel-3.0.13-1.aix5.1.ppc.rpm
* libpng-1.6.3-1.aix5.1.ppc.rpm
* libXft-2.3.1-1.aix5.1.ppc.rpm
* libXrender-0.9.7-1.aix5.1.ppc.rpm
* m4-1.4.16-1.aix5.1.ppc.rpm
* ncurses-5.9-1
* ncurses-devel-5.9-1
* openssl-1.0.1e-2.aix5.1.ppc.rpm
* openssl-devel-1.0.1e-2.aix5.1.ppc.rpm
* pkg-config-0.28-1.aix5.1.ppc.rpm
* python-2.7.5-1.src.rpm
* readline-6.2-4.aix5.1.ppc.rpm
* readline-devel-6.2-4.aix5.1.ppc.rpm
* sed-4.2.2-1.aix5.1.ppc.rpm
* sqlite-3.7.17-1.aix5.1.ppc.rpm
* sqlite-devel-3.7.17-1.aix5.1.ppc.rpm
* tcl-8.5.14-1.aix5.1.ppc.rpm
* tcl-devel-8.5.14-1.aix5.1.ppc.rpm
* tk-8.5.14-1.aix5.1.ppc.rpm
* tk-devel-8.5.14-1.aix5.1.ppc.rpm
* tkinter-2.7.5-1.aix6.1.ppc.rpm
* zlib-1.2.8-1.aix5.1.ppc.rpm
* zlib-devel-1.2.8-1.aix5.1.ppc.rpm

The python binaries from the IBM Linux Toolbox and the binaries from perzl.org
are both built with the IBM XLC compiler. If you have XLC, you can use the
binaries from perzl.org. If not, you can use a spec I have adapted from the
original perzl srpm. It will compile python with GCC and setup the module
builder to use GCC instead of XLC.

    rpm -i python-2.7.5-1.src.rpm
    wget -O /opt/freeware/src/packages/spec/python-2.7.5-2.spec \
    "http://github.com/kriberg/6351551/raw/27a8cf905468fd08a5ae619b9c73fa5ffcc334c2/python-2.7.5-2.spec"
    rpm -bb /opt/freeware/src/packages/spec/python-2.7.5-2.spec

If everything compiles correctly, you should have the python 2.7.5 rpms in
/opt/freeware/src/packages/RPMS/ppc/. 

    cd /opt/freeware/src/packages/RPMS/ppc/
    rpm -Uvh python-2.7.5-1.aix7.1.ppc.rpm python-devel-2.7.5-1.aix7.1.ppc.rpm \
    python-libs-2.7.5-1.aix7.1.ppc.rpm 


## Distribute ##

* [distribute-0.7.3.zip](https://pypi.python.org/packages/source/d/distribute/distribute-0.7.3.zip#md5=c6c59594a7b180af57af8a0cc0cf5b4a)

Now that we have a functioning python 2.7 installation, setup with GCC as the
compiler, we can use distribute to install most of salt's dependencies. Both
the packages from perzl and IBM are for python 2.6, so I installed it from
scratch instead.

    cd /opt/freeware/src
    wget "https://pypi.python.org/packages/source/d/distribute/distribute-0.7.3.zip#md5=c6c59594a7b180af57af8a0cc0cf5b4a"
    unzip distribute-0.7.3.zip
    cd distribute-0.7.3
    python setup.py install

# Salt dependencies #

As of 0.16.3, these are dependencies from the [salt requirements file](https://github.com/saltstack/salt/blob/develop/requirements.txt):

* Jinja2
* M2Crypto
* msgpack-python
* pycrypto
* PyYAML
* pyzmq >= 2.1.9

## PyZMQ ##

PyZMQ is the 0MQ wrapper. We'll build 0MQ though, instead of using the bundled 0MQ library.

* [zeromq-3.2.3.tar.gz](http://download.zeromq.org/zeromq-3.2.3.tar.gz)

To compile 0MQ, we need to setup some build flags first.

    export OBJECT_MODE=32
    export CC=gcc
    export CFLAGS="-maix32 -g -mminimal-toc -DSYSV -D_AIX -D_AIX32 -D_AIX41 -D_AIX43 -D_AIX51 -D_ALL_SOURCE -DFUNCPROTO=15 -O2 -I/opt/freeware/include"
    export CXX=g++
    export CXXFLAGS=$CFLAGS
    export F77=xlf
    export FFLAGS="-O -I/opt/freeware/include"
    export LD=ld
    export LDFLAGS="-L/opt/freeware/lib -Wl,-blibpath:/opt/freeware/lib64:/opt/freeware/lib:/usr/lib:/lib -Wl,-bmaxdata:0x80000000"
    export PATH="/opt/freeware/bin:/opt/freeware/sbin:/usr/local/bin:/usr/lib/instl:/usr/bin:/bin:/etc:/usr/sbin:/usr/ucb:/usr/bin/X11:/sbin:/usr/vac/bin:/usr/vacpp/bin:/usr/ccs/bin:/usr/dt/bin:/usr/opt/perl5/bin"

These are the same build flags that are used in the python spec.

    cd /opt/freeware/src
    wget "http://download.zeromq.org/zeromq-3.2.3.tar.gz"
    gunzip zeromq-3.2.3.tar.gz
    tar fx zeromq-3.2.3.tar
    cd zeromq-3.2.3
    ./configure --prefix=/opt/freeware
    gmake
    gmake install

Now that we have 0MQ, we can install the python wrapper.

    cd /opt/freeware/src
    easy_install -b . pyzmq
    cd pyzmq
    python setup.py build --zmq=/opt/freeware
    python setup.py install

## Jinja2 ##

Jinja has one dependency, MarkupSafe, but distribute will install that for us.

    easy_install jinja2

## M2Crypto ##

* openssl-1.0.1e-2.aix5.1.ppc.rpm
* openssl-devel-1.0.1e-2.aix5.1.ppc.rpm
* swig-1.3.40-1.aix5.1.ppc.rpm

I had issues with using easy_install directly for M2Crypto. SWIG failed with
some apparent problem with OpenSSL, but this worked for me:

    cd /opt/freeware/src
    easy_install -b . M2Crypto
    cd m2crypto
    python setup.py build_ext --openssl=/opt/freeware/
    python setup.py install

Remember to use the compiler settings listed for 0MQ, if you have changed shell.

## msgpack-python ##

I installed this with easy_install, but I got an compiler error. According to
the output though, it would use the python implementation instead. It's
probably slower, though.

## pycrypto ##

So pycrypto is an adventure in itself. I was smashing my head into walls
for some time before I managed to successfully compile it. First, I fetched the
source and tried to install it with magic:

    cd /opt/freeware/src
    easy_install -b . pycrypto
    cd pycrypto

That proved rather unsuccessfull with loads of warnings from the preprocessor.
After some googling, I moved the include for Python.h to the top of the list of
includes in the source files the compiler complained about. That did the trick,
but I can't really fathom why, as pycrypto compiles just fine with on linux.
Last, I ran into a linking error, comlaining about not able to link with
.rpl_malloc. I omitted that by setting ``#define HAVE_MALLOC 1`` and ``#define
rpl_malloc malloc`` in src/config.h.

    python setup.py build
    python setup.py install

## salt ##

I used the RPM spec from ``pkg/rpm/salt.spec`` as a reference for writing a
spec for AIX. I removed all the dependency checks on purpose, as I don't want
to make packages for all the dependencies. If I roll this out in a
non-development environment, I'll probably create a single rpm with all
dependencies and salt included.

    cd /opt/freeware/src/packages/SOURCES
    wget https://pypi.python.org/packages/source/s/salt/salt-0.16.3.tar.gz
    cd ../SPECS/
    wget https://gist.github.com/kriberg/6365078/raw/5fece1f9e5ffcd14663f9b4978967d21bd8f9cc8/salt.spec
    rpm -bb salt.spec
    cd ../RPMS/noarch/
    rpm -Uvh salt-*.rpm

You should now be able to start salt with:

    salt-minion -c /opt/freeware/etc/salt -l debug

If you didn't, well... 
