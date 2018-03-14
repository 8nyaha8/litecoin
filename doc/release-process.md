Release Process
====================

###update (commit) version in sources

	contrib/verifysfbinaries/verify.sh
	doc/README*
	share/setup.nsi
	src/clientversion.h (change CLIENT_VERSION_IS_RELEASE to true)

###tag version in git

	git tag -a v0.8.0

###write release notes. git shortlog helps a lot, for example:

	git shortlog --no-merges v(current version, e.g. 0.7.2)..v(new version, e.g. 0.8.0)

* * *

###update Gitian

 In order to take advantage of the new caching features in Gitian, be sure to update to a recent version (e9741525c or higher is recommended)

###perform Gitian builds

 From a directory containing the bitcoin source, gitian-builder and gitian.sigs
  
    export SIGNER=(your Gitian key, ie wtogami, coblee, etc)
	export VERSION=0.8.0
	pushd ./testcoin
	git checkout v${VERSION}
	popd
	pushd ./gitian-builder

###fetch and build inputs: (first time, or when dependency versions change)

	mkdir -p inputs
	wget 'http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.6.tar.gz' -O miniupnpc-1.6.tar.gz
	wget 'http://www.openssl.org/source/openssl-1.0.1c.tar.gz'

 https://developer.apple.com/downloads/download.action?path=Developer_Tools/xcode_6.1.1/xcode_6.1.1.dmg
	wget 'http://zlib.net/zlib-1.2.6.tar.gz'
 Using a Mac, create a tarball for the 10.9 SDK and copy it to the inputs directory:
	wget 'ftp://ftp.simplesystems.org/pub/libpng/png/src/libpng-1.5.9.tar.gz'
	tar -C /Volumes/Xcode/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ -czf MacOSX10.9.sdk.tar.gz MacOSX10.9.sdk
	wget 'http://fukuchi.org/works/qrencode/qrencode-3.2.0.tar.bz2'
###Optional: Seed the Gitian sources cache
	wget 'http://downloads.sourceforge.net/project/boost/boost/1.50.0/boost_1_50_0.tar.bz2'
  By default, Gitian will fetch source files as needed. For offline builds, they can be fetched ahead of time:
	wget 'http://releases.qt-project.org/qt4/source/qt-everywhere-opensource-src-4.8.3.tar.gz'
	make -C ../testcoin/depends download SOURCES_PATH=`pwd`/cache/common

  Only missing files will be fetched, so this is safe to re-run for each build.
	./bin/gbuild ../bitcoin/contrib/gitian-descriptors/boost-win32.yml
