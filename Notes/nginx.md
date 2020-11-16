[TOC]

# nginx

nginx is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server. 



# Features

Read more about the features here: http://nginx.org/en/

## Basic HTTP Server Features

* Serving static and index files
* Accelerated reverse proxying with caching
* Load balancing and fault tolerance
* Accelerated support with caching of FastCGI
* Modular architecture
* SSL and TLS SNI support
* Support for HTTP/2 with weighted and dependency-based prioritization



# Important File Locations

## Configuration File

```bash
/usr/local/nginx/conf/nginx.conf
/etc/nginx/nginx.conf
/usr/local/etc/nginx/nginx.conf
```

## Error Logs

In case something does not work as expected, more information can be found in files *access.log* or *error.log* located in ...

```
/usr/local/nginx/logs/
/var/log/nginx/
```

## Process ID

The process ID of the nginx master process is written, by default, to the *nginx.pid* file located in ...

```bash
/usr/local/nginx/logs/nginx.pid
/var/run/nginx.pid
```

For getting a list of all running nginx processes (master and workers), the `ps` command may be used in the following way ...

```bash
ps -ax | grep nginx
```



# References

* Beginner's guide: http://nginx.org/en/docs/beginners_guide.html
* Documentation: http://nginx.org/en/docs/
  * Directives: http://nginx.org/en/docs/dirindex.html
  * Variables: http://nginx.org/en/docs/varindex.html

