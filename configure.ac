#                                               -*- Autoconf -*-

# autoconf <= 2.64 has a bug in AC_OPENMP
AC_PREREQ([2.69])
#AC_INIT([ermod],[0.3.%rev%],[Matubayasi Laboratory])
AC_INIT([slvfe],[experimental],[Matubayasi Laboratory])
AC_CONFIG_MACRO_DIR(m4)
AM_INIT_AUTOMAKE

AC_ARG_ENABLE([debug-symbol],
              [AS_HELP_STRING([--enable-debug-symbol],
                        [Enable debug symbols (-g)])],
              [enable_debug=yes], [enable_debug=no])

AS_IF([test x$enable_debug = xno],
      [ac_cv_prog_fc_g=no])

# Checks for programs.
AC_PROG_FC([ifx ifort xlf90 xlf f95 f90 frt gfortran g95])
AC_PROG_LN_S

# Check for buggy variant of ifort.
AS_IF([test x$FC = xifort],
      [version=`$FC --version`
       case "x$version" in # (
       *12.1.*):
         AC_MSG_ERROR([This program does not run correctly with Intel Compiler 12.1.x])
       ;;
       esac
       ])

# modified MPI detector (searches mpiifort first)
AC_LANG(Fortran)

: ${enable_openmp=yes}
#AC_ARG_ENABLE([openmp],
#              AS_HELP_STRING([--enable-openmp],
#                             [use OpenMP (default = yes)]))
AC_OPENMP

AS_IF([test "x$FC" = "x"],
      [AC_MSG_ERROR("Failed to find Fortran Compiler (Fortran 90). Set environment variable FC to work around")]
      [])

AC_DEFUN([MY_CHECK_BLAS], [
  AC_MSG_CHECKING([whether $1 is available])
  save_LIBS=$LIBS
  LIBS="$LIBS $2"
  AC_LINK_IFELSE([AC_LANG_CALL([], [sgemm])],
                 [blas_ok=yes], [blas_ok=no])
  LIBS=$save_LIBS
  AC_MSG_RESULT([$blas_ok])
])

AC_ARG_WITH([mkl],
  AS_HELP_STRING([--with-mkl[[=(MKL version)]]],
    [Automatically set environmental variables for MKL; for a specific version, use this option like --with-mkl=10.2.3]))

AS_IF([test "x$with_mkl" != "x" -a "x$with_mkl" != "xyes"],
      [AC_MSG_FAILURE([Specifying version to --with-mkl=(version) is now deprecated because of Intel MKl update])])

AS_IF([test "x$with_mkl" != "x"],
  [case x`uname -m` in # (
  xx86_64) :
  mkl_plat=em64t
  mkl_platnew=intel64
  ;; # (
  xi386) :
  mkl_plat=32
  mkl_platnew=ia32
  AC_MSG_WARN([i386 platform is no longer actively maintained; if you have trouble please contact ERmod usergroup])
  ;; # (
  *) :
  ;;
  esac

  AS_IF([test "x$MKLROOT" = "x"],
    [AC_MSG_FAILURE([could not get MKL version; set MKLROOT or source /path/to/mkl/env/vars.sh])])
  mklroot=$MKLROOT
  AC_MSG_NOTICE([Using MKL installed at $MKLROOT])
  # Check for recent Intel Composer (-qmkl=parallel works; super old format "-mkl=" is now assumed extinct)
  with_blas="-qmkl=parallel"
  MY_CHECK_BLAS([Intel composer default], [$with_blas])
  AS_IF([test "x$blas_ok" = "xyes"],
        [FCFLAGS="-qmkl=parallel $FCFLAGS"],
        [
         # check 2017-format
         with_blas="-L$mklroot/lib/$mkl_platnew -lmkl_rt -Wl,--no-as-needed -lpthread -lm -ldl"
         MY_CHECK_BLAS([MKL 2017+], 
                       [$with_blas])
         AS_IF([test "x$blas_ok" = "xyes"],
               [],
               [AC_MSG_WARN([Failed to find appropriate MKL. If you are using old mkl, please update])])
         with_lapack=""
         FCFLAGS="-I$mklroot/include $FCFLAGS"
        ])
])

