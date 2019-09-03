#
# A makefile to build MingW-w64 automatically.
#
BINUTILS_VER := 2.32
GCC_VER      := 9.2.0
MINGW_VER    := 6.0.0
PREFIX       := /opt/mingw64-dev
DL_CMD       := curl -L -s

GNU_MIRROR   := https://mirrors.kernel.org/gnu

MINGW_FILE    := mingw-w64-v${MINGW_VER}.tar.bz2
BINUTILS_FILE := binutils-${BINUTILS_VER}.tar.xz
GCC_FILE      := gcc-${GCC_VER}.tar.xz

BINUTILS_URL := ${GNU_MIRROR}/binutils/${BINUTILS_FILE}
GCC_URL      := ${GNU_MIRROR}/gcc/gcc-${GCC_VER}/${GCC_FILE}
MINGW_URL    := https://downloads.sourceforge.net/project/mingw-w64/mingw-w64/mingw-w64-release/mingw-w64-v${MINGW_VER}.tar.bz2?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fmingw-w64%2Ffiles%2Fmingw-w64%2Fmingw-w64-release%2Fmingw-w64-v${MINGW_VER}.tar.bz2%2Fdownload&ts=1567467229

SUDO         := sudo

PATH := /usr/bin:${PREFIX}/bin
export PATH

all:   ;
clean:
	rm -Rf *-build *.log *.exe

all-clean: clean
	rm -Rf binutils-* mingw-* gcc-* wine-*

${MINGW_FILE}.stamp:
	@echo "Downloading MinGW-w64 ${MINGW_VER}..."
	@rm -f ${MINGW_FILE}
	@${DL_CMD} "${MINGW_URL}" -o ${MINGW_FILE}
	@touch $@

${BINUTILS_FILE}.stamp:
	@echo "Downloading GNU Binutils ${BINUTILS_VER}..."
	@rm -f ${BINUTILS_FILE}
	@${DL_CMD} "${BINUTILS_URL}" -o ${BINUTILS_FILE}
	@touch $@

${GCC_FILE}.stamp:
	@echo "Downloading GCC ${GCC_VER}..."
	@rm -f ${GCC_FILE}
	@${DL_CMD} "${GCC_URL}" -o ${GCC_FILE}
	@touch $@

${PREFIX}:
	mkdir -p $@

binutils-${BINUTILS_VER}: ${BINUTILS_FILE}.stamp
	@tar xf ${BINUTILS_FILE}

${PREFIX}/bin/x86_64-w64-mingw32-as: binutils-${BINUTILS_VER}
	@echo "Building and installing binutils ${BINUTILS_VER}..."
	@mkdir -p binutils-build
	@( cd binutils-build && \
		../binutils-${BINUTILS_VER}/configure --prefix=${PREFIX} --with-sysroot=${PREFIX} \
		--target=x86_64-w64-mingw32 --enable-targets=x86_64-w64-mingw32,i686-w64-mingw32 --disable-nls && \
		${MAKE} ${NUM_JOBS} && ${SUDO} ${MAKE} install ) > binutils-build.log 2>&1

mingw-w64-v${MINGW_VER}: ${MINGW_FILE}.stamp
	@tar xf ${MINGW_FILE}

${PREFIX}/x86_64-w64-mingw32/include/windows.h: ${PREFIX}/bin/x86_64-w64-mingw32-as mingw-w64-v${MINGW_VER}
	@echo "Installing headers..."
	@mkdir -p mingw-build
	@( cd mingw-build && \
		../mingw-w64-v${MINGW_VER}/configure --prefix=${PREFIX}/x86_64-w64-mingw32 --build=x86_64-pc-linux-gnu --host=x86_64-w64-mingw32 --without-crt && \
		${SUDO} ${MAKE} install ) > headers-build.log 2>&1

${PREFIX}/mingw: ${PREFIX}/x86_64-w64-mingw32/include/windows.h
	@cd ${PREFIX} && ${SUDO} ln -s x86_64-w64-mingw32 mingw

gcc-${GCC_VER}: ${GCC_FILE}.stamp
	@tar xf ${GCC_FILE}

${PREFIX}/bin/x86_64-w64-mingw32-gcc: gcc-${GCC_VER} ${PREFIX}/mingw
	@echo "Building and installing GCC ${GCC_VER} (pass #1)..."
	@mkdir -p gcc-build
	@( cd gcc-build && \
		../gcc-${GCC_VER}/configure --prefix=${PREFIX} --with-sysroot=${PREFIX} --target=x86_64-w64-mingw32 --enable-languages=c,c++ --enable-targets=all && \
		${MAKE} ${NUM_JOBS} all-gcc && ${SUDO} ${MAKE} install-gcc ) > gcc-build.log 2>&1 

${PREFIX}/mingw/lib/libshell32.a: ${PREFIX}/bin/x86_64-w64-mingw32-gcc
	@echo "Installing mingw-w64 runtime libraries..."
	@mkdir -p mingw-build2
	@( cd mingw-build2 && \
		../mingw-w64-v${MINGW_VER}/configure --prefix=${PREFIX}/x86_64-w64-mingw32 --build=x86_64-pc-linux-gnu --host=x86_64-w64-mingw32 && \
		${MAKE} ${NUM_JOBS} && ${SUDO} ${MAKE} install ) > crt-build.log 2>&1

${PREFIX}/mingw/lib/libgcc.a: ${PREFIX}/mingw/lib/libshell32.a
	@echo "Building and installing GCC ${GCC_VER} (pass #2)..."
	@mkdir -p gcc-build2
	@( cd gcc-build2 && \
		../gcc-${GCC_VER}/configure --prefix=${PREFIX} --with-sysroot=${PREFIX} --target=x86_64-w64-mingw32 --enable-targets=all && \
		${MAKE} ${NUM_JOBS} all && ${SUDO} ${MAKE} install ) > gcc-build2.log 2>&1 

hello32.exe: hello.c | ${PREFIX}/mingw/lib/libgcc.a
	x86_64-w64-mingw32-gcc -m32 -o $@ $<
hello64.exe: hello.c | ${PREFIX}/mingw/lib/libgcc.a
	x86_64-w64-mingw32-gcc -o $@ $<
hello.exe: hello32.exe hello64.exe
	ln -sf hello64.exe hello.exe

all: hello.exe
