[![Go Reference](https://pkg.go.dev/badge/github.com/dnjp/zync.svg)](https://pkg.go.dev/github.com/dnjp/zync)

# Zync

Zync is a utility for backing up your files and folders to [IPFS](https://ipfs.io/). In it's current state, Zync is sort of like a primitive version of [Dropbox](https://www.dropbox.com/home) that can be used on the command line. Files and directories managed with `zync` will be continuously backed up to IPFS when they are changed. The list of all of your actively managed files is also backed up to IPFS, providing you a single [CID](https://docs.ipfs.io/concepts/content-addressing/) that can be used to restore all managed files to their original location.

## How does this work?

There are two components that make up Zync - the daemon (`zyncd`) and the command line client (`zync`). Once installed, `zyncd` will be launched by the daemon manager for your operating system - [launchd](https://en.wikipedia.org/wiki/Launchd) on MacOS and [systemd](https://en.wikipedia.org/wiki/Systemd) on Linux. `zync`, the command line client, is the tool you use to add and remove files as well as list those that are already managed.

## Installation

Zync depends on having recent versions of [Go](https://go.dev/learn/) and
[Protobuf](https://grpc.io/docs/protoc-installation/) installed. The linked
guides will take you through the process of getting them installed on your
system.

Once the dependencies are installed, you can install Zync by cloning this
repository and running `sudo make
install`:

```
$ git clone https://github.com/dnjp/zync.git
$ cd ./zync
$ sudo make install
$ which zync
/usr/local/bin/zync
$ which zyncd
/usr/local/bin/zyncd
```

## Usage

After the executables are installed, run `make start` to launch the `zyncd` daemon, allowing `zync` (the command line client) to be able to connect to it:

```
$ make start
./bin/zyncd start
$ cat zyncd.log
2022/01/26 21:40:46 - - - - - - - - - - - - - - -
2022/01/26 21:40:46         zyncd started
2022/01/26 21:40:46 - - - - - - - - - - - - - - -
...
```

Now that `zyncd` has started, you can use `zync` to add files:

```
$ echo 'hello world' >> /tmp/hello
$ cat /tmp/hello
hello world
$ zync add /tmp/hello
file: cid:"QmPQWuv5cwbKWCHkYxEseFawk76gacbP4p2DXkWniY5azS"  absolute_path:"/tmp/hello"
```

Once files are added, you can use `zync ls` to list the files that are currently managed:

```
$ zync ls
file: cid:"QmPQWuv5cwbKWCHkYxEseFawk76gacbP4p2DXkWniY5azS"  absolute_path:"/tmp/hello"
```

Now, let's change the contents of our hello file to see the [CID](https://docs.ipfs.io/concepts/content-addressing/) change in real time:

```
$ echo 'hello there' > /tmp/hello
$ ./bin/zync ls hello
file: cid:"QmSwZjAMN4jkE5rZ1Ewm3hLUVAgVuVeGh1EK3kR2mw1wDo"  absolute_path:"/tmp/hello"
```

Notice that the CID has changed? Files managed with Zync are always kept up to date. Let's try removing that file:

```
$ rm /tmp/hello
$ ./bin/zync ls hello
```

The file has been automatically removed from Zync. You can also remove files from Zync without removing them from your machine:

```
$ zync rm hello
file: cid:"QmSwZjAMN4jkE5rZ1Ewm3hLUVAgVuVeGh1EK3kR2mw1wDo"  absolute_path:"/tmp/hello"
```

Did you catch that? The full path to the file did not need to be supplied to `zync rm` because `add`, `ls`, and `rm` all support accessing files using a [regex](https://github.com/google/re2/wiki/Syntax).