ACX_BLAS
ACX_LAPACK([lapack_exist=yes],
           [lapack_exist=no])

################################

AC_ARG_ENABLE([double],
    [AS_HELP_STRING([--enable-double],
        [Enable conversion of single precision real to double precision (real(8))])],
    [enable_double=yes],
    [enable_double=no])

AC_DEFUN([MY_FC_REAL_DOUBLE],[
AC_CACHE_CHECK([how to read real as real(8) as default],
my_fc_real_double_cv_flag,
[AC_LANG_PUSH(Fortran)
save_FCFLAGS=$FCFLAGS
# for default, gfortran, ifort, xlf, Fujitsu f90, Fujitsu f90 (old), Portland, Cray
for flag in "" "-fdefault-real-8 -fdefault-double-8" "-fdefault-real-8" "-autodouble" "-qrealsize=8" "-CcdRR8" "-Ad" "-Mr8" "-s default64"; do
FCFLAGS="$save_FCFLAGS $flag"
# This is very tricky indeed...
  AC_COMPILE_IFELSE(
    [AC_LANG_PROGRAM([], [dnl
      implicit none
      real :: a
      integer, parameter :: len = kind(a)
      integer, parameter :: x(len) = (/ 1, 2, 3, 4, 5, 6, 7, 8 /)
      print *, "Success"
])], [my_check_realasdouble=yes], [my_check_realasdouble=no])
  if test $my_check_realasdouble = yes; then
    break
  fi
done
if test $my_check_realasdouble = no; then
  AC_MSG_ERROR([unable to find compiler flag to read real as real(8)])
fi
FCFLAGS="$save_FCFLAGS $flag"
if test "x$flag" = x; then
  echo -n "(none)"
else
  echo -n $flag
fi
AC_LANG_POP(Fortran)
])])

AS_IF([test "x$enable_double" = "xyes"],
      [MY_FC_REAL_DOUBLE
       FCFLAGS="$FCFLAGS -DDP"])

################################

AC_LANG_PUSH(Fortran)
AC_LINK_IFELSE(
  [AC_LANG_PROGRAM([], [dnl
      implicit none
      integer(8) :: a = 1
      real(8) :: b = 1.0d0
      
      a = transfer(b, a)
      print *, a
])], [my_check_have_transfer=yes], [my_check_have_transfer=no])

echo -n $my_check_have_transfer
AC_LANG_POP(Fortran)

AM_CONDITIONAL([USE_TRANSFER], [test "x$my_check_have_transfer" = "xyes"])

################################

AC_LANG_PUSH(Fortran)
AC_LINK_IFELSE(
  [AC_LANG_PROGRAM([], [dnl
      implicit none

      open(10,file='/tmp/configure.ermod.flush.test')
      write(10,*) 342
      flush(10)
      close(10)
])], [my_check_have_flush=yes], [my_check_have_flush=no])

echo -n $my_check_have_flush
AC_LANG_POP(Fortran)

AM_CONDITIONAL([USE_FLUSH], [test "x$my_check_have_flush" = "xyes"])

################################

# set to f90 as default
AC_FC_SRCEXT(f90)

# check whether mkl_dfti.f90 is accesible
# FIXME: This may fail to detect with gfortran 4.4.0-4.4.3
AS_IF([test "x$with_fft" = xmkl], [dnl
  AC_FC_SRCEXT(F90)
  AC_MSG_CHECKING([mkl_dfti.f90 is accessible])
  AC_COMPILE_IFELSE([#include "mkl_dfti.f90"
program main 
end program], [echo yes], [echo no
  AC_MSG_FAILURE([could not find mkl_dfti.f90 in include path])])
  AC_FC_SRCEXT(f90)])

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

# FIXME: check cpp flags

AC_ARG_ENABLE([defaults],
	AS_HELP_STRING([--disable-defaults],
		[(deprcated)]))

AS_IF([test x$enable_defaults != xno],
CFLAGS="$CFLAGS"
FCFLAGS="$FCFLAGS")

