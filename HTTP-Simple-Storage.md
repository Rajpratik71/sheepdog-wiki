Sheepdog HTTP simple storage provides simple object store/retrieve service via RESTful API, similar to Openstack Swift or Amazone S3. Our HTTP API only support GET/PUT/DELETE/HEAD operation and we implement a subset of Openstack Swift API for now and plan to be Amazone S3 compatible in the future.

Users can think of HTTP simple storage as a 3 layer of traditional UNIX file system, with top two layers are directory only (map to user account and container) and the last layer store user files(we call it object), which are fixed in size. To reference these files, you need specify URI(such as http://192.168.1.2/v1/user/container/myfile) instead of file pathname.

# Usage
 Enable compile support

     $./configure --enable-http ...

 Sheepdog make use of fastcgi to handle HTTP requests, it means you need to set a http proxy server at first. Nearly all of web server support fastcgi. In this example, I'll take nginx as example. Create a nginx.conf as in [[nginx.conf]], and then

     $ sudo nginx -c /path/to/nginx.conf

 Enable http service at startup (run 'sheep -r' for detailed help message)
 
     $ sheep -r swift ... # we have 's3' and 'swift' as options, but currently only swift is working

Note, currently only local driver and zookeeper(you need zookeeper 1.4.5 or plus) support http service.
# Play with httpie or curl
I'll take httpie as client to get/put user objects. Firstly install httpie

      $ pip install --upgrade httpie

Suppose you have set up a sheepdog cluster, then

<pre>
$ http PUT http://localhost/v1/mandy # create a user account named 'mandy'
$ http PUT http://localhost/v1/mandy/books # create a book container for 'mandy'
$ http PUT http://localhost/v1/mandy/movies # create a movie container for 'mandy'
$ http GET http://localhost/v1/mandy # to see what containers 'mandy' has
$ http PUT http://localhost/v1/mandy/books/yourbookname &lt; yourbook # upload your book
$ http PUT http://localhost/v1/mandy/movies/yourmoviename &lt; yourmovie
$ http HEAD http://localhost/v1/mandy/movies/ # Get the statistics of movie container
$ http GET http://localhost/v1/mandy/books/yourbookname &gt; yourbook # Download your book
$ http DELETE http://localhost/v1/mandy/books/yourbookname # delete the book in the container
</pre>

# Features
* there is no limit on the object size, we support object size ranging from 1 bytes to 16PB
* one container support as many as 4 billion objects ,one user support 4 billion containers
* support concurrent upload/download/create/delete different objects from different clients

# Client Library
You can use any http library such as libcurl to get the service from HTTP simple storage.

# TODO
Need more inputs from users.