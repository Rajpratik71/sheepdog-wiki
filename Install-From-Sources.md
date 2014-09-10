## Compile-time dependencies
* GNU Autotools
* pkg-config
* corosync devel package
* git (when compiling from source repo)
* liburcu
* optional:fuse-devel (for sheepfs)



### Compile or install the Corosync packages
Nearly every modern Linux distribution has x86_64 corosync binaries pre-built available via their repositories.  We recommend you use these packages if they are available on your distribution.

For debian package based systems:
<pre>
$ sudo aptitude install corosync libcorosync-dev
</pre>

For RPM package based systems:
<pre>
$ sudo yum install corosynclib-devel
</pre>

For EL6 (RHEL, CentOS, SL, etc), the provided version of corosync is too old and you must install corosync from source. The following command will remove any existing copies of corosync.

<pre>
$ sudo yum remove corosync corosynclib corosynclib-devel 
</pre>

If your distribution doesn't provide the corosync packages, or you prefer to compile from source:
<pre>
$ git clone git://github.com/corosync/corosync.git
$ cd corosync
$ git checkout -b flatiron origin/flatiron
$ ./autogen.sh
$ ./configure --enable-nss
$ sudo make install
</pre>

See also: [[Corosync Config]]

### Download, build and install QEMU with Sheepdog support

QEMU 0.13 provides built-in support for sheepdog devices.  Some distributions provide pre-built versions of this newer version of QEMU.  If your distribution has an older version of QEMU or you prefer to compile from source, retrieve the latest QEMU and compile:
<pre>
$ git clone git://git.sv.gnu.org/qemu.git
$ cd qemu
$ ./configure
$ sudo make install
$ cd ..
</pre>

### Download, build and install the Sheepdog server and command line tools

If you want to use a stable sheepdog, please download our source archive of stable branch and build it. Stable branches of each version are listed [here] (https://github.com/sheepdog/sheepdog/wiki/Stable-branches#obtain-stable-release-based-on-the-stable-branches). Below is an example of v0.7.6.

<pre>
$ wget -O sheepdog-v0.7.6.tar.gz https://github.com/sheepdog/sheepdog/archive/v0.7.6.tar.gz
$ tar xvzf sheepdog-v0.7.6.tar.gz
$ cd sheepdog-v0.7.6
$ ./autogen.sh
$ ./configure
$ sudo make install
$ cd ..
</pre>

If you want to use every feature of the latest sheepdog, please clone our git repository.

<pre>
$ git clone git://github.com/sheepdog/sheepdog.git
$ cd sheepdog
$ ./autogen.sh
$ ./configure
$ sudo make install
$ cd ..
</pre>

Please note, sheepdog supports a "make rpm" target which will generate an rpm package that can be installed on the local machine.  To use this installation method, use the following instructions:
<pre>
$ git clone git://github.com/collie/sheepdog.git
$ cd sheepdog
$ ./autogen.sh
$ ./configure
$ make rpm
$ sudo rpm -ivh x86_64/sheepdog-0.*
$ cd ..
</pre>