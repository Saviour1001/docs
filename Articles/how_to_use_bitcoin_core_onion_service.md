# How to run Bitcoin Core Onion service

This is a quick summary about how to get the Bitcoin Core onion service up and running. The purpose of this summary is to list all the required steps to get the Onion service running in one place.

## Preparation

## 1. Grab the latest release of the Bitcoin Core

You can either download the binary from [here](https://bitcoincore.org/en/download/), or compile the source code of the main branch of the repository.

### Steps to compile Bitcoin Core from source code

All steps mentioned below need to be run from your terminal.

### 1. Get the dependancies

- Linux:

```
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libssl-dev libevent-dev libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-test-dev libboost-thread-dev libminiupnpc-dev libzmq3-dev libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler git libsqlite3-dev ccache
```

- macOS: <code>brew install automake berkeley-db4 libtool boost miniupnpc pkg-config python qt libevent qrencode sqlite ccache</code>

### 2. Download the source code

- git clone https://github.com/bitcoin/bitcoin.git

### 3. Install Berkley DB

Currently the Berkley DB is under use but the Core will make a complete shift to SQlite in nearby future. Will update this document then.

- Enter your local copy of the bitcoin: <code>cd bitcoin</code>
- Run: <code>./contrib/install_db4.sh `pwd`</code>
- Take note of the instructions displayed in the terminal at the end of the BDB installation
  <code> db4 build complete.
  When compiling bitcoind, run `./configure` in the following way:
  export BDB_PREFIX='<PATH-TO>/db4'
  ./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include" ...</code>

### 4. Compile Bitcoin from source.

- <code>export BDB_PREFIX='<PATH-TO>/db4'</code> (you can use the output from the BDB build above).
- <code>./autogen.sh</code>
- <code>./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include"</code> if using BDB 4.8, otherwise <code>./configure --with-incompatible-bdb
  </code>
- <code>make</code> or if you have multiple CPU cores, you can use all of them to reduce compile time.
- <code>make -j "$(($(nproc)+1))"</code> on Linux
- <code>make -j "$(($(sysctl -n hw.physicalcpu)+1))"</code> on macOS

## 2. Installing TOR

The easiest way to install tor is to use [homebrew](https://brew.sh/).

- To install homebrew

  <code>/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"</code>

- Installing TOR

  <code>brew install tor</code>

- To get the Tor Running:

  <code>tor</code>