AS_IF([test "x$use_mpi" = "xyes"],
	    [mpi_result="enabled"
	     FC_CANONICAL="$FC"
	     FC="$MPIFC"
             FCFLAGS="-DMPI $FCFLAGS"],
	    [mpi_result="disabled"
	     FC_CANONICAL="$FC"])
AC_SUBST([FC_CANONICAL])

# optimization flags
AC_DEFUN([MY_FC_OPT_ACCEPT_NATIVE],[
AC_CACHE_CHECK([fortran compiler accepts -march=native],
my_fc_opt_accept_native_cv_flag,
[AC_LANG_PUSH(Fortran)
save_FCFLAGS=$FCFLAGS
flag="-march=native"
FCFLAGS="$save_FCFLAGS $flag"
AC_COMPILE_IFELSE([      program main
      end program], [my_fc_opt_accept_native=yes], [my_fc_opt_accept_native=no])
echo -n $my_fc_opt_accept_native
FCFLAGS=$save_FCFLAGS
AC_LANG_POP(Fortran)
])])

AC_ARG_ENABLE([opt],
	AS_HELP_STRING([--disable-opt],
		[disable automatic edit to optimization flags]))

AS_IF([test x$enable_opt = xyes || test x$enable_opt = x],
  [FCFLAGS="$FCFLAGS"
   AS_IF([test x$ac_compiler_gnu = xyes],
        [optflag="-O3 -ffast-math -fno-finite-math-only"
         MY_FC_OPT_ACCEPT_NATIVE
         AS_IF([test x$my_fc_opt_accept_native = xyes],
                optflag="$optflag -march=native")],
        [case x$FC in #(
           xmpiifort|xifort|xmpiifx|xifx)
             optflag="-O3 -no-prec-div -xHost"
             ;; #(
           *)
             optflag="-O"
             ;;
         esac])
   FCFLAGS="$FCFLAGS $optflag"])

# Check OpenMP
AS_IF([test "$enable_openmp" != "no"],
  [FCFLAGS="$FCFLAGS $OPENMP_FCFLAGS"
   # Check OpenMP version
   AC_MSG_CHECKING([whether OpenMP version >= 3.0])
   AC_COMPILE_IFELSE(
     [AC_LANG_PROGRAM([], [dnl
!\$omp task
  print *, "test"
!\$omp end task
])], 
     [AC_MSG_RESULT([yes])], 
     [AC_MSG_RESULT([no])
      AC_MSG_ERROR([OpenMP must be >= 3.0])])])

# AM_CONDITIONAL([MPI], [test x$use_mpi = xyes])
#AM_CONDITIONAL([USE_FFTW], [test x$with_fft = xfftw])
AM_CONDITIONAL([USE_MKL], [test x$with_fft = xmkl])
#AM_CONDITIONAL([USE_VMDPLUGIN], [test x$enable_vmdplugin = xyes || test x$enable_vmdplugin = x])

AS_IF([test "x$lapack_exist" != xyes],
	    [AC_MSG_ERROR(
		[dnl
LAPACK library was not found.
Try specifying --with-lapack=<lib> options or set FCFLAGS/FLIBS environment variables to fix the problem.])])

lapack_result="$LAPACK_LIBS $BLAS_LIBS"

AC_CONFIG_FILES([Makefile])
AC_CONFIG_HEADERS([config.h])
AC_OUTPUT

case "x/$mklroot" in
  */2020.?.?/*|*/2021.?.?/*|*/2022.[01].?/*)
    AS_IF([test "x$enable_openmp" = "xyes" -a "x$with_fft" = "xmkl"],
          [AC_MSG_ERROR([MKL -2022.1.0's FFT is suspected to have problem with OpenMP. Please choose one from (1) disabling OpenMP (recommended), (2) using newer MKL, or (3) compiling FFTW then adding --with-fft=fftw. Note ERmod works the best without OpenMP.])])
  ;;
esac

AC_MSG_RESULT([dnl
LAPACK: $lapack_result
OpenMP: $enable_openmp
FC=$FC_CANONICAL
FCFLAGS=$FCFLAGS
LDFLAGS=$LDFLAGS
LIBS=$LIBS

Configuration complete; type "make" to start compiling])

