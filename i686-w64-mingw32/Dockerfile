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
	g++-mingw-w64-i686 gfortran-mingw-w64-i686 mingw-w64 m4

USER builder

ENV HOST i686-w64-mingw32
ENV BUILD x86_64-w64-mingw32
ENV TARGET i686-w64-mingw32
ENV BUILDROOT /home/builder/mingw
ENV SRC $BUILDROOT/sources
ENV DEST $BUILDROOT/dest
ENV PREFIX $DEST/$TARGET

RUN \
	mkdir -p $SRC && \
	mkdir -p $DEST

RUN \
	cd $SRC && \
	wget http://www.mpfr.org/mpfr-current/mpfr-3.1.2.tar.bz2 && \
	tar xvjf mpfr-3.1.2.tar.bz2 && \
	wget http://isl.gforge.inria.fr/isl-0.12.2.tar.bz2 && \
	tar xvjf isl-0.12.2.tar.bz2 && \
	wget http://gmplib.org/download/gmp/gmp-6.0.0a.tar.bz2 && \
	tar xvjf gmp-6.0.0a.tar.bz2 && \
	wget http://ftp.gnu.org/gnu/binutils/binutils-2.25.tar.bz2 && \
	tar xjf binutils-2.25.tar.bz2 && \
	wget http://downloads.sourceforge.net/project/mingw-w64/mingw-w64/mingw-w64-release/mingw-w64-v3.1.0.tar.bz2 && \
	tar xjf mingw-w64-v3.1.0.tar.bz2 && \
	wget http://ftp.gnu.org/gnu/gcc/gcc-4.9.2/gcc-4.9.2.tar.bz2 && \
	tar xjf gcc-4.9.2.tar.bz2

RUN \
	mkdir -p $BUILDROOT/binutils && \
	cd $BUILDROOT/binutils && \
	$SRC/binutils-2.25/configure --prefix=$DEST --with-sysroot=$DEST --host=$HOST --target=$TARGET --disable-multilib && \
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
	$SRC/mingw-w64-v3.1.0/mingw-w64-headers/configure --prefix=$PREFIX --host=$HOST --build=$BUILD && \
	make && \
	make install

RUN \
	ln -s $PREFIX $DEST/mingw && \
	ln -s $PREFIX/lib $PREFIX/lib32

RUN \
	mkdir -p $BUILDROOT/gmp && \
	cd $BUILDROOT/gmp && \
	$SRC/gmp-6.0.0/configure --host=$HOST --build=$BUILD --prefix=$PREFIX --disable-shared --enable-static --enable-cxx && \
	make && \
	make install

RUN \
	mkdir -p $BUILDROOT/isl && \
	cd $BUILDROOT/isl && \
	CPPFLAGS=-I$PREFIX/include LDFLAGS=-L$PREFIX/lib $SRC/isl-0.12.2/configure --host=$HOST --build=$BUILD --prefix=$PREFIX --disable-shared --enable-static --with-gmp-prefix=$BUILDROOT && \
	make && \
	make install

RUN \
	mkdir -p $BUILDROOT/mpfr && \
	cd $BUILDROOT/mpfr && \
	CPPFLAGS="-I$SRC/gcc-4.9.2/gcc/ginclude -I$PREFIX/include" LDFLAGS="-L$PREFIX/lib" $SRC/mpfr-3.1.2/configure --host=$HOST --build=$BUILD --prefix=$PREFIX --disable-shared --enable-static --with-gmp=$BUILDROOT && \
	make && \
	make install

RUN \
	mkdir -p $BUILDROOT/gcc && \
	cd $BUILDROOT/gcc && \
	$SRC/gcc-4.9.2/configure --host=$HOST --build=$BUILD --target=$TARGET \
	  --prefix=$PREFIX --with-sysroot=$DEST --disable-multilib --enable-languages=c,c++,fortran \
	  --with-gmp=$PREFIX \
	  --disable-shared --enable-static --enable-plugins \
	  --enable-sjlj-exceptions --disable-dw2-exceptions --enable-threads=win32 \
	  --disable-cloog-version-check --enable-cloog-backend=isl --disable-isl-version-check \
	  --enable-libgomp --disable-libstdcxx-pch --enable-fully-dynamic-string --enable-libstdcxx-time \
	  --disable-nls --disable-werror --enable-checking=release \
	  --with-gnu-as --with-gnu-ld \
	  CFLAGS="-I$SRC/gcc-4.9.2/gcc/ginclude -I$PREFIX/include -I/usr/x86_64-w64-mingw32" \
	  LDFLAGS="-L$PREFIX/lib -static" && \
	make all-gcc && \
	make install-gcc
