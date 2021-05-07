# Monero-RaspberryPi4-arm64
Monero-RaspberryPi4-arm64


This information has been create to support newcommers how to install Monero on a RaspberryPi4-arm64 8GB with OS raspios_64.

Basic`s:

Hardware:		RaspberryPi4-arm64 8GB
Sdcard:		bigger than 128 GB
OS:			raspios_arm-64
	Download:	https://downloads.raspberrypi.org/raspios_arm64/images

To create the OS on sdcard you can use Etcher.

	Download: https://www.balena.io/etcher/

After installation it is needed to configure the OS, because by starting of the installation a shell / terminal will be displayed.
Change with the command „su“ to root and hit the enter key, at this moment no root password has been set ( it is blank).

Example:
xxx@raspian: su
root@raspian:

For security reasons the password of root should be changed by the commant „passwd“.

Example:
root@raspian: passwd root

Now it is needed to create an user. This profile is needed to step forward with the installation of Monero. Change the root folder by using the command „su -“ and create the user by command „add user monero“.

Example:
root@raspian: su -
root@raspian: add user monero

Add password for user monero.

Example:
root@raspian: passwd monero

Now it is important to know if you want to use a desktop or the terminal. 
In my case the XFCE desktop has been installed and is working well.
To add a desktop the command „tasksel“ is needed.

Example:
root@raspian: tasksel.

After the installation there are some packgages needed to complete the the basic installation. Therefore use the command „apt install“.

It makes sense to install x11vnc and ssh for a remote control and a firewall ist needed too.
For downloading to a later time the Monero packages  GIT is needed.

Example:
root@raspian: apt install x11vnc ssh ufw git

Now make a reboot and login as user „monero“.

If you want that x11vnc will be started by system start so add the command 

x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/>USERNAMEN/.vnc/passwd -rfbport 5900 -shared

to your start session option ( this depend on the Desktop / Terminal you use where the command must be added ).

In Xfce you can add this simple by click on applications / settings / session and startup / application for autostart . Create here a new line by click on „+ add“  name the app like x11vnc and add in the field command the order. 

x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/>USERNAMEN/.vnc/passwd -rfbport 5900 -shared

After a reboot the x11vnc is starting automatically and can be used for remote control.

Now it is needed to install some dependencies for the later installation of Monero.
Therefore read the readme.md of monero-project
https://github.com/monero-project/monero/blob/master/README.md

Packages can only be installed as root, so change to root by using „ su“

Example:
root@raspian: su 

[1] On Debian/Ubuntu libgtest-dev only includes sources and headers. You must build the library binary manually. This can be done with the following command 
sudo apt-get install libgtest-dev && cd /usr/src/gtest && sudo cmake . && sudo make && sudo mv libg* /usr/lib/ 

[2] libnorm-dev is needed if your zmq library was built with libnorm, and not needed otherwise

Install all dependencies at once on Debian/Ubuntu:

sudo apt update && sudo apt install build-essential cmake pkg-config libssl-dev libzmq3-dev libunbound-dev libsodium-dev libunwind8-dev liblzma-dev libreadline6-dev libldns-dev libexpat1-dev libpgm-dev qttools5-dev-tools libhidapi-dev libusb-1.0-0-dev libprotobuf-dev protobuf-compiler libudev-dev libboost-chrono-dev libboost-date-time-dev libboost-filesystem-dev libboost-locale-dev libboost-program-options-dev libboost-regex-dev libboost-serialization-dev libboost-system-dev libboost-thread-dev ccache doxygen graphviz


If the sudo packeage is not installed on your system, cancle the „sudo“ order in the command. Apt install or apt-get is sufficient here.

Has the installation be finished successfully leave root by „exit“

Example:
root@raspian: exit
monero@raspian: 

Now the dowload of Monero must be started by the command  
git clone --recursive https://github.com/monero-project/monero

Example:
monero@raspian: git clone --recursive https://github.com/monero-project/monero

After the download you have a new folder in the home-directory named „monero“.
Change to the directory „monero“. Here is the „Makefile“.
To use the „Makefile“ under a RaspberryPi4-arm64 it is needed to do some changes in this file.  All lines which are starting with „-D“ must be edit with -DNO_AES=ON .

Example Makefile:

all: release-all

depends:
	cd contrib/depends && $(MAKE) HOST=$(target) && cd ../.. && mkdir -p build/$(target)/release
	cd build/$(target)/release && cmake -DNO_AES=ON -DCMAKE_TOOLCHAIN_FILE=$(CURDIR)/contrib/depends/$(target)/share/toolchain.cmake ../../.. && $(MAKE)

cmake-debug:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D CMAKE_BUILD_TYPE=Debug $(topdir)


Here the Makefile with added code.

##########################################

# Copyright (c) 2014-2020, The Monero Project
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification, are
# permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this list of
#    conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice, this list
#    of conditions and the following disclaimer in the documentation and/or other
#    materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its contributors may be
#    used to endorse or promote products derived from this software without specific
#    prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL
# THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
# STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF
# THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

ANDROID_STANDALONE_TOOLCHAIN_PATH ?= /usr/local/toolchain

dotgit=$(shell ls -d .git/config)
ifneq ($(dotgit), .git/config)
  USE_SINGLE_BUILDDIR=1
endif

subbuilddir:=$(shell echo  `uname | sed -e 's|[:/\\ \(\)]|_|g'`/`git branch | grep '\* ' | cut -f2- -d' '| sed -e 's|[:/\\ \(\)]|_|g'`)
ifeq ($(USE_SINGLE_BUILDDIR),)
  builddir := build/"$(subbuilddir)"
  topdir   := ../../../..
  deldirs  := $(builddir)
else
  builddir := build
  topdir   := ../..
  deldirs  := $(builddir)/debug $(builddir)/release $(builddir)/fuzz
endif

all: release-all

depends:
	cd contrib/depends && $(MAKE) HOST=$(target) && cd ../.. && mkdir -p build/$(target)/release
	cd build/$(target)/release && cmake -DNO_AES=ON -DCMAKE_TOOLCHAIN_FILE=$(CURDIR)/contrib/depends/$(target)/share/toolchain.cmake ../../.. && $(MAKE)

cmake-debug:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D CMAKE_BUILD_TYPE=Debug $(topdir)

debug: cmake-debug
	cd $(builddir)/debug && $(MAKE)

# Temporarily disable some tests:
#  * libwallet_api_tests fail (Issue #895)
debug-test:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON  -D BUILD_TESTS=ON -D CMAKE_BUILD_TYPE=Debug $(topdir) &&  $(MAKE) && $(MAKE) ARGS="-E libwallet_api_tests" test

debug-test-asan:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D SANITIZE=ON -D CMAKE_BUILD_TYPE=Debug $(topdir) &&  $(MAKE) && $(MAKE) ARGS="-E libwallet_api_tests" test

debug-test-trezor:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D TREZOR_DEBUG=ON -D CMAKE_BUILD_TYPE=Debug $(topdir) &&  $(MAKE) && $(MAKE) ARGS="-E libwallet_api_tests" test

debug-all:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D BUILD_SHARED_LIBS=OFF -D CMAKE_BUILD_TYPE=Debug $(topdir) && $(MAKE)

debug-static-all:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D STATIC=ON -D CMAKE_BUILD_TYPE=Debug $(topdir) && $(MAKE)

debug-static-win64:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -G "MSYS Makefiles" -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=Debug -D BUILD_TAG="win-x64" -D CMAKE_TOOLCHAIN_FILE=$(topdir)/cmake/64-bit-toolchain.cmake -D MSYS2_FOLDER=$(shell cd ${MINGW_PREFIX}/.. && pwd -W) $(topdir) && $(MAKE)
 
debug-static-win32:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -G "MSYS Makefiles" -DNO_AES=ON -D STATIC=ON -D ARCH="i686" -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=Debug -D BUILD_TAG="win-x32" -D CMAKE_TOOLCHAIN_FILE=$(topdir)/cmake/32-bit-toolchain.cmake -D MSYS2_FOLDER=$(shell cd ${MINGW_PREFIX}/.. && pwd -W) $(topdir) && $(MAKE)
 
cmake-release:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D CMAKE_BUILD_TYPE=Release $(topdir)

release: cmake-release
	cd $(builddir)/release && $(MAKE)

release-test:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D CMAKE_BUILD_TYPE=release $(topdir) && $(MAKE) && $(MAKE) test

release-all:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D CMAKE_BUILD_TYPE=release $(topdir) && $(MAKE)

release-static:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release $(topdir) && $(MAKE)

coverage:
	mkdir -p $(builddir)/debug
	cd $(builddir)/debug && cmake -DNO_AES=ON -D BUILD_TESTS=ON -D CMAKE_BUILD_TYPE=Debug -D COVERAGE=ON $(topdir) && $(MAKE) && $(MAKE) test

# Targets for specific prebuilt builds which will be advertised for updates by their build tag

release-static-linux-armv6:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D BUILD_TESTS=OFF -D ARCH="armv6zk" -D STATIC=ON -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="linux-armv6" $(topdir) && $(MAKE)

release-static-linux-armv7:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D BUILD_TESTS=OFF -D ARCH="armv7-a" -D STATIC=ON -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="linux-armv7" $(topdir) && $(MAKE)

release-static-android-armv7:
	mkdir -p $(builddir)/release/translations
	cd $(builddir)/release/translations && cmake ../../../translations && $(MAKE)
	cd $(builddir)/release && CC=arm-linux-androideabi-clang CXX=arm-linux-androideabi-clang++ cmake -DNO_AES=ON -D BUILD_TESTS=OFF -D ARCH="armv7-a" -D STATIC=ON -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=release -D ANDROID=true -D INSTALL_VENDORED_LIBUNBOUND=ON -D BUILD_TAG="android-armv7" -D CMAKE_SYSTEM_NAME="Android" -D CMAKE_ANDROID_STANDALONE_TOOLCHAIN="${ANDROID_STANDALONE_TOOLCHAIN_PATH}" -D CMAKE_ANDROID_ARM_MODE=ON -D CMAKE_ANDROID_ARCH_ABI="armeabi-v7a" ../.. && $(MAKE)

release-static-android-armv8:
	mkdir -p $(builddir)/release/translations
	cd $(builddir)/release/translations && cmake ../../../translations && $(MAKE)
	cd $(builddir)/release && CC=aarch64-linux-android-clang CXX=aarch64-linux-android-clang++ cmake -DNO_AES=ON -D BUILD_TESTS=OFF -D ARCH="armv8-a" -D STATIC=ON -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release -D ANDROID=true -D INSTALL_VENDORED_LIBUNBOUND=ON -D BUILD_TAG="android-armv8" -D CMAKE_SYSTEM_NAME="Android" -D CMAKE_ANDROID_STANDALONE_TOOLCHAIN="${ANDROID_STANDALONE_TOOLCHAIN_PATH}" -D CMAKE_ANDROID_ARCH_ABI="arm64-v8a" ../.. && $(MAKE)

release-static-linux-armv8:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D BUILD_TESTS=OFF -D ARCH="armv8-a" -D STATIC=ON -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="linux-armv8" $(topdir) && $(MAKE)

release-static-linux-x86_64:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="linux-x64" $(topdir) && $(MAKE)

release-static-freebsd-x86_64:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="freebsd-x64" $(topdir) && $(MAKE)

release-static-mac-x86_64:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="mac-x64" $(topdir) && $(MAKE)

release-static-linux-i686:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -DNO_AES=ON -D STATIC=ON -D ARCH="i686" -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=release -D BUILD_TAG="linux-x86" $(topdir) && $(MAKE)

release-static-win64:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -G "MSYS Makefiles" -DNO_AES=ON -D STATIC=ON -D ARCH="x86-64" -D BUILD_64=ON -D CMAKE_BUILD_TYPE=Release -D BUILD_TAG="win-x64" -D CMAKE_TOOLCHAIN_FILE=$(topdir)/cmake/64-bit-toolchain.cmake -D MSYS2_FOLDER=$(shell cd ${MINGW_PREFIX}/.. && pwd -W) $(topdir) && $(MAKE)

release-static-win32:
	mkdir -p $(builddir)/release
	cd $(builddir)/release && cmake -G "MSYS Makefiles" -DNO_AES=ON -D STATIC=ON -D ARCH="i686" -D BUILD_64=OFF -D CMAKE_BUILD_TYPE=Release -D BUILD_TAG="win-x32" -D CMAKE_TOOLCHAIN_FILE=$(topdir)/cmake/32-bit-toolchain.cmake -D MSYS2_FOLDER=$(shell cd ${MINGW_PREFIX}/.. && pwd -W) $(topdir) && $(MAKE)

fuzz:
	mkdir -p $(builddir)/fuzz
	cd $(builddir)/fuzz && cmake -DNO_AES=ON -D STATIC=ON -D SANITIZE=ON -D BUILD_TESTS=ON -D USE_LTO=OFF -D CMAKE_C_COMPILER=afl-gcc -D CMAKE_CXX_COMPILER=afl-g++ -D ARCH="x86-64" -D CMAKE_BUILD_TYPE=fuzz -D BUILD_TAG="linux-x64" $(topdir) && $(MAKE)

clean:
	@echo "WARNING: Back-up your wallet if it exists within ./"$(deldirs)"!" ; \
    read -r -p "This will destroy the build directory, continue (y/N)?: " CONTINUE; \
	[ $$CONTINUE = "y" ] || [ $$CONTINUE = "Y" ] || (echo "Exiting."; exit 1;)
	rm -rf $(deldirs)

clean-all:
	@echo "WARNING: Back-up your wallet if it exists within ./build!" ; \
	read -r -p "This will destroy all build directories, continue (y/N)?: " CONTINUE; \
	[ $$CONTINUE = "y" ] || [ $$CONTINUE = "Y" ] || (echo "Exiting."; exit 1;)
	rm -rf ./build

tags:
	ctags -R --sort=1 --c++-kinds=+p --fields=+iaS --extra=+q --language-force=C++ src contrib tests/gtest

.PHONY: all cmake-debug debug debug-test debug-all cmake-release release release-test release-all clean tags

#####################################################

If you have copied the needed Makefile on your PC the file can be transfered by command  „scp“ to your RaspberryPi4-arm64.

Example:
hostdevice@user: scp
home/Desktop/Makefile raspberrypi4-user@IPAddress:/home/raspberrypi4-user/monero/

After the change in the Makefile has been done the order for compiling can be started with the command „make“

Therefore read the readme.md of monero-project
https://github.com/monero-project/monero/blob/master/README.md

In case of RaspberryPi4-arm64 it makes sense to start the make option with – j 4 this means that the hardware will be used in full scope.

Example:
monero@raspian:~/monero$ make -j 4

The configuration will take a while.

In the next step it is needed to change the path in the „bin“ folder by command „cd“ for starting the download of the blockchain.

Example:
monero@raspian:cd monero/build/Linux/master/release/bin

Download of the blockchain can be started with command „ ./monerod “.

Example:
monero@raspian:/home/monero/monero/build/Linux/master/release/bin$ ./monerod

Is the download finished, this can take some hour‘s or day‘s, you can start the wallet by using the command „ ./ monero-wallet-cli „ in a new terminal.

Example new terminal:
monero@raspian:/home/monero/monero/build/Linux/master/release/bin$ ./monero-wallet-cli

Now your Moneronode on RaspberryPi4-arm64 8GB should be running well.

If you want to support me then please make a donation :

48othJAKNWDc45iVavvMM6JXR66bQXCbnaccvNta6mF1gnAuCxC5oUsJM8MZo6VR9XQ7wma7UbzVw27eRuVnzzjLN4p4YSZ
