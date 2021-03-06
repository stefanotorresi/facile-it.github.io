---
authors: ["lorenzo"]
comments: true
date: "2016-01-11"
draft: false
image: ""
menu: ""
share: true
categories: [English, Fuse, Filesystem, C, CMake]
title: "Write a filesystem with FUSE"
type: "post"
aliases:
  - "/write-filesystem-fuse"
---
During the past year I experimented a lot with file systems in Userspace using FUSE, I wrote this post to share my thoughts about what I did and to give you a starting point to do something by yourself.

# Introduction

A filesystem is that piece of software that is in charge of storing, organizing and generally taking care of data represented as files and directories.
If you are using a device to read this post you are probably using at least one filesystem at the moment.

Implementing a filesystem is not an easy task to accomplish and requires that a few parts of it have to be written at kernel level, fortunately that's not our case since we are not writing a real on-disk filesystem, but rather we want to write something on top of it to solve a specific problem.

The most common tool to do that in user space is precisely **FUSE, Filesystem in USErspace**.

There are a lot of filesystem examples built on top of FUSE out there that cover the most different use cases like:

- [GlusterFS](https://www.gluster.org/): scalable network filesystem
- [SSHFS](https://github.com/libfuse/sshfs): allows mounting a remote filesystem over SSH
- [GMailFS](https://sr71.net/projects/gmailfs/): allows to use GMail storage as a filesystem
- [LoggedFS](http://loggedfs.sourceforge.net/): filesystem that logs operations that happens in it


The main **advantages** of FUSE over writing a low level kernel filesystem are:

- Is usable by non-privileged users;
- Clean and easy interface to do FS operations;
- Has bindings in most available programming language;
- No need of advanced kernel development skills;
- Comes with user isolation, more secure;
- Since you are not hacking in kernel space there are a few chances that a crash in your program takes down the entire system;

However there are also a few **disadvantages** of this approach:

- The target system need libfuse installed;
- Slower than low level implementations;
- Not the best option if you need multiple users to access your filesystem;


Here's a flow-chart diagram showing how FUSE works, source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:FUSE_structure.svg)
![A flow-chart diagram showing how FUSE works](/images/write-filesystem-fuse/FUSE_structure.svg)

# Getting started with FUSE

This section of the post is designed to introduce you on how to practically get your hands dirt with FUSE. Anyway you can understand what's going on whether you execute the code or not.

## Build dependencies
From now you'll need a few build dependencies and a text editor or an IDE to build and edit the code and do your experiments.

### Linux

- GCC or Clang
- CMake >= 3
- make
- FUSE 2.6 or later
- FUSE development files

To obtain those dependencies you can issue the following commands (depending on your Linux distribution).

Fedora/CentOS

```bash
yum install gcc fuse fuse-devel make cmake
```

Debian/Ubuntu

```bash
apt-get install gcc fuse libfuse-dev make cmake
```

### Mac OSX

- Command line tools for Xcode (contains Clang and make)
- CMake >= 3
- FUSE >= 2.6 or later
- FUSE development files

You can obtain all the needed dependencies at the following sources:

- [Xcode](https://developer.apple.com/xcode/features/)
- [CMake](https://cmake.org/download/)
- [OSXFuse](https://osxfuse.github.io/)


## FUSE API

The most important thing to be aware of when working with FUSE is its API.
The libfuse library exposes a set of callbacks that you have to implement in order to tell your filesystem how to behave.

The most complete source of documentation on what are the callbacks and their behavior is the `fuse.h` declaration file. You can find an online version [here](https://github.com/libfuse/libfuse/blob/579c3b03f57856e369fd6db2226b77aba63b59ff/include/fuse.h#L102-L577).


## Example project

For the purpose of showing you how simple is the creation of a FUSE filesystem, I wrote this little implementation that, when mounted, only exposes a file named `file` and its content.

You can find the example project on [GitHub](https://github.com/fntlnz/fuse-example).

I think that the best way to start your own implementation is to take an example and start adding your features.

So, as first thing clone the example project:

```
git clone https://github.com/fntlnz/fuse-example.git
```

As you can see the project structure is quite simple:

```
.
├── CMake
│   └── FindFUSE.cmake
├── CMakeLists.txt
└── fuse-example.c
```

### [CMakeLists.txt](https://github.com/fntlnz/fuse-example/blob/master/CMakeLists.txt)

As you may know CMake is a tool used to manage project builds in a cross platform way. The scope of this file is to define what CMake is supposed to do for our project. The `CMake/FindFuse.cmake` is needed in order to tell CMake where to find the FUSE related things while compiling/linking.

### [fuse-example.c](https://github.com/fntlnz/fuse-example/blob/master/fuse-example.c)

Here's where the magic actually happen!

In this example I implemented four of the FUSE API callbacks namely: getattr, open, read, readdir.

#### getattr

The getattr callback is in charge of reading the metadata of a given path, this  callback is always called before any operation made on the filesystem.


```c
static int getattr_callback(const char *path, struct stat *stbuf) {
  memset(stbuf, 0, sizeof(struct stat));

  if (strcmp(path, "/") == 0) {
    stbuf->st_mode = S_IFDIR | 0755;
    stbuf->st_nlink = 2;
    return 0;
  }

  if (strcmp(path, filepath) == 0) {
    stbuf->st_mode = S_IFREG | 0777;
    stbuf->st_nlink = 1;
    stbuf->st_size = strlen(filecontent);
    return 0;
  }

  return -ENOENT;
}
```

What we are doing here is simple: 

- if the value of path equals to root `/`, we declare it as a directory and return.
- if the value of path equals to filepath `/file`, we declare it as a file and explicit its size and then return.
- Otherwise nothing exists at the given path, and we return `-ENOENT`.

As you can see, we are telling FUSE that the current entry is a file or a directory using the `stat` struct.

In general, if the entry is a directory, `st_mode` have to be set to `S_IFDIR` and `st_nlink` to 2, while if it's a file, `st_mode` have to be set to `S_IFREG` (that stands for regular file) and `st_nlink` to 1. Files also require that the `st_size` (the full file size) is specified.

[Here](http://pubs.opengroup.org/onlinepubs/007908799/xsh/sysstat.h.html) you can find more information about `<sys/stat.h>` 

#### open
The open callback is called when the system requests for a file to be opened. Since we don't have real file but only in-memory representations, we are going to implement this callback just because is needed for FUSE to work and therefore return 0.

#### read
This callback is called when FUSE is reading data from an opened file.
It should return exactly the number of bytes requested and fill the second argument `buf` with the content of those bytes.
As done in the getattr callback, here I'm checking if the given path equals to a known one, I copy the `filecontent` into the `buf` and then return the requested number of bytes.


```c
static int read_callback(const char *path, char *buf, size_t size, off_t offset,
    struct fuse_file_info *fi) {

  if (strcmp(path, filepath) == 0) {
    size_t len = strlen(filecontent);
    if (offset >= len) {
      return 0;
    }

    if (offset + size > len) {
      memcpy(buf, filecontent + offset, len - offset);
      return len - offset;
    }

    memcpy(buf, filecontent + offset, size);
    return size;
  }

  return -ENOENT;
}
```

#### readdir
The readdir callback has the task of telling FUSE the exact structure of the accessed directory.
Since at the moment the only available directory is `/`, this function always return its representation, we are doing it by filling `buf` with the two links for the upper directory `..` and current directory `.` and with the only file we have: `file`.

```c
static int readdir_callback(const char *path, void *buf, fuse_fill_dir_t filler,
    off_t offset, struct fuse_file_info *fi) {
  (void) offset;
  (void) fi;

  filler(buf, ".", NULL, 0);
  filler(buf, "..", NULL, 0);

  filler(buf, filename, NULL, 0);

  return 0;
}
```

#### main

Last but not least, the `main` function here is acting as a proxy to the `fuse_main` passing arguments through it and configuring it with the implemented FUSE operation callbacks via the `fuse_example_operations` variable.

```c
static struct fuse_operations fuse_example_operations = {
  .getattr = getattr_callback,
  .open = open_callback,
  .read = read_callback,
  .readdir = readdir_callback,
};

int main(int argc, char *argv[])
{
  return fuse_main(argc, argv, &fuse_example_operations, NULL);
}
```

### Build and run

Do you remember that you installed CMake, make, gcc and libfuse? It's time to use them!

The first tool we are using is CMake to **check dependencies, setup environment and generate Makefiles**.

```
cmake -DCMAKE_BUILD_TYPE=Debug .
```

If you don't want Debug flags and other development related enabled features, just change `Debug` to `Release`

The second tool we are using is `make`, that using the CMake generated **Makefiles** is now able to build our project.

```
make -j
```

The `-j` parts tells make to parallelize the build to all your cores, remove it if you run out of CPU.

Now that everything is ready, if no build error has occurred, we can enjoy our new filesystem!

### Run!

Before doing anything we need a mountpoint, so let's create the directory where the filesystem will be mounted:

```
mkdir /tmp/example
```

and then, mount the filesystem:

```
./bin/fuse-example -d -s -f /tmp/example
```

Now check that it has been mounted:

```
$ ls -la
total 0
drwxr-xr-x.  2 root root   0 Jan  1  1970 .
drwxrwxrwt. 14 root root 320 Jan 10 16:03 ..
-rwxrwxrwx.  1 root root  49 Jan  1  1970 file

```

```
$ mount | grep fuse-example
fuse-example on /tmp/example type fuse.fuse-example (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
```

As you may notice, we mounted the filesystem with three arguments which are:

- **d**: enable debugging
- **s**: run single threaded
- **f**: stay in foreground

You can see the list of all mount options using `-h`.

## Thoughts and notes
- An important thing to notice is that write and read operations by default have a size of 4kb so if your file is, let's say, 399kb you have to deal with the fact that to read it the read callback will be called 100 times with 100 different offset and 99 equals size but one that will have 3kb as size because the file is 399kb and not 400kb so the latest chunk has size 3kb and not 4kb.
- FUSE is more secure than low level kernel development, but security is not free so if you are going to write a network filesystem, for example you may want not to mount it as root.
- By default, accessing the mounted filesystem for other users is not allowed.


## Other resources
- [Fuse bindings in Go](https://github.com/hanwen/go-fuse)
- [Fuse bindings in NodeJS](https://github.com/bcle/fuse4js)
- [Fuse bindings in Python](https://github.com/terencehonles/fusepy)
- [Fuse bindings in Java](https://github.com/EtiennePerot/fuse-jna)
- [Other examples in C](https://github.com/libfuse/libfuse/tree/master/example)


These were my 2 cents, if you are interested in seeing something from me I started a little project on GitHub to create a filesystem that can use online services via FUSE to store data. You can find the project here: [WebFS on GitHub](https://github.com/fntlnz/webfs).

Thanks for reading!
