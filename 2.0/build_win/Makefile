# MinGW/MinGW-w64 Makefile for PLINK 2.00.
#
# Compilation options (leave blank after "=" to disable, put "= 1" to enable):
#   Do not use AVX2 instructions: NO_AVX2
#   Do not use SSE 4.2 instructions: NO_SSE42
#   Print clear error message if SSE42/AVX2 needed but missing: CPU_CHECK
#   Do not link to OpenBLAS: NO_OPENBLAS
#   Use only -O2 optimization for zstd: ZSTD_O2
NO_AVX2 = 1
NO_SSE42 =
CPU_CHECK = 1
NO_OPENBLAS =
ZSTD_O2 = 1

OPENBLAS_ROOT = ../../openblas
ZLIB_STATIC = ../../zlib-1.2.11/libz.a

BASEFLAGS=
# ***** end configuration *****

include ../Makefile.src

BASEFLAGS += -DSTATIC_ZLIB -fno-exceptions
LINKFLAGS=-lm -static-libgcc -L. ${ZLIB_STATIC}
ifdef NO_OPENBLAS
  BASEFLAGS += -DNOLAPACK
  BLASFLAGS=
else
  BASEFLAGS += -DUSE_OPENBLAS
  BLASFLAGS=-Wl,-Bstatic -L. ${OPENBLAS_ROOT}/lib/libopenblas.a
endif

CFLAGS=-O2 -std=gnu99
CXXFLAGS=-O2 -std=gnu++11
CPUCHECK_FLAGS =

ifdef NO_AVX2
  ifndef NO_SSE42
    BASEFLAGS += -msse4.2 -DZSTD_MULTITHREAD
    ifdef CPU_CHECK
      BASEFLAGS += -DCPU_CHECK_SSE42
      CPUCHECK_FLAGS = -O2 -DCPU_CHECK_SSE42 ${CXXWARN2}
    endif
  endif
else
  BASEFLAGS += -mavx2 -mbmi2 -mlzcnt -DZSTD_MULTITHREAD
  ifdef CPU_CHECK
    BASEFLAGS += -DCPU_CHECK_AVX2
    CPUCHECK_FLAGS = -O2 -DCPU_CHECK_AVX2 ${CXXWARN2}
  endif
endif

ifdef ZSTD_O2
  ZCFLAGS=-O2 -std=gnu99
else
  ZCFLAGS=-O3 -std=gnu99
endif

BASEFLAGS += -I${OPENBLAS_ROOT}/include

ZCFLAGS += ${BASEFLAGS}

ifndef NO_AVX2
  BASEFLAGS += -mbmi
endif
CFLAGS += ${BASEFLAGS} ${CWARN2} ${CINCLUDE2}
ZCFLAGS += ${CWARN2} ${ZSTD_INCLUDE2}
CXXFLAGS += ${BASEFLAGS} ${CXXWARN2}

all: plink2 pgen_compress

plink2: $(CSRC2) $(ZCSRC2) $(CCSRC2) ../plink2_cpu.cc
	gcc $(CFLAGS) $(CSRC2) -c
	gcc $(ZCFLAGS) $(ZCSRC2) -c
	g++ $(CXXFLAGS) $(CCSRC2) -c
	g++ $(CPUCHECK_FLAGS) ../plink2_cpu.cc -c
	gfortran $(OBJ2) plink2_cpu.o -o plink2 $(BLASFLAGS) $(LINKFLAGS)

pgen_compress: ../plink2_base.cc ../pgenlib_internal.cc ../pgen_compress.cc
	g++ $(CXXFLAGS) ../plink2_base.cc ../pgenlib_internal.cc ../pgen_compress.cc -o pgen_compress

.PHONY: clean
clean:
	rm -f *.o
	rm -f plink2.exe
	rm -f pgen_compress.exe
