AC_INIT(netcf, 0.2.3)
AC_CONFIG_SRCDIR([src/netcf.c])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([gnulib/m4])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([-Wno-portability 1.11 color-tests parallel-tests])
AM_SILENT_RULES([yes]) # make --enable-silent-rules the default.
AC_CANONICAL_HOST

AC_SUBST([LIBNETCF_VERSION_INFO], [5:0:4])

AC_GNU_SOURCE

AC_PROG_CC
gl_EARLY

AC_PROG_LIBTOOL

dnl Compiler flags to be used everywhere
AC_SUBST([NETCF_CFLAGS], ['--std=gnu99 -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fasynchronous-unwind-tables'])

gl_INIT

NETCF_COMPILE_WARNINGS

dnl Need to test if pkg-config exists
PKG_PROG_PKG_CONFIG
PKG_CHECK_MODULES([LIBXML], [libxml-2.0])
PKG_CHECK_MODULES([LIBXSLT], [libxslt])
PKG_CHECK_MODULES([LIBEXSLT], [libexslt])

NETCF_CHECK_READLINE

AC_CHECK_HEADER([pthread.h],
	[AC_CHECK_LIB([pthread],[pthread_join],[
		AC_DEFINE([HAVE_LIBPTHREAD],[],[Define if pthread (-lpthread)])
		AC_DEFINE([HAVE_PTHREAD_H],[],[Define if <pthread.h>])
		LIBS="-lpthread $LIBS"
	])])

dnl if --prefix is /usr, don't use /usr/var for localstatedir
dnl or /usr/etc for sysconfdir
dnl as this makes a lot of things break in testing situations

if test "$prefix" = "/usr" && test "$localstatedir" = '${prefix}/var' ; then
    localstatedir='/var'
fi
if test "$prefix" = "/usr" && test "$sysconfdir" = '${prefix}/etc' ; then
    sysconfdir='/etc'
fi

dnl
dnl network driver flavor
dnl
AC_ARG_WITH([driver],
            [AS_HELP_STRING([--with-driver],
                            [network driver name])],
            [],[with_driver=check])

AC_MSG_CHECKING([network driver name])
if test "x$with_driver" = "xcheck" ; then
  case "$host" in
	*-*-mingw*)
          with_driver=mswindows
	;;
  esac
fi
if test "x$with_driver" = "xcheck" && test -f /etc/redhat-release ; then
  with_driver=redhat
fi
if test "x$with_driver" = "xcheck" && test -f /etc/fedora-release ; then
  with_driver=redhat
fi
if test "x$with_driver" = "xcheck" && test -f /etc/debian_version ; then
  with_driver=debian
fi
if test "x$with_driver" = "xcheck" && test -f /etc/ubuntu_version ; then
  with_driver=debian
fi
if test "x$with_driver" = "xcheck" && (test -f /etc/suse-release || test -f /etc/SuSE-release) ; then
  with_driver=suse
fi
if test "x$with_driver" = "xcheck" ; then
  AC_MSG_ERROR([Cannot detect network driver, use --with-driver=NAME to select implementation])
fi
AC_MSG_RESULT([$with_driver])
NETCF_DRIVER=$with_driver
AC_SUBST([NETCF_DRIVER])
AC_DEFINE_UNQUOTED([NETCF_DRIVER], $NETCF_DRIVER, [The network driver])
AM_CONDITIONAL([NETCF_DRIVER_REDHAT], test "x$with_driver" = "xredhat")
AM_CONDITIONAL([NETCF_DRIVER_DEBIAN], test "x$with_driver" = "xdebian")
AM_CONDITIONAL([NETCF_DRIVER_SUSE], test "x$with_driver" = "xsuse")
AM_CONDITIONAL([NETCF_DRIVER_MSWINDOWS], test "x$with_driver" = "xmswindows")

if test "x$with_driver" = "xredhat"; then
    AC_DEFINE_UNQUOTED([NETCF_TRANSACTION],
                       [LIBEXECDIR "/netcf-transaction.sh"],
                       [Location of the netcf-transaction shell script])
fi

dnl
dnl system init flavor
dnl
AC_MSG_CHECKING([for system init flavor])
AC_ARG_WITH([sysinit],
  [AS_HELP_STRING([--with-sysinit@<:@=STYLE@:>@],
    [Style of init script to install: initscripts, systemd,
     check, none @<:@default=check@:>@])],
  [],[with_sysinit=check])

