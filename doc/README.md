Testcoin Core 0.10.4
=====================

Copyright (c) 2009-2013 Bitcoin Developers
Copyright (c) 2011-2014 Testcoin Developers
Setup
---------------------
Bitcoin is a free open source peer-to-peer electronic cash system that is
[Bitcoin Core](http://bitcoin.org/en/download) is the original Bitcoin client and it builds the backbone of the network. However, it downloads and stores the entire history of Bitcoin transactions (which is currently several GBs); depending on the speed of your computer and network connection, the synchronization process can take anywhere from a few hours to a day or more. Thankfully you only have to do this once. If you would like the process to go faster you can [download the blockchain directly](bootstrap.md).

Running
---------------------
The following are some helpful notes on how to run Bitcoin on your native platform.

### Unix

You need the Qt4 run-time libraries to run Testcoin-Qt. On Debian or Ubuntu:

	sudo apt-get install libqtgui4

Unpack the files into a directory and run:

- bin/32/testcoin-qt (GUI, 32-bit)
- bin/32/testcoind (headless, 32-bit)
- bin/64/testcoin-qt (GUI, 64-bit)
- bin/64/testcoind (headless, 64-bit)

See the documentation at the [Testcoin Wiki](http://testcoin.info)
for help and more information.


Other Pages
---------------------
- [Unix Build Notes](build-unix.md)
- [OSX Build Notes](build-osx.md)
- [Windows Build Notes](build-msw.md)
- [Coding Guidelines](coding.md)
- [Release Process](release-process.md)
- [Release Notes](release-notes.md)
- [Multiwallet Qt Development](multiwallet-qt.md)
- [Unit Tests](unit-tests.md)
- [Translation Process](translation_process.md)
