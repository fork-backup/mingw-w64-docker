FROM debian:jessie

RUN \
	useradd -ms /bin/bash builder && \
	apt-get update && \
	apt-get -y dist-upgrade && \
	apt-get install -y wget make bzip2 gcc texinfo binutils-mingw-w64-i686 mingw-w64-i686-dev gcc-mingw-w64-i686 g++-mingw-w64-i686 gfortran-mingw-w64-i686 mingw-w64

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
	$SRC/binutils-2.25/configure --prefix=$DEST --with-sysroot=$DEST --host=i686-w64-mingw32 --target=i686-w64-mingw32 --enable-targets=i686-w64-mingw32,x86_64-w64-mingw32 && \
	make && \
	make install
