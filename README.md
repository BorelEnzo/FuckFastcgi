# FuckFastCGI

## Description

This is a php script to exploit fastcgi protocol to bypass `open_basedir` and `disable_functions`.

It will help you to bypass strict `disable_functions` to RCE by loading the malicious extension.

## Usage

- Update the config between the markers:
```
// ---- BEGIN CONFIG
$ext_dir_path = '/tmp';
$ext_name = 'hello.so';
// ---- END CONFIG
```
- Also update the socket path:
```
$client = new FCGIClient("127.0.0.1:9000", -1);
```
or (should be a symlink to the real socket)
```
$client = new FCGIClient("unix:///var/run/php/php-fpm.sock", -1);
```

## Changes

Compared to the original exploit, it requires only one PHP to work, based on [this exploit](https://balsn.tw/ctf_writeup/20190323-0ctf_tctf2019quals/#wallbreaker-easy).

### Make it work with an external library

Also, if it runs < PHP8, the file `hello.c` must be slighty modified, line 28: the value `TSRMLS_CC` should be uncommented. Otherise compilation will fail ([https://stackoverflow.com/questions/66194531/how-to-compile-php-module-in-php8-0-that-used-to-use-tsrmls-cc-in-php7-but-is-d](https://stackoverflow.com/questions/66194531/how-to-compile-php-module-in-php8-0-that-used-to-use-tsrmls-cc-in-php7-but-is-d))

I put a Docker config to test it.

First the evil *.so should be built:
* run `docker-compose up --build`
* then open a shell on the container and browse to `/tmp/ffcgi/ext_example/system`
* run the following commands (can be challenging in a real-life scenario, because it requires a machine with the same setup as the victim...):
```
$ cd /tmp/ffcgi
tmp/ffcgi$ phpize //requires php-dev package
tmp/ffcgi$ ./configure
tmp/ffcgi$ make //a 'make clean' may be required in between
tmp/ffcgi$ cp modules/hello.so ../
``` 

Now browse to the PHP exploit file (`localhost:8081`), and pass a `cmd` GET parameter.  If it fails, it would be worth trying to refresh, it happens sometimes when the first request after a reboot contains the `cmd` parameter, dunno exactly y

Try to run `phpinfo` from the main script, it will indicate that all exec-like routines are disabled.

Now, to make it work on a victim server:
* upload the`docker/code/index.php` in a writeable directory
* upload the *.so in the folder `$ext_dir_path`

### Make it work without external library

Why make things even more complicated, when we can keep them simple ? The only_php version works by modifying the MTA path ... No need to upload an evil *.so now
