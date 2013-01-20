# Testing OpenStack with Sheepdog
   Here is a quick start guide to test OpenStack with Sheepdog. DevStack is a tool for OpenStack developers, but this document would help you understand how to use Sheepdog with OpenStack until more user-friendly documents come.

   1. Install Ubuntu 12.04 (Precise) or Fedora 16
   2. Install Sheepdog to the appropriate location on your system
   2. Start Sheepdog and format it (See [[Getting Started]])
   3. Download DevStack
<pre>
$ git clone git://github.com/openstack-dev/devstack.git
</pre>
   4. Start the install
<pre>
$ cd devstack; CINDER_DRIVER=sheepdog ./stack.sh
</pre>

See also http://devstack.org/
