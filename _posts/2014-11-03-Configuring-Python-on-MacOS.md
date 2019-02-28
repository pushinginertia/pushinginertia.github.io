---
layout: post
title: Configuring Python on MacOS
tags: [Configuration]
---

This post goes through the steps to add the necessary modules in Python that I use.

## Install pip

```py
$ sudo easy_install pip
```

## Set up MySQL prerequisites to install the Python MySQLdb module

`mysql_config` must be on the path in order to install the `MySQL-python` module. I install MySQL in `/usr/local` and then create a symlink `/usr/local/mysql` pointing to the actual directory:

```sh
$ ls -l /usr/local
lrwxr-xr-x  mysql -> mysql-5.6.21-osx10.8-x86_64
drwxr-xr-x  mysql-5.6.21-osx10.8-x86_64
```

Add the MySQL bin directory to the path, either in `/etc/profile` or `~/.bash_login`. I threw in a few other MySQL alias customizations I've made, which don't actually have to be included below.

```sh
export set MYSQL_HOME=/usr/local/mysql

# Add the ./bin directory to the path.
export set PATH=$PATH:$MYSQL_HOME/bin

# Aliases make it easier to manually start and stop the MySQL daemon.
alias mysqlstart="/Library/StartupItems/MySQLCOM/MySQLCOM start"
alias mysqlstop="/Library/StartupItems/MySQLCOM/MySQLCOM stop"
```

Check that `mysql-config` is on the path:

```sh
$ which mysql_config
/usr/local/mysql/bin/mysql_config
```

## Install MySQL and peewee ORM modules

```sh
$ sudo pip install peewee MySQL-python
```

Verify that the modules are installed. peewee can be verified just by checking it's installed.

```sh
$ python -c 'help("modules peewee")'
```

Try importing MySQLdb to make sure all of its dependencies are satisfied. If the following error message appears, it means that the MySQL shared library cannot be found by Python.

```py
>>> import MySQLdb
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/MySQLdb/__init__.py", line 19, in <module>
    import _mysql
ImportError: dlopen(/Library/Python/2.7/site-packages/_mysql.so, 2): Library not loaded: libmysqlclient.18.dylib
  Referenced from: /Library/Python/2.7/site-packages/_mysql.so
  Reason: image not found
</module>
```

One solution is to set the `DYLD_LIBRARY_PATH` environment variable, but this seems a bit unnecessary for a library dependency, so a symlink works too. The following command works well.

```sh
$ sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib \
  /usr/lib/libmysqlclient.18.dylib
```

## Install requests and lxml modules

These ones are simple.

```sh
$ sudo pip install requests
$ sudo pip install lxml
```
