---
title: Acoustic Toolbox和Arlpy配置  ## 标题
author: Zane ## 作者
comments: true
date:  2025-07-03 13:24:12
description: 如何配置声学工具箱 ## 描述
tags:  配置 ## 标签
categories:
 - [编程] ## 分类
top_img: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg
cover: https://img3.wallspic.com/crops/2/9/8/1/7/171892/171892-lu_xing-cheng_shi-li_cheng_bei-cheng_shi_jing_guan-3840x2160.jpg  ## 封面

---



# Acoustic Toolbox和Arlpy配置

python的arlpy包是调用Acoustic Toolbox中的fortran进行仿真计算，首先需要下载Acoustic Toolbox

### 下载Acoustic Toolbox，

下载链接：

- http://oalib.hlsresearch.com/AcousticsToolbox/at_2024_12_25.zip（Mac/Linux/Windows – 必须要手动进行编译）
- http://oalib.hlsresearch.com/AcousticsToolbox/atWin10_2020_11_4.zip（旧版本，涵盖了MATLAB I/O）

### 下载后，将工具包解压，打开命令行，进行配置

```bash
cd at/at  			#访问解压文件所在路径
make clean 			#清除之前的编译文件
make all  			#进行编译
sudo make install   #安装
```

（这里可能会因为电脑操作系统不同产生报错,需要修改makefile文件)

