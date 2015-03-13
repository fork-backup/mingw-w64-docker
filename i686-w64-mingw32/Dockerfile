# This uses the Debian (or Ubuntu) cross compiler to build a native gcc for win32 with multilib support
#
# Based on: http://sourceforge.net/p/mingw-w64/wiki2/Native%20Win64%20compiler/
#
# Cross compiling notes:
# - The minor version of gcc must match that of our cross compiler (4.9 in this case)
# - Important parameters: http://gcc.gnu.org/onlinedocs/gccint/Configure-Terms.html

FROM debian:jessie

RUN \
	useradd -ms /bin/bash builder && \
	apt-get update && \
	apt-get -y dist-upgrade && \
	apt-get install -y wget make bzip2 gcc texinfo binutils-mingw-w64-i686 mingw-w64-i686-dev gcc-mingw-w64-i686 \
	g++-mingw-w64-i686 gfortran-mingw-w64-i686 mingw-w64

USER builder

ENV BUILDROOT /home/builder/mingw
ENV SRC $BUILDROOT/sources
ENV DEST $BUILDROOT/dest

RUN \
	mkdir -p $SRC && \
	mkdir -p $DEST

RUN \
	cd $SRC && \
	wget http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.bz2 && \
	tar xjf binutils-2.25.tar.bz2 && \
	wget http://downloads.sourceforge.net/project/mingw-w64/mingw-w64/mingw-w64-release/mingw-w64-v3.1.0.tar.bz2 && \
	tar xjf mingw-w64-v3.1.0.tar.bz2 && \
	wget http://ftp.gnu.org/gnu/gcc/gcc-4.9.2/gcc-4.9.2.tar.bz2 && \
	tar xjf gcc-4.9.2.tar.bz2

RUN \
	mkdir -p $BUILDROOT/binutils && \
	cd $BUILDROOT/binutils && \
	$SRC/binutils-2.25/configure --prefix=$DEST --with-sysroot=$DEST --host=i686-w64-mingw32 --target=i686-w64-mingw32 --disable-multilib && \
	make && \
	make install

RUN \
	cd $SRC/gcc-4.9.2 && \
	./contrib/download_prerequisites

RUN \
	mkdir -p $SRC/gcc-4.9.2/gcc/winsup/mingw && \
	ln -s /usr/i686-w64-mingw32/include $SRC/gcc-4.9.2/gcc/winsup/mingw/include

RUN \
	mkdir -p $BUILDROOT/headers && \
	cd $BUILDROOT/headers && \
	$SRC/mingw-w64-v3.1.0/mingw-w64-headers/configure --prefix=$DEST/i686-w64-mingw32 --host=i686-w64-mingw32 --build=x86_64-w64-mingw32 && \
	make && \
	make install

RUN \
	ln -s $DEST/i686-w64-mingw32 $DEST/mingw && \
	ln -s $DEST/i686-w64-mingw32/lib $DEST/i686-w64-mingw32/lib32

RUN \
	mkdir -p $BUILDROOT/gcc && \
	cd $BUILDROOT/gcc && \
	$SRC/gcc-4.9.2/configure --host=i686-w64-mingw32 --build=x86_64-w64-mingw32 --target=i686-w64-mingw32 \
	  --prefix=$DEST/i686-w64-mingw32 --with-sysroot=$DEST --disable-multilib --enable-languages=c,c++,fortran \
	  --disable-shared --enable-sjlj-exceptions --disable-dw2-exceptions --enable-threads=win32 \
	make all-gcc && \
	make install-gcc
