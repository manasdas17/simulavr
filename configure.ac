##
## $Id$
##

####
# common initialisation of autotools
####
AC_INIT([SimulAVR], [1.1dev], [simulavr-devel@nongnu.org])
AM_INIT_AUTOMAKE([1.10 -Wall -Werror foreign])
AC_PREREQ([2.61])
AC_CONFIG_MACRO_DIR([m4])
AM_MAINTAINER_MODE

####
# check for libdl and libz, if it is necessary to link simulavr successful
####
AC_ARG_ENABLE(
  libdl,
  AC_HELP_STRING([--enable-libdl],
    [check an link libdl and libz, use only for distributions or OS, where it's necessary to link all together]),
    [
      link_libdl_enable="yes"
    ],[
      link_libdl_enable="no"
    ])
AC_CHECK_LIB(dl, dlopen, [EXTRA_LIBS_LDL="yes"])
AC_CHECK_LIB(z, deflate, [EXTRA_LIBS_LZ="yes"])
if test "$link_libdl_enable" = "yes"; then
  if test x"$EXTRA_LIBS_LDL" = x""; then
    AC_MSG_ERROR([required library libdl not found])
  fi
  if test x"$EXTRA_LIBS_LZ" = x""; then
    AC_MSG_ERROR([required library libz not found])
  fi
  EXTRA_LIBS="$EXTRA_LIBS -ldl -lz"
fi

####
# check for OS and build system: MSYS/MingW
####
AC_SYS_CHECK_MINGW
if test "$ac_sys_check_uname_o" = "Msys"; then
  AC_SUBST([PYTHON_MODULE_EXTENSION],[.pyd])
else
  AC_SUBST([PYTHON_MODULE_EXTENSION],[.so])
fi

####
# support of libtool, g++, swig (for tcl module and python module used)
####
LT_INIT
LT_PREREQ([2.2])
AC_PROG_CXX
AC_CHECK_PROG(CXX_PROG,${CXX},yes,no)
if test ${CXX_PROG} = no ; then
  # AC_PROG_CXX sets g++, if nothing is found, but if not found make will fail
  AC_MSG_ERROR([C++ compiler ${CXX} not found],1)
fi
AC_PROG_SWIG(2.0.0)
if test "x$SWIG_LIB" = "x"; then
  # fall back to SWIG version 1.3.x in case of SWIG 2.0.x not found!
  AC_MSG_NOTICE([Don't worry, we can try to find SWIG 1.3.x!])
  AC_PROG_SWIG(1.3.18)
fi
SWIG_ENABLE_CXX
SWIG_MULTI_MODULE_SUPPORT
AM_CONDITIONAL([USE_SWIG],[test "x$SWIG" != 'x'])

####
# tcl/tk support
####
## Check all about Tcl and parse command line arguments
AX_TCL_ENVIRONMENT

####
# python support, used for regression test and python module
####
AZ_PYTHON_DEFAULT
AZ_PYTHON_ENABLE
AZ_PYTHON_VERSION_ENSURE([2.4.0])
SWIG_PYTHON
AM_PATH_PYTHON

###
# Regression test suite support programs
##
# If 'time' command is installed, we want to use it for the regression test suite,
# it's behavior this is different from the built-in 'time' command in bash.
AC_CHECK_PROG(GNUTIME,time,yes,no)
  
####
# Check about the AVR cross compilation environment, check for libz (only on msys,
# used automaticaly by libtool!), libbfd and libiberty
####
AX_AVR_ENVIRONMENT
if test "$ac_sys_check_uname_o" = "Msys"; then
  # need libwsock32 too
  AVR_CHECK_WINSOCK
  # libtool links to libz by default, but on MSYS/MingW this is on other place
  AVR_CHECK_ZLIB_MSYS
  # add extra libs setting, if set on commandline, but only on msys
  AC_SUBST([EXTRA_LIBS])
fi

## Check for the XML files in src/fab/atmel-xml. This just checks for the
## existence of the file ATmega8.xml and assumes that all files have been placed into
## this directory.
AC_MSG_CHECKING([XML part description files in src/fab/atmel-xml/])
AM_CONDITIONAL([USE_FAB], [test -f "${srcdir}/src/fab/atmel-xml/ATmega8.xml"])
if test X"${USE_FAB_FALSE}" == X"" ; then
   AC_MSG_RESULT([NOT available])
else
   AC_MSG_RESULT([available])
fi

## Examples compile to avr code.  Do not build them if avr tools not installed
## NOTE: Cheap check -- only seeing if avr-gcc was found
if test X"${AVR_GCC}" = X"" ; then
  AC_MSG_WARN([])
  AC_MSG_WARN([AVR Cross tools not installed])
  AC_MSG_WARN([Not building examples and do tests])
  AC_MSG_WARN([])
  ac_have_avr_cross_tools=no
else
  ac_have_avr_cross_tools=yes
fi

AM_CONDITIONAL([USE_AVR_CROSS], [test x"${ac_have_avr_cross_tools}" = x"yes"])

####
# support verilog interface to build avr.vpi
####
AC_CHECK_HEADERS([vpi_user.h iverilog/vpi_user.h],
    [AC_DEFINE(HAVE_VERILOG, [1], Icarus verilog interface)
        WE_HAVE_VERILOG="yes"],  
    [])
AC_ARG_ENABLE([verilog],
    [AS_HELP_STRING([--enable-verilog],[enables build of verilog modul avr.vpi])],
    [test "$WE_HAVE_VERILOG" = "yes" && AVR_BUILD_VERILOG="yes"],
    [AVR_BUILD_VERILOG="no"])
AM_CONDITIONAL([USE_VERILOG], [test "$AVR_BUILD_VERILOG" = "yes"])
AC_PATH_PROG([GTKWAVE], [gtkwave])
AM_CONDITIONAL([USE_GTKWAVE],[test "x$GTKWAVE" != "x"])
AC_PATH_PROG([IVERILOG], [iverilog])
AC_PATH_PROG([VVP], [vvp])
AM_CONDITIONAL([USE_VERILOG_TOOLS],[test "x$IVERILOG" != "x" -a "x$VVP" != "x"])

####
# support for some documenatation tools, texinfo and doxygen
####
AC_CHECK_PROGS(TEX, [tex], [no])
AM_CONDITIONAL([HAVE_TEX], [test x"${TEX}" != x"no"])
AC_CHECK_PROG(MAKEINFO_PROG,makeinfo,yes,no)
if test ${MAKEINFO_PROG} = no ; then
  # makeinfo is used by creating info document
  AC_MSG_ERROR([makeinfo not found],1)
fi
DX_DOXYGEN_FEATURE(OFF)
DX_PDF_FEATURE(OFF)
DX_PS_FEATURE(OFF)
DX_INIT_DOXYGEN([Simulavr], [simulavr.doxy], )

####
# support for google test
####
AC_MSG_CHECKING([google test library])
AC_ARG_WITH([gtest],
  [AS_HELP_STRING([--with-gtest=path  directory with google test library files])],
  [if test ! -d ${with_gtest} -o ! -f ${with_gtest}/include/gtest/gtest.h -o ! -f ${with_gtest}/src/gtest.cc ; then
     AC_MSG_ERROR([(${with_gtest}) is not a directory with google test library files])
     with_gtest=
     AC_MSG_RESULT([NOT available])
   else
     AC_MSG_RESULT([available])
   fi
  ],
  [with_gtest=
   AC_MSG_RESULT([NOT configured])
  ]
)
AM_CONDITIONAL([GTEST_USE], [test x"${with_gtest}" != x])
AC_SUBST([GTEST_LIB_PATH],[${with_gtest}])

####
# definitions for distcheck
####
## We have to propagate all the configure flags we want to use when
## configuring for make distcheck
DISTCHECK_CONFIGURE_FLAGS="${DISTCHECK_CONFIGURE_FLAGS} \
  --with-bfd=${libbfd_root_location} \
  --with-libiberty=${libiberty_root_location}"
if test X"${tclconfig_root_path}" != X ; then
  DISTCHECK_CONFIGURE_FLAGS="${DISTCHECK_CONFIGURE_FLAGS} \
    --with-tclconfig=${tclconfig_root_path}"
fi
AC_SUBST([DISTCHECK_CONFIGURE_FLAGS])

####
# specify files to create
####
## Output all the configure generated files that are required all the time
AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_FILES(
  [regress/regress.py],
  [chmod +x regress/regress.py])
AC_CONFIG_FILES([
  Makefile src/Makefile src/hwtimer/Makefile src/cmd/Makefile src/ui/Makefile
  src/python/Makefile src/setup.py doc/Makefile doc/conf.py doc/web/Makefile
  doc/web/conf.py doc/config.texi regress/Makefile regress/modules/Makefile
  regress/test_opcodes/Makefile regress/avrtest/Makefile regress/gtest/Makefile
  regress/timertest/Makefile regress/extinttest/Makefile regress/modtest/Makefile
  examples/verilog/Makefile examples/Makefile examples/anacomp/Makefile
  examples/atmega48/Makefile examples/atmega128_timer/Makefile
  examples/atmel_key/Makefile examples/feedback/Makefile examples/simple_ex1/Makefile
  examples/spi/Makefile examples/stdiodemo/Makefile examples/python/Makefile
])

## Certain files should only be generated if Tcl is available and enabled
if test x"${build_tcl_libs}" = x"yes" -a x"${HAVE_TCL}" != x"no" ; then
  AC_CONFIG_FILES(
    [examples/gui.tcl],
    [chmod +x examples/gui.tcl])
  AC_CONFIG_FILES(
    [examples/simulavr.tcl],
    [chmod +x examples/simulavr.tcl])
  AC_CONFIG_FILES(
    [examples/feedback/simfeedback.tcl],
    [chmod +x examples/feedback/simfeedback.tcl])
  AC_CONFIG_FILES(
    [examples/stdiodemo/checkdebug.tcl],
    [chmod +x examples/stdiodemo/checkdebug.tcl])
  AC_CONFIG_FILES(
    [examples/spi/check.tcl],
    [chmod +x examples/spi/check.tcl])
  AC_CONFIG_FILES(
    [examples/atmega48/check.tcl],
    [chmod +x examples/atmega48/check.tcl])
else
  AC_MSG_WARN([])
  AC_MSG_WARN([Tcl Development Kit Not Installed])
  AC_MSG_WARN([Not building Tcl interface or examples])
  AC_MSG_WARN([])
fi

####
# generate files
####
AC_OUTPUT

####
# print out a summary, what configure found out
####
echo
echo "Summary:"
echo "build system = $ac_sys_check_uname_o"
echo "AVR_GCC=${AVR_GCC}"
echo "AVR_LIBBFD_INC=${AVR_LIBBFD_INC}"
echo "AVR_LIBBFD_LIB=${AVR_LIBBFD_LIB}"
echo "AVR_LIBIBERTY_LIB=${AVR_LIBIBERTY_LIB}"
echo "PYTHON=${PYTHON}"
if test x"${build_tcl_libs}" = x"yes" -a x"${HAVE_TCL}" != x"no" ; then
  echo "TCL_WISH=${TCL_WISH}"
  echo "tcl has package Itcl? ${check_itcl_available}"
fi
echo "build verilog modul avr.vpi = ${AVR_BUILD_VERILOG}"
# EOF