![makefile](http://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/202507031310234.png)

如果是Mac电脑，将makefile替换成以下内容

```bash
# These lines are used under Mac OSX; the syntax is different under Windows
# !!!!!!! You also need to use xiar instead of ar
# That change needs to be made in at/misc/Makefile and at/tslib/Makefile
# Note that the -assume byterecl is important, otherwise the output files have the wrong format

# If you use the -parallel option, the compiler links into a dynamic library for OpenMP
# Then you need to make sure that dylib is in the path

# Choose the best architecture (target machine) using the -x switch
# According to the latest benchmarks on polyhedron, it looks like -parallel is very helpful

# need -heap-arrays to avoid stack overflows for big runs ...
# -no-wrap-margin was added to avoid ascii arrival files being wrapped at 80 characters

export FC=ifort
# export FFLAGS= -O3 -parallel -axSSE4.2 -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -fast -axAVX                           -parallel              -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -axAVX              -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -axAVX              -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -xHost -qopt-report -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -xHost              -funroll-loops           -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -no-wrap-margin

# recommended settings from POLYHEDRON site
# export FFLAGS= -O3 -fast           -ipo -nostandard-realloc-lhs
# export FFLAGS= -O3 -fast -parallel -ipo -nostandard-realloc-lhs
# export FFLAGS= -O3 -fast -parallel -ipo -no-prec-div                                       -assume byterecl -heap-arrays
# export FFLAGS= -O3 -fast -parallel -ipo -no-prec-div -qopt-report-phase=par -qopt-report:5 -assume byterecl -heap-arrays
# export FFLAGS= -O3 -nologo                                               -inline-level=2 -assume byterecl -threads -heap-arrays
# export FFLAGS= -O3 -ipo -funroll-loops -xAVX -no-prec-div -axAVX -nologo -inline-level=2 -assume byterecl -threads -heap-arrays

# compilation diagnostics on:
# export FFLAGS= -O3 -fast -parallel -ipo -nostandard-realloc-lhs -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -check -traceback
# export FFLAGS= -O3 -fast           -ipo -nostandard-realloc-lhs -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -check -traceback

# runtime diagnostics on as well:
#export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all -ftrapuv -fpe0 -gen-interfaces -traceback
#export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all -ftrapuv                       -traceback -check noarg_temp_created
export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all                                -traceback -check noarg_temp_created

# profiling:
# export FFLAGS= -g -O3 -profile-functions -profile-loops=all -xHost -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -traceback

###########export FFLAGS+= -I/opt/intel/compilers_and_libraries_2018.2.164/mac/compiler/lib/libiomp5.dylib

# ______________________________________________________________________________

# *** GNU Compiler Collection GFORTRAN

# use -march=generic if you get warning messages about instructions that don't make sense
# -march=generic assumes an old Intel architecture that the newer versions can all execute (slowly)
# -march=native should normally be the best; however, it produced AVX instructions on the Mac that the default assembler could not process
# -O2 was the highest level of optimization that worked under Windows

# -static can be used to tell gfortran not to rely on a dynamic link library (the compiler may or may not support)
# -static does not seem to work on Macs though, and produces larger executables

# Have had various problems where some installed dynamic link library is incompatible with the one the compiler used and expects at run time
# For instance, Matlab changes paths and may point to an incompatible library.
# One user found that it was necessary to delete /usr/local/gfortran/lib/libquadmath.dylib to force a static link. See:
# http://stackoverflow.com/questions/17590525/correct-way-to-statically-link-in-gfortran-libraries-on-osx
#
# The -Wa,-q flag can be used to select the Mac CLANG assembler instead of the GNU assembler
# At one time that was necessary to get the AVX operations; however, I saw no speed benefit
# -march=corei7-avx works on my Mac

export FC=gfortran

# export FFLAGS= -march=native  -Wall -std=gnu -O3 -ffast-math -funroll-all-loops -msse3 -fomit-frame-pointer -mtune=native -Q
# export FFLAGS= -march=corei7 -Bstatic -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation       -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native
# -L/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk/usr/lib
# export FFLAGS= -march=corei7 -Bstatic -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation        -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer
# export FFLAGS= -march=native          -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation -Wa,-q -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native
# Had a problem with -O2 in the KRAKENC root finder for at/tests/Noise/Sduct. Switching to O1 (4//25/2023)
export FFLAGS= -march=native -Bstatic -Waliasing -Wampersand              -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation         -std=gnu  -O1 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native

# Compilation and run-time diagnostics on:
# omni.env fails trap=invalid
# export FFLAGS= -march=native -ffpe-trap=invalid,zero,overflow -Wall                  -std=gnu -O1 -fcheck=all -fbacktrace
# export FFLAGS= -march=native -ffpe-trap=zero,overflow         -Wall                  -std=gnu -O1 -fcheck=all -fbacktrace
#export FFLAGS= -march=native -ffpe-trap=zero,overflow         -Wall -pedantic-errors -std=gnu -O1 -fcheck=all -fbacktrace -finit-real=nan

# Profiling:
# I read that the -pg flag is needed for profiling, but there's some problem with the library and it doesn't compile
# It does compile with just the -p and -g flags but does not appear to have enough info for the xcode instruments
# export FFLAGS= -p -g -pg -march=native -Wall -std=gnu
# export FFLAGS= -p -g -march=native -Bstatic -Wa,-q -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation -std=gnu -O3 -ffast-math -funroll-all-loops -fcheck=all

# ______________________________________________________________________________

# *** g95
# This is no longer working with the Acoustics Toolbox because it uses Fortran features that the g95 compiler has not implemented
# export FC=g95
# export FFLAGS = -Wall -std=f2003 -O3

# compilation diagnostics on:
# export FFLAGS = -Wall -std=f2003 -ftrace=full -fbounds-check
# export FFLAGS = -pg   -std=f2003

# ______________________________________________________________________________

# *** Portland Group FORTRAN
# -Mnoframe caused erroneous results
# -Munroll  caused erroneous results
# These are defaults under -fast, so can't use -fast either

# export FC=pgfortran
# export FFLAGS= -Mconcur
# export FFLAGS= -fast
# export FFLAGS= -O2 -Munroll=c:1 -Mnoframe -Mlre -Mpre -Mvect=sse -Mcache_align -Mflushz -Mvect
# export FFLAGS= -O2 -Mlre -Mpre -Mvect=sse -Mcache_align -Mflushz -Mvect

# compilation diagnostics on:
# export FFLAGS= -g -Minfo=ccff -Minform=inform -C

# ______________________________________________________________________________

export RM=rm
export CC=gcc
export CFLAGS=-g
#export FFLAGS+= -I../misc -I../tslib -I/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include
export FFLAGS+= -I../misc -I../tslib

# KRAKEL is commented out below because it requires the LAPACK library.
# If you have the LAPACK library installed on your system, first edit the
# LAPACK_LIBS variable below to point to your installation, then you can
# uncomment the make commands below (uncomment both the "all" and "install"
# targets).

export LAPACK_LIBS = -llapack

all:
	(cd misc;	make -k all)
	(cd tslib;	make -k all)
	(cd Bellhop;	make -k all)
	(cd Kraken;	make -k all)
	(cd KrakenField;	make -k all)
	# (cd Krakel;	make -k all)
	(cd Scooter;	make -k all)
	@echo " "
	@echo "***********************************"
	@echo "***** Acoustics Toolbox built *****"
	@echo "***********************************"

install:
	(cd misc;	make -k all)
	(cd tslib;	make -k all)
	(cd Bellhop;	make -k install)
	(cd Kraken;	make -k install)
	(cd KrakenField;        make -k install)
	# (cd Krakel;	make -k install)
	(cd Scooter;	make -k install)
	@echo " "
	@echo "***************************************"
	@echo "***** Acoustics Toolbox installed *****"
	@echo "***************************************"

clean:
	-rm -f bin/*.exe
	find . -name '*.dSYM' -exec rm -r {} +
	find . -name '*.png'  -exec rm -r {} +
	find . -name '*.eps'  -exec rm -r {} +
	find . -name '*.mod'  -exec rm -r {} +
	find . -name '*.grn'  -exec rm -r {} +
	find . -name '*.shd'  -exec rm -r {} +
	find . -name '*.shd.mat'  -exec rm -r {} +
	find . -name '*.prt'  -exec rm -r {} +
	(cd misc;	make -k -i clean)
	(cd tslib;	make -k -i clean)
	(cd Bellhop;	make -k -i clean)
	(cd Kraken;	make -k -i clean)
	(cd KrakenField;	make -k -i clean)
	(cd Krakel;	make -k -i clean)
	(cd Scooter;	make -k -i clean)
	(cd tests;	make -k -i clean)
```

如果是Linux操作内核，替换成以下内容

```bash
#
# To install the Acoustics Toolbox:
#
# 1) Uncomment the appropriate lines below to select your FORTRAN compiler
#    (also be sure to comment out all of the lines corresponding to the other compilers).

# 2) If you're using gfortran check the -march switch that selects the chip you're using.
#    Usually -march=native works

# 3) From a command line shell, run:
#    % make clean
#    % make

# on some machines you need to say -mcmodel=medium (or large) to allow for variables larger than 2 gig

# *** Windows ***
# If you don't have a FORTRAN compiler, the MinGW ("Minimalist GNU for Windows") compiler suite
# is a good choice as it is much easier to install than Cygwin. See http://www.mingw.org/ for details.
# Not sure if this is still necessary, but we used to have to change the options -O3 to -O2 below when using gfortran

# *** Linux ***
# Most Linux distributions have gfortran already packaged and it can be installed with the respective 
# package manager (e.g. apt-get, dnf, yum). The packaged versions of the LAPACK library are generally
# compatible with Krakel. If you want statically linked executables, "-static" works with gfortran.

# *** Mac ***
# The option to create an executable that uses a static library has been a problem.
# The gcc community seems to feel that dynamic vs. static libraries is an issue of religious importance.
# I prefer static libraries because users have a lot fewer problems installing the code when they don't have
# to worry about getting the LD_LIBRARY_PATH set, or the fact that some programs (e.g. Matlab) may reset
# that path to an incompatible version.
# -static is supposed to work if gfortran has been compiled to enable that option.
# However, crt0.o comes up as misssing
# -Bstatic worked
# You may need to specify an architecture flag under gfortran:
# -march=native ( for M2 processor)
# -mcpu=ARMv8.6-A ( for generic ARM processor instruction sets)
# For further info, see https://oalib-acoustics.org/website_resources/AcousticsToolbox/AT_compile_mac_linux.html
# 11/27/2023:
# A user reported issues with security on the HPC binary files. Apparently some of the downloaded files were quarantined by the Mac OS
# and needed fixing using the xattr command
# Also a user noted that Xcor developer tools are not needed. Apparently the Command Line Tools are sufficient.
# A user reported that the warning messages produced when running the Makefile were confusing, but that the compiled code worked anyway.

# The make utility tends to get confused with modules because it does not necessarily update the *.mod file
# when it compiles the *.f90 file. (The *.mod file contains interface information that doesn't necessarily change
# even when the *.f90 file has changed.)
# As a result, you may find that make keeps compiling a module, that was already compiled.
# A 'make clean' will fix that
# ______________________________________________________________________________

# *** ifort
# These lines are used under Mac OSX; the syntax is different under Windows
# !!!!!!! You also need to use xiar instead of ar
# That change needs to be made in at/misc/Makefile and at/tslib/Makefile
# Note that the -assume byterecl is important, otherwise the output files have the wrong format

# If you use the -parallel option, the compiler links into a dynamic library for OpenMP
# Then you need to make sure that dylib is in the path

# Choose the best architecture (target machine) using the -x switch
# According to the latest benchmarks on polyhedron, it looks like -parallel is very helpful

# need -heap-arrays to avoid stack overflows for big runs ...
# -no-wrap-margin was added to avoid ascii arrival files being wrapped at 80 characters

export FC=ifort
# export FFLAGS= -O3 -parallel -axSSE4.2 -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -fast -axAVX                           -parallel              -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -axAVX              -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -axAVX              -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -xHost -qopt-report -funroll-loops -parallel -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays
export FFLAGS= -O3 -xHost              -funroll-loops           -no-prec-div -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -no-wrap-margin

# recommended settings from POLYHEDRON site
# export FFLAGS= -O3 -fast           -ipo -nostandard-realloc-lhs
# export FFLAGS= -O3 -fast -parallel -ipo -nostandard-realloc-lhs
# export FFLAGS= -O3 -fast -parallel -ipo -no-prec-div                                       -assume byterecl -heap-arrays
# export FFLAGS= -O3 -fast -parallel -ipo -no-prec-div -qopt-report-phase=par -qopt-report:5 -assume byterecl -heap-arrays
# export FFLAGS= -O3 -nologo                                               -inline-level=2 -assume byterecl -threads -heap-arrays
# export FFLAGS= -O3 -ipo -funroll-loops -xAVX -no-prec-div -axAVX -nologo -inline-level=2 -assume byterecl -threads -heap-arrays

# compilation diagnostics on:
# export FFLAGS= -O3 -fast -parallel -ipo -nostandard-realloc-lhs -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -check -traceback
# export FFLAGS= -O3 -fast           -ipo -nostandard-realloc-lhs -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -check -traceback

# runtime diagnostics on as well:
#export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all -ftrapuv -fpe0 -gen-interfaces -traceback
#export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all -ftrapuv                       -traceback -check noarg_temp_created
#export FFLAGS= -nologo     -inline-level=2 -assume byterecl -threads -heap-arrays -check all                                -traceback -check noarg_temp_created

# profiling:
# export FFLAGS= -g -O3 -profile-functions -profile-loops=all -xHost -nologo -inline-level=2 -assume byterecl -threads -heap-arrays -traceback

###########export FFLAGS+= -I/opt/intel/compilers_and_libraries_2018.2.164/mac/compiler/lib/libiomp5.dylib

# ______________________________________________________________________________

# *** GNU Compiler Collection GFORTRAN

# use -march=generic if you get warning messages about instructions that don't make sense
# -march=generic assumes an old Intel architecture that the newer versions can all execute (slowly)
# -march=native should normally be the best; however, it produced AVX instructions on the Mac that the default assembler could not process
# -O2 was the highest level of optimization that worked under Windows

# -static can be used to tell gfortran not to rely on a dynamic link library (the compiler may or may not support)
# -static does not seem to work on Macs though, and produces larger executables

# Have had various problems where some installed dynamic link library is incompatible with the one the compiler used and expects at run time
# For instance, Matlab changes paths and may point to an incompatible library.
# One user found that it was necessary to delete /usr/local/gfortran/lib/libquadmath.dylib to force a static link. See:
# http://stackoverflow.com/questions/17590525/correct-way-to-statically-link-in-gfortran-libraries-on-osx
#
# The -Wa,-q flag can be used to select the Mac CLANG assembler instead of the GNU assembler
# At one time that was necessary to get the AVX operations; however, I saw no speed benefit
# -march=corei7-avx works on my Mac

export FC=gfortran

# export FFLAGS= -march=native  -Wall -std=gnu -O3 -ffast-math -funroll-all-loops -msse3 -fomit-frame-pointer -mtune=native -Q
# export FFLAGS= -march=corei7 -Bstatic -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation       -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native
# -L/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk/usr/lib
# export FFLAGS= -march=corei7 -Bstatic -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation        -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer
# export FFLAGS= -march=native          -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation -Wa,-q -std=f2008 -O3 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native
# Had a problem with -O2 in the KRAKENC root finder for at/tests/Noise/Sduct. Switching to O1 (4//25/2023)
# export FFLAGS= -march=native -Bstatic -Waliasing -Wampersand              -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation         -std=gnu  -O1 -ffast-math -funroll-all-loops -fomit-frame-pointer -mtune=native
export FFLAGS= -march=native -Bstatic -Waliasing -Wampersand              -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation         -std=gnu  -O1 -ffast-math -funroll-all-loops -fomit-frame-pointer

# Compilation and run-time diagnostics on:
# omni.env fails trap=invalid
# export FFLAGS= -march=native -ffpe-trap=invalid,zero,overflow -Wall                  -std=gnu -O1 -fcheck=all -fbacktrace
# export FFLAGS= -march=native -ffpe-trap=zero,overflow         -Wall                  -std=gnu -O1 -fcheck=all -fbacktrace
#export FFLAGS= -march=native -ffpe-trap=zero,overflow         -Wall -pedantic-errors -std=gnu -O1 -fcheck=all -fbacktrace -finit-real=nan

# Profiling:
# I read that the -pg flag is needed for profiling, but there's some problem with the library and it doesn't compile
# It does compile with just the -p and -g flags but does not appear to have enough info for the xcode instruments
# export FFLAGS= -p -g -pg -march=native -Wall -std=gnu
# export FFLAGS= -p -g -march=native -Bstatic -Wa,-q -Waliasing -Wampersand -Wsurprising -Wintrinsics-std -Wno-tabs -Wintrinsic-shadow -Wline-truncation -std=gnu -O3 -ffast-math -funroll-all-loops -fcheck=all

# ______________________________________________________________________________

# *** g95
# This is no longer working with the Acoustics Toolbox because it uses Fortran features that the g95 compiler has not implemented
# export FC=g95
# export FFLAGS = -Wall -std=f2003 -O3

# compilation diagnostics on:
# export FFLAGS = -Wall -std=f2003 -ftrace=full -fbounds-check
# export FFLAGS = -pg   -std=f2003

# ______________________________________________________________________________

# *** Portland Group FORTRAN
# -Mnoframe caused erroneous results
# -Munroll  caused erroneous results
# These are defaults under -fast, so can't use -fast either

# export FC=pgfortran
# export FFLAGS= -Mconcur
# export FFLAGS= -fast
# export FFLAGS= -O2 -Munroll=c:1 -Mnoframe -Mlre -Mpre -Mvect=sse -Mcache_align -Mflushz -Mvect
# export FFLAGS= -O2 -Mlre -Mpre -Mvect=sse -Mcache_align -Mflushz -Mvect

# compilation diagnostics on:
# export FFLAGS= -g -Minfo=ccff -Minform=inform -C

# ______________________________________________________________________________

export RM=rm
export CC=gcc
export CFLAGS=-g
#export FFLAGS+= -I../misc -I../tslib -I/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include
export FFLAGS+= -I../misc -I../tslib

# KRAKEL is commented out below because it requires the LAPACK library.
# If you have the LAPACK library installed on your system, first edit the
# LAPACK_LIBS variable below to point to your installation, then you can
# uncomment the make commands below (uncomment both the "all" and "install"
# targets).

export LAPACK_LIBS = -llapack

all:
	(cd misc;	make -k all)
	(cd tslib;	make -k all)
	(cd Bellhop;	make -k all)
	(cd Kraken;	make -k all)
	(cd KrakenField;	make -k all)
	# (cd Krakel;	make -k all)
	(cd Scooter;	make -k all)
	@echo " "
	@echo "***********************************"
	@echo "***** Acoustics Toolbox built *****"
	@echo "***********************************"

install:
	(cd misc;	make -k all)
	(cd tslib;	make -k all)
	(cd Bellhop;	make -k install)
	(cd Kraken;	make -k install)
	(cd KrakenField;        make -k install)
	# (cd Krakel;	make -k install)
	(cd Scooter;	make -k install)
	@echo " "
	@echo "***************************************"
	@echo "***** Acoustics Toolbox installed *****"
	@echo "***************************************"

clean:
	-rm -f bin/*.exe
	find . -name '*.dSYM' -exec rm -r {} +
	find . -name '*.png'  -exec rm -r {} +
	find . -name '*.eps'  -exec rm -r {} +
	find . -name '*.mod'  -exec rm -r {} +
	find . -name '*.grn'  -exec rm -r {} +
	find . -name '*.shd'  -exec rm -r {} +
	find . -name '*.shd.mat'  -exec rm -r {} +
	find . -name '*.prt'  -exec rm -r {} +
	(cd misc;	make -k -i clean)
	(cd tslib;	make -k -i clean)
	(cd Bellhop;	make -k -i clean)
	(cd Kraken;	make -k -i clean)
	(cd KrakenField;	make -k -i clean)
	(cd Krakel;	make -k -i clean)
	(cd Scooter;	make -k -i clean)
	(cd tests;	make -k -i clean)
```

### 设置路径

每次开机时，都需要输入Acoustic Toolbox所在路径，这样电脑每次计算时才会使用里面的程序进行计算

打开终端，输入以下内容，查看bellhop安装路径

```bash
find ~ -name "bellhop.exe"
```

（如果你是windows操作系统，直接将路径加入环境变量**PATH**中（系统属性 -> 环境变量））

![image-20250703131147956](http://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/202507031311006.png)

或者直接在终端中输入

```bash
export PATH=$PATH:/Users/yourusername/Documents/Bellhop # 这里设置at中Bellhop文件夹的路径
```

为了每次电脑启动后，都不用手动配置，可以将这条命令加入进zsh或者bash配置文件中

```bash
echo 'export PATH=$PATH:/Users/yourusername/Documents/Bellhop' >> ~/.zshrc#如果时bash改成.bashrc
source ~/.zshrc#配置生效，如果时bash改成.bashrc
```

验证路径是否生效

```bash
which bellhop.exe
```

如果返回了bellhop.exe的路径，说明配置成功

#### 在jupyter中查看路径是否安装成功

```python
import os
os.system("which bellhop.exe")
```

如果没有返回正确路径，则每次使用前需要手动添加

```bash
import os
os.environ["PATH"] += ":/Users/zanesing/Documents/at/Bellhop"     #这里填写Bellhop文件夹路径(注意是文件夹路径，不是.exe路径)
```

### 在python中设置Arlpy

在命令行输入以下命令，下载arlpy

```bash
pip install arlpy
```

输入以下代码

```python
import arlpy.uwapm as pm
pm.models()
```

如果返回**['Bellhop']**,说明安装成功

![arlpy](http://cdn.jsdelivr.net/gh/vanillaholic/image-bed@main/202507031307424.png)

参考文档：

- https://github.com/patel999jay/Bellhop-ARLPY-ECED6575/issues/3#issuecomment-2684876150

- https://patel999jay.github.io/post/bellhop-acoustic-toolbox/
- http://oalib.hlsresearch.com/AcousticsToolbox/

