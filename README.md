# openvpn3

# Building OpenVPN 3 on Mac OS X
Official link: https://staging.openvpn.net/openvpn3/

------------------------------

OpenVPN 3 should be built in a non-root Mac OS X account. Make sure that Xcode is installed with optional command-line tools. (These instructions have been tested with Xcode 7.3.1).

## Step 1
Create a directory ~/src and ~/src/mac. This can be done with the command:
<pre>
  mkdir -p ~/src/mac
</pre>

## Step 2
Expand the OpenVPN 3 tarball:
<pre>
 cd ~/src
 tar xf openvpn3.tar.gz
</pre>

## Step 3
Export the shell variable O3 to point to the OpenVPN 3 top level directory:
<pre>
 export O3=~/src/openvpn3
</pre>

## Step 4
Download source tarballs (.tar.gz or .tgz) for these dependency libraries into 
<pre>
~/Downloads
</pre>

## Step 5
See the file ~/src/openvpn3/lib-versions for the expected version numbers of each dependency.  If you want to use a different version of the library than listed here, you can edit this file.

<pre>
export LZO_VERSION=lzo-2.06
export LZ4_VERSION=lz4-r131
export SNAPPY_VERSION=snappy-1.1.3
export POLARSSL_VERSION=polarssl-1.3.6
export OPENSSL_VERSION=openssl-1.0.1g
export BOOST_VERSION=boost_1_61_0
</pre>

1. Boost -- http://www.boost.org/
https://sourceforge.net/projects/boost/files/boost/1.61.0/boost_1_61_0.tar.gz

2. PolarSSL (1.3.4 or higher) -- https://polarssl.org/
https://tls.mbed.org/download/polarssl-1.3.6-gpl.tgz

3. OpenSSL (1.0.1) -- http://www.openssl.org/
ftp://ftp.openssl.org/source/old/1.0.1/openssl-1.0.1g.tar.gz

4. Snappy -- https://code.google.com/p/snappy/
https://github.com/google/snappy/releases/download/1.1.3/snappy-1.1.3.tar.gz

5. LZ4 -- https://code.google.com/p/lz4/
https://codeload.github.com/Cyan4973/lz4/tar.gz/r131

Note that while LZO is listed in lib-versions, it is
not required for Mac builds.

OpenSSL is required, however OpenVPN 3 for Mac doesn't use OpenSSL in the standard way.  Instead, it cherry-picks some ASM-optimized crypto and hash algorithms from OpenSSL to speed up PolarSSL's low-level crypto processing, and assembles them into a library called libminicrypto.a.


## Step 6
Add build target in ~src/openvpn3/scripts/mac/build-boost
<pre>
Line 18: export TARGETS="osx osx64 osx-dbg"
</pre>

## Step 7
Update environment variables in ~src/openvpn3/vars-osx
<pre>
Line 6: export MIN_DEPLOY_TARGET="-mmacosx-version-min=10.11"
Line 8: export OTHER_COMPILER_FLAGS="-fvisibility=hidden -fvisibility-inlines-hidden -stdlib=libc++"
</pre>

## Step 8
Update environment variables in ~src/openvpn3/vars-osx64
<pre>
Line 6: export MIN_DEPLOY_TARGET="-mmacosx-version-min=10.11"
Line 8: export OTHER_COMPILER_FLAGS="-fvisibility=hidden -fvisibility-inlines-hidden -stdlib=libc++"
</pre>

Be aware that your double quote marks should be " , not ”   


## Step 9
Update Jam file path in ~src/openvpn3/boost/build-boost
<pre>
Line 31: cat >>tools/build/boost-build.jam <<EOF
Line 39: tail -30 tools/build/boost-build.jam
</pre>

## Step 10
Update source code file path in ~src/openvpn3/lz4/build-lz4
<pre>
Line 63: cd $LZ4_VERSION/lib
</pre>

## Step 11
Install cmake
<pre>
brew install cmake
</pre>

## Step 12
Build the dependencies:
<pre>
OSX_ONLY=1 $O3/scripts/mac/build-all
</pre>

## Step 13
Now build the OpenVPN 3 client executable:
<pre>
cd $O3
. vars-osx64
. setpath
cd test/ovpncli
GCC_EXTRA="-stdlib=libc++" STRIP=1 PSSL=1 MINI=1 SNAP=1 LZ4=1 build cli
</pre>

This will build the OpenVPN 3 client library with a small client wrapper (cli).  It will also statically link in all external dependencies (Boost, PolarSSL, libminicrypto (derived from OpenSSL), LZ4, and Snappy), so "cli" may be distributed to other Macs and will run as a standalone executable. It's best to build OpenVPN 3 in C++11 mode to allow for the use of optimized move constructors. But keep in mind that OpenVPN 3 strictly follows the C++ 2003 standard (not C++ 2011), so building with -std=c++11 is optional for optimization purposes.

These build scripts will create a "fat" Mac OS X executable with support for both x86_x64 and i386 architectures, with a minimum deployment target of 10.6.x. The Mac OS X tuntap driver is not required, as OpenVPN 3 can use the integrated utun interface if available.


## Enjoy it
### To view the client wrapper options:
<pre>
./cli -h
</pre>

### To connect:
<pre>
  ./cli client.ovpn
</pre>
