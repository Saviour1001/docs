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

## Setting up the basics first

<code>bitcoin.conf</code> file contains the configurations/settings on which the Bitcoin Core should run upon. The default locations of this file can be found via [this](https://github.com/bitcoin/bitcoin/blob/master/doc/bitcoin-conf.md#configuration-file-path). Currently we will setup the file at <code>/home/.bitcoin</code>

```
export DATA_DIR=$HOME/.bitcoin
mkdir $DATA_DIR
touch $DATA_DIR/bitcoin.conf
```

Next, specify the following paths. For source compiled, start from the root of your release candidate directory and run:

```
export BINARY_PATH=$(pwd)/src
export QT_PATH=$(pwd)/src/qt
```

For the downloaded binary, start from the root of the downloaded release candidate (cd ~/bitcoin-22.0rc2, for example) and run:

```
export BINARY_PATH=$(pwd)/bin
export QT_PATH=$BINARY_PATH
```

Here are some basic commands to play with your Bitcoin Node.

**Start Node**

```
$BINARY_PATH/bitcoind -datadir=$DATA_DIR -daemon
```

**Stop Node**

```
$BINARY_PATH/bitcoin-cli -datadir=$DATA_DIR stop
```

**Wipe and recreate the directory**

```
rm -r $DATA_DIR
mkdir $DATA_DIR
```

### Tor configurations

<code>torrc</code> file is the file used for configuration of the Tor settings. The file is by default located at
<code>/home/linuxbrew/.linuxbrew/etc/tor/torrc, or $HOME/.torrc if that
file is not found</code> for Linux distributions. You can find the location of your default file using the command.

```
man tor
```

You can even make a new file in a new directory and point the path towards it. Use the following command and this [FAQ](https://2019.www.torproject.org/docs/faq#torrc) to understand how to do it.

```
tor -f <INSERT PATH TO THE FILE>/torrc
```

You may need to set up the Tor Control Port. On Linux distributions there may be some or all of the following settings in /etc/tor/torrc, generally commented out by default (if not, add them):

```
ControlPort 9051
CookieAuthentication 1
CookieAuthFileGroupReadable 1
```

Add or uncomment those, save, and restart Tor (usually systemctl restart tor or sudo systemctl restart tor on most systemd-based systems, including recent Debian and Ubuntu, or just restart the computer).

On some systems (such as Arch Linux), you may also need to add the following line:

```
DataDirectoryGroupReadable 1
```

## Using Tor

To see verbose Tor information in the bitcoind debug log, pass -debug=tor.

### Listening to TorV3 Nodes

The <code>bitcoin.conf</code> file is used to configure how your node will run. We will simply create the file and add the settings we want our node to have. You can read more about the different features we can add in <code>bitcoin.conf</code> [here](https://github.com/bitcoin/bitcoin/blob/master/doc/tor.md#1-run-bitcoin-core-behind-a-tor-proxy).
Making the file.

```
touch $HOME/.bitcoin/bitcoin.conf
```

Adding the following in your <code>bitcoin.conf</code> should work.

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
./src/bitcoind -datadir=$DATA_DIR
```

Binary build:

```
./bin/bitcoind -datadir=$DATA_DIR
```

To check if the peers are connected, open a new terminal and run a query on your running node:

Source Code

```
./src/bitcoin-cli -datadir=$DATA_DIR -netinfo 4
```

Binary Build

```
./bin/bitcoin-cli -datadir=$DATA_DIR -netinfo 4
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
./src/qt/bitcoin-qt -datadir=$DATA_DIR
```

For Binary Build:

```
./bin/bitcoin-qt -datadir=$DATA_DIR
```

Congrats! :tada: You got your Bitcoin Core running on Tor and also got connected to other nodes. Let the chain sync up for now !

## Resources

These are the resources which acted as a source for this guide, you can use them for further digging and studying more.

- [Tor.md](https://github.com/bitcoin/bitcoin/blob/master/doc/tor.md)
- [Bitcoin Core 0.21 Testing Guide](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/0.21-Release-Candidate-Testing-Guide)
- [Jon Attack's Blog on How to compile Bitcoin Core](https://jonatack.github.io/articles/how-to-compile-bitcoin-core-and-run-the-tests)
- [Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/questions/70069/how-can-i-setup-bitcoin-to-be-anonymous-with-tor)
- [Bitcoin Wiki](https://en.bitcoin.it/wiki/Setting_up_a_Tor_hidden_service)
