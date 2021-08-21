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

- macOS:

```
brew install automake berkeley-db4 libtool boost miniupnpc pkg-config python qt libevent qrencode sqlite ccache
```

### 2. Download the source code

```
git clone https://github.com/bitcoin/bitcoin.git
```

### 3. Install Berkley DB

Currently the Berkley DB is under use but the Core will make a complete shift to SQlite in nearby future. Will update this document then.

- Enter your local copy of the bitcoin:

```
cd bitcoin
```

- Run:

```
./contrib/install_db4.sh `pwd`
```

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

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Installing TOR

```
brew install tor
```

- To get the Tor Running:

```
tor
```

## Using TOR

### Listening to TorV3

The <code>bitcoin.conf</code> file is used to configure how your node will run. We will simply create the file and add the settings we want our node to have. You can read more about the different features we can add in <code>bitcoin.conf</code> [here](https://github.com/bitcoin/bitcoin/blob/master/doc/tor.md#1-run-bitcoin-core-behind-a-tor-proxy).
Making the file.

```
touch /tmp/21-rc-test/bitcoin.conf
```

Adding the following in your <code>bitcoin.conf</code> should work for now.

```
proxy=127.0.0.1:9050 #If you use Windows, this could possibly be 127.0.0.1:9150 in some cases.
listen=1
bind=127.0.0.1
onlynet=onion

addnode=sxjbhmhob2xasx3vdsy5ke5j5jwecmh3ca4wbs7wf6sg4g2lm3mbszqd.onion:8333
addnode=rp7k2go3s5lyj3fnj6zn62ktarlrsft2ohlsxkyd7v3e3idqyptvread.onion:8333
addnode=d6jwdcoo2l3gbjps6asgg4nhp2gn5oao3wj333o43ssqnjaliehytfad.onion:8333
```

You can test if you are connected to the peers by starting the node.

If you compiled the source code:

```
./src/bitcoind -datadir=/tmp/21-rc-test
```

Binary build:

```
./bin/bitcoind -datadir=/tmp/21-rc-test
```

To check if the peers are connected, open a new terminal and run a query on your running node:

Source Code

```
./src/bitcoin-cli -datadir=/tmp/21-rc-test -netinfo 4
```

Binary Build

```
./bin/bitcoin-cli -datadir=/tmp/21-rc-test -netinfo 4
```

All the peers you are connected to will be listed, and your <code>netinfo</code> might look like:

```
Peer connections sorted by direction and min ping
<-> relay   net  mping   ping send recv  txn  blk  age id address                                                             version
out  full onion    452    452    0    0         0    0  2 rp7k2go3s5lyj3fnj6zn62ktarlrsft2ohlsxkyd7v3e3idqyptvread.onion:8333 70016/Satoshi:21.99.0(Medea)/
out  full onion    792    792    0    0         0    1  0 sxjbhmhob2xasx3vdsy5ke5j5jwecmh3ca4wbs7wf6sg4g2lm3mbszqd.onion:8333 70016/Satoshi:0.21.0/
out  full onion    795    795    0    0         0    0  9 knbmm3arciehs6d7.onion:8333                                         70015/Satoshi:0.20.0/
out  full onion    987    987    0    0         0    0  4 paot7ercftbiyb5o.onion:8333                                         70015/Satoshi:0.18.1/
out  full onion   1090   1090    0    0         0    1  1 js5qbirosytw42jg.onion:8333                                         70015/Satoshi:0.20.1/
out  full onion   1361   1361   20   17              0  6 4le6zxaxgstmk2w7.onion:8333                                         70015/Satoshi:0.20.1/
out  full onion   1834   1834    1    1         0    0  7 thcyj2dhddbe45z6.onion:8333                                         70015/Satoshi:0.20.0/
out  full onion   5230   5230    4    0              0  8 bvkbr4qudict6pv7.onion:8333                                         70015/Satoshi:0.20.0/
                    ms     ms  sec  sec  min  min  min

        ipv4    ipv6   onion   total  block-relay
in         0       0       0       0       0
out        0       0       8       8       0
total      0       0       8       8       0

Local addresses: n/a
```

If you have <code>rp7k2go3s5lyj3fnj6zn62ktarlrsft2ohlsxkyd7v3e3idqyptvread</code>, <code>sxjbhmhob2xasx3vdsy5ke5j5jwecmh3ca4wbs7wf6sg4g2lm3mbszqd</code>, or <code>d6jwdcoo2l3gbjps6asgg4nhp2gn5oao3wj333o43ssqnjaliehytfad</code> in the response, you've successfully connected to a Tor v3 node!

The connection can also be made using the GUI of Bitcoin core, the instructions to that are as follows.

For Source Build:

```
./src/qt/bitcoin-qt -datadir=/tmp/21-rc-test
```

For Binary Build:

```
./bin/bitcoin-qt -datadir=/tmp/21-rc-test
```
