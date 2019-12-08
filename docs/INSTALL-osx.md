# Motivation

Get glewlwyd building from source in an IDE on OSX.
In my case, this is for development and debugging purposes.  I would run a production install on Linux.
Also to enable debugging the glewlwyd process from the IDE.

## Three-step approach

1. get required libraries installed (mostly homebrew)
2. get the build working from the command line
3. setup the IDE, in my case CLion

## Step 1: installing required libraries

### libmysqlclient

I downloaded mysql from Oracle and extracted it into /usr/local.

MySQL 8 did not work because the `my_bool` type has disappeared.  I downgraded to `5.7.x`
and all is well.

1. Navigate to Oracle downloads, find the "previous versions" link for MySQL, and get the
tar.gz download for 5.7.x for OSX.

2. Untar it into /usr/local.  It will be in an isolated foler like
   `/usr/local/mysql-5.7.28-macos10.14-x86_64` so it won't clobber other files in /usr/local.
   
   Commands:
   
   ```
   sudo tar -C /usr/local -tzvf ~/Downloads/mysql-5.7.28-macos10.14-x86_64.tar.gz | head -10
   ```
   
   If the output looks right, replace -tzvf with -xzvf and it will extract.
   
   Then link ``/usr/local/mysql` to ``/usr/local/mysql-5.7.28-macos10.14-x86_64``:
   
   ```
   (cd /usr/local; sudo ln -s mysql-5.7.28-macos10.14-x86_64 mysql)
   ```
   
   Now `ls -l /usr/local/mysql/` should look like mysql installation root.

### libcbor

At the moment there is a PR to get libcbor into homebrew, but it was closed.

Run `brew info libcbor`.  If it indicates libcbor is installable, the PR has been merged.
just install it.

`brew` has a handy `pull` to install a formula from a PR.  Assuming
https://github.com/Homebrew/homebrew-core/pull/46071 is still visible, you can install it
with:

```
brew pull 46071
```

### Remaining brew packages

Here is a tree of packages relevant to glewlwyd.  Installing the packages at the roots of
the trees should be sufficieent, that should be simply:

```
brew install cmake \         
             openldap \       
             libconfig\      
             sqlite \         
             libjwt \         
             oath-toolkit \
             libmicrohttpd   
```

```
libcbor                          <---------- dependency, covered above
cmake                            <---------- dependency
openldap                         <---------- dependency
libconfig                        <---------- dependency
sqlite                           <---------- dependency

libjwt                           <---------- dependency
├── jansson                      <---------- dependency
└── openssl@1.1

oath-toolkit                     <---------- dependency
└── libxmlsec1
    ├── gnutls                   <---------- dependency
    │   ├── gmp
    │   ├── libidn2
    │   │   ├── gettext
    │   │   └── libunistring
    │   ├── libtasn1
    │   ├── libunistring
    │   ├── nettle
    │   │   └── gmp
    │   ├── p11-kit
    │   │   └── libffi
    │   └── unbound
    │       ├── libevent
    │       │   └── openssl@1.1
    │       └── openssl@1.1
    ├── libgcrypt
    │   └── libgpg-error
    ├── libxml2
    │   ├── python
    │   │   ├── gdbm
    │   │   ├── openssl@1.1
    │   │   ├── readline
    │   │   ├── sqlite
    │   │   │   └── readline
    │   │   └── xz
    │   └── readline
    └── openssl@1.1

libmicrohttpd                    <---------- dependency
├── gnutls
│   ├── gmp
│   ├── libidn2
│   │   ├── gettext
│   │   └── libunistring
│   ├── libtasn1
│   ├── libunistring
│   ├── nettle
│   │   └── gmp
│   ├── p11-kit
│   │   └── libffi
│   └── unbound
│       ├── libevent
│       │   └── openssl@1.1
│       └── openssl@1.1
└── libgcrypt
    └── libgpg-error
```

## Step 2: get the build process running

I have an gist called [osx-build-shell.sh](https://gist.github.com/etuttle/b520fd2769410b70aed14a6ba8846a98).
Put it on your path, cd into the glewlwyd source and run it.

`osx-build-shell.sh` starts a new shell with ENV setup for cmake to build using clang.  It also sets `$cmake_opts` to the
-D options required to tell cmake to use the homebrew libraries.

### Usage on the command line

```
$ mkdir build
$ cd build
$ cmake $cmake_opts ..
$ make
```

### Usage in an IDE

Find the configuration for CMake in your IDE (in Clion: Preferences -> Build, Execution, Deployment -> CMake).

* Set the environment variables from those spit out by `osx-build-shell.sh`
* Set the CMake options from $cmake_opts.  This helped in my shell: `echo $cmake_options | pbcopy`, then pastee into Clion. 

Build away!