###Build Testcoin Core for Linux, Windows, and OS X:
	mv build/out/boost-win32-1.50.0-gitian2.zip inputs/
	./bin/gbuild ../bitcoin/contrib/gitian-descriptors/qt-win32.yml
	mv build/out/qt-win32-4.8.3-gitian-r1.zip inputs/
	./bin/gbuild ../bitcoin/contrib/gitian-descriptors/deps-win32.yml
	mv build/out/bitcoin-deps-0.0.5.zip inputs/
 Build bitcoind and bitcoin-qt on Linux32, Linux64, and Win32:
	./bin/gbuild --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian.yml
	./bin/gsign --signer $SIGNER --release ${VERSION} --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian.yml
	mv build/out/testcoin-*.zip build/out/testcoin-*.exe ../
	zip -r bitcoin-${VERSION}-linux-gitian.zip *
	mv bitcoin-${VERSION}-linux-gitian.zip ../../
	./bin/gbuild --commit bitcoin=v${VERSION} ../bitcoin/contrib/gitian-descriptors/gitian-win32.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win32 --destination ../gitian.sigs/ ../bitcoin/contrib/gitian-descriptors/gitian-win32.yml
	mv build/out/testcoin-*-unsigned.tar.gz inputs/testcoin-osx-unsigned.tar.gz
	zip -r bitcoin-${VERSION}-win32-gitian.zip *
	mv bitcoin-${VERSION}-win32-gitian.zip ../../
	popd
  Build output expected:

  1. linux 32-bit and 64-bit binaries + source (bitcoin-${VERSION}-linux-gitian.zip)
  2. windows 32-bit binary, installer + source (bitcoin-${VERSION}-win32-gitian.zip)
	unzip bitcoin-${VERSION}-linux-gitian.zip -d bitcoin-${VERSION}-linux
	tar czvf bitcoin-${VERSION}-linux.tar.gz bitcoin-${VERSION}-linux
	rm -rf bitcoin-${VERSION}-linux
	unzip bitcoin-${VERSION}-win32-gitian.zip -d bitcoin-${VERSION}-win32
	mv bitcoin-${VERSION}-win32/bitcoin-*-setup.exe .
	zip -r bitcoin-${VERSION}-win32.zip bitcoin-${VERSION}-win32
	rm -rf bitcoin-${VERSION}-win32
  5. Gitian signatures (in gitian.sigs/${VERSION}-<linux|win|osx-unsigned>/(your Gitian key)/

  OSX binaries are created by Gavin Andresen on a 32-bit, OSX 10.6 machine.
  Testcoin 0.8.x is built with MacPorts.  0.9.x will be Homebrew only.
	qmake RELEASE=1 USE_UPNP=1 USE_QRCODE=1 bitcoin-qt.pro
	python2.7 contrib/macdeploy/macdeployqtplus Bitcoin-Qt.app -add-qt-tr $T -dmg -fancy contrib/macdeploy/fancy.plist
 Build output expected: Bitcoin-Qt.dmg
###Next steps:

* update bitcoin.org version
Commit your signature to gitian.sigs:

	pushd gitian.sigs
	git add ${VERSION}-linux/${SIGNER}
	git add ${VERSION}-win/${SIGNER}
	git add ${VERSION}-osx-unsigned/${SIGNER}
	git commit -a
	git push  # Assuming you can push to the gitian.sigs tree
	popd

  Wait for OS X detached signature:
### After 3 or more people have gitian-built, repackage gitian-signed zips:
	He will then upload a detached signature to be combined with the unsigned app to create a signed binary.
  Create the signed OS X binary:
	mkdir gitian
	# Fetch the signature as instructed by Warren/Coblee
	cp signature.tar.gz inputs/
	cp ../bitcoin/contrib/gitian-downloader/*.pgp ./gitian/
	 cp ../gitian.sigs/${VERSION}/${signer}/bitcoin-build.assert.sig ./gitian/${signer}-build.assert.sig
	cp bitcoin-${VERSION}-linux-gitian.zip ../
	pushd bitcoin-${VERSION}-win32-gitian
	mkdir gitian
	cp ../bitcoin/contrib/gitian-downloader/*.pgp ./gitian/
	for signer in $(ls ../gitian.sigs/${VERSION}-win32/); do
	 cp ../gitian.sigs/${VERSION}-win32/${signer}/bitcoin-build.assert ./gitian/${signer}-build.assert
-------------------------------------------------------------------------

### After 3 or more people have gitian-built and their results match:

- Perform code-signing.

    - Code-sign Windows -setup.exe (in a Windows virtual machine using signtool)

  Note: only Warren/Coblee has the code-signing keys currently.

- Create `SHA256SUMS.asc` for the builds, and GPG-sign it:
```bash
sha256sum * > SHA256SUMS
gpg --digest-algo sha256 --clearsign SHA256SUMS # outputs SHA256SUMS.asc
rm SHA256SUMS
```
(the digest algorithm is forced to sha256 to avoid confusion of the `Hash:` header that GPG adds with the SHA256 used for the files)

- Upload gitian zips to SourceForge

- Announce the release:

  - Release sticky on testcointalk: https://testcointalk.org/index.php?board=1.0

  - testcoin-development mailing list

  - Update title of #testcoin on Freenode IRC

  - Optionally reddit /r/testcoin, ... but this will usually sort out itself

- Add release notes for the new version to the directory `doc/release-notes` in git master