case "$with_sysinit" in
    systemd)
       ;;
    initscripts)
       ;;
    none)
       ;;
    check)
       if test "$cross_compiling" != "yes" && \
          test "$with_driver" = "redhat"; then
         if test -d /etc/systemd; then
           with_sysinit=systemd
         else
           with_sysinit=initscripts
         fi
       else
         with_sysinit=none
       fi
       ;;
    *)
       AC_MSG_ERROR([Unknown system initscript flavor $with_sysinit])
    ;;
esac
if test "x$with_sysinit" = "xcheck" ; then
  with_sysinit=none
fi

AM_CONDITIONAL([NETCF_USE_INITSCRIPTS], test "$with_sysinit" = "initscripts")
AM_CONDITIONAL([NETCF_USE_SYSTEMD], test "$with_sysinit" = "systemd")
AM_CONDITIONAL([NETCF_TRANSACTION_SUPPORT], test "$with_sysinit" != "none")
AC_MSG_RESULT($with_sysinit)
if test "$with_sysinit" != "none" && test "$with_driver" != "redhat"; then
  AC_MSG_ERROR([netcf does not have support for $with_sysinit combined with the $with_driver driver])
fi

AC_ARG_WITH([libnl1],
            AS_HELP_STRING([--with-libnl1], [Force usage of libnl1 @<:@default=no@:>@]),
	      [force_libnl1=${withval}], [force_libnl1=no])

if test "x$with_driver" != "xmswindows"
then
	PKG_CHECK_MODULES([LIBAUGEAS], [augeas >= 0.5.0])

	have_libnl="no"
	if (test "${force_libnl1}" = "no"); then
		PKG_CHECK_MODULES([LIBNL], [libnl-3.0],
				  [have_libnl3=yes],
				  [have_libnl3=no])
		PKG_CHECK_MODULES([LIBNL_ROUTE3], [libnl-route-3.0],
				  [have_libnl_route3=yes],
				  [have_libnl_route3=no])
	fi

	if (test "${have_libnl3}" = "yes" -a "${have_libnl_route3}" = "yes"); then
		AC_DEFINE([HAVE_LIBNL3], [1], [Use libnl-3.0])
		have_libnl="yes"
	else
		PKG_CHECK_MODULES([LIBNL], [libnl-1],
				  [have_libnl1=yes],
				  [have_libnl1=no])
		if (test "${have_libnl1}" = "yes"); then
			AC_DEFINE([HAVE_LIBNL], [1], [Use libnl-1])
			have_libnl="yes"
		else
			dnl libnl version 2.x is not supported by netcf
			PKG_CHECK_MODULES([LIBNL], [libnl-2.0],
				[AC_MSG_ERROR([libnl-2.x is unsupported, please install libnl-3.x])])
		fi

	fi

	if (test "${have_libnl}" = "no"); then
		AC_MSG_ERROR([missing dependency: libnl library not installed])
	fi
fi

if test "x$with_driver" = "xdebian"; then
	dnl Check for ifup and ifdown in debian
	AC_PATH_PROG(IFDOWN, ifdown, false, $PATH$PATH_SEPARATOR/sbin)
	if test "$IFDOWN" = "false"; then
	   AC_MSG_ERROR([ifdown not found])
	fi
	AC_DEFINE_UNQUOTED([IFDOWN], "$IFDOWN", [path to ifdown binary])

	AC_PATH_PROG(IFUP, ifup, false, $PATH$PATH_SEPARATOR/sbin)
	if test "$IFUP" = "false"; then
	   AC_MSG_ERROR([ifup not found])
	fi
	AC_DEFINE_UNQUOTED([IFUP], "$IFUP", [path to ifup binary])
fi

NETCF_LIBDEPS=$(echo $LIBAUGEAS_LIBS $LIBEXSLT_LIBS $LIBXSLT_LIBS $LIBXML_LIBS $LIBNL_LIBS)
AC_SUBST([NETCF_LIBDEPS])

AC_OUTPUT(Makefile                                          \
          gnulib/lib/Makefile                               \
          gnulib/tests/Makefile                             \
          src/Makefile                                      \
          tests/Makefile                                    \
          doc/Makefile                                      \
          netcf.pc netcf.spec mingw32-netcf.spec)
