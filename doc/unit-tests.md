Compiling/running bitcoind unit tests
------------------------------------

bitcoind unit tests are in the `src/test/` directory; they
and tests weren't explicitly disabled.

After configuring, they can be run with 'make check'.

	make -f makefile.unix test_bitcoin  # Replace makefile.unix if you're not on unix
	./test_bitcoin   # Runs the unit tests

To add more bitcoind tests, add `BOOST_AUTO_TEST_CASE` functions to the existing
.cpp files in the test/ directory or add new .cpp files that
implement new BOOST_AUTO_TEST_SUITE sections.
set up to add test/*.cpp to test_bitcoin automatically).

Compiling/running Bitcoin-Qt unit tests
To run the bitcoin-qt tests manually, launch src/qt/test/bitcoin-qt_test

	./bitcoin-qt_test
To add more bitcoin-qt tests, add them to the `src/qt/test/` directory and
the `src/qt/test/test_main.cpp` file.
