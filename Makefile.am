ACLOCAL_FLAGS = -I m4
ACLOCAL_AMFLAGS = -I m4

SUBDIRS = src ebin include priv

APPDIRS = applications/chat applications/mail applications/wiki applications/yapp

if !WINDOWS
  # don't c-compile anything on win32 (yet)
  # I don't know how to make a linked in driver using gcc, or any other compiler
  # there, and have no intentions of finding out
  SUBDIRS += c_src
endif

if MINGW
  SUBDIRS += win32
endif

SUBDIRS += examples/src	examples/ebin examples/include \
	   test doc man scripts www/code www/shoppingcart

EXTRA_DIST = ChangeLog known_dialyzer_warnings LICENSE			\
	     README README.osx README.pkg-config 			\
	     rebar.config rebar.config.script two-mode-mode.el vsn.mk	\
	     include.mk erlang_deps.mk ssl munin contrib www rel

if !MINGW
pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = yaws.pc
endif

all-local:
clean-local:
	$(AM_V_at)rm -f dialyzer_warnings

install-data-local: all
	$(AM_V_at)$(INSTALL) -d $(DESTDIR)$(wwwdir)
	$(AM_V_at)(cd @top_srcdir@; tar cf - www ) | (cd $(DESTDIR)$(wwwdir)/..; tar xf - )
	$(AM_V_at)chmod -R u+rw $(DESTDIR)$(wwwdir)

	$(AM_V_at)$(INSTALL) -d $(DESTDIR)$(logdir)

	$(AM_V_at)$(INSTALL) -d $(DESTDIR)$(sysconfdir)/yaws
	$(AM_V_at)$(INSTALL) -m 644 @srcdir@/ssl/yaws-cert.pem $(DESTDIR)$(sysconfdir)/yaws
	$(AM_V_at)$(INSTALL) -m 644 @srcdir@/ssl/yaws-key.pem $(DESTDIR)$(sysconfdir)/yaws

	@echo "-------------------------------"
	@echo
	@echo "** etc files went into        $(sysconfdir)"
	@echo "** executables went into      $(bindir)"
	@echo "** library files went into    $(ERLANG_INSTALL_LIB_DIR_yaws)"
	@echo "** logs went into             $(logdir)"
	@echo "** doc files went into        $(docdir)"
	@echo "** man files went into        $(mandir)"
	@echo "** default docroot went into  $(wwwdir)"

	$(AM_V_at)if test $(yawsdir) != $(ERLANG_INSTALL_LIB_DIR_yaws); then	\
	  $(INSTALL) -d $(DESTDIR)$(libdir);					\
	  rm -f $(DESTDIR)$(libdir)/yaws;					\
	  $(LN_S) -f $(ERLANG_INSTALL_LIB_DIR_yaws) $(DESTDIR)$(libdir)/yaws;	\
	  echo;									\
	  echo "create link $(ERLANG_INSTALL_LIB_DIR_yaws) -> $(yawsdir)";	\
	fi

	@echo
	@echo "--------------------------------"

uninstall-local:
	$(AM_V_at)(cd @top_srcdir@/www && find * -type f -exec rm -f $(DESTDIR)$(wwwdir)/{} \;)
	$(AM_V_at)(cd @top_srcdir@/www && find * -type d -exec rm -fr $(DESTDIR)$(wwwdir)/{} \;)
	$(AM_V_at)[ ! -h $(yawsdir) ] || rm -f $(DESTDIR)$(libdir)/yaws
	$(AM_V_at)rm -r $(DESTDIR)$(sysconfdir)/yaws/yaws-cert.pem
	$(AM_V_at)rm -r $(DESTDIR)$(sysconfdir)/yaws/yaws-key.pem

debug:
	$(MAKE) DEBUG_ERLC_FLAGS=" -Ddebug" CFLAGS="-Werror -Wall -O0 -g"

docs: doc

doc:
	$(AM_V_at)( cd doc && $(MAKE) doc )

test: check

cleantest:
	$(AM_V_at)( cd test && $(MAKE) clean )

yaws.plt:
	$(AM_V_GEN)dialyzer --build_plt -r ebin --output_plt yaws.plt	\
	  -r $(ERLANG_LIB_DIR_kernel) 					\
	  -r $(ERLANG_LIB_DIR_stdlib) 					\
	  -r $(ERLANG_LIB_DIR_sasl) 					\
	  -r $(ERLANG_LIB_DIR_erts)					\
	  -r $(ERLANG_LIB_DIR_compiler)					\
	  -r $(ERLANG_LIB_DIR_crypto)					\
	  -r $(ERLANG_LIB_DIR_xmerl)

dialyzer: all yaws.plt
	$(AM_V_at)$(DIALYZER) -q --plt yaws.plt -r ebin > dialyzer_warnings || true
	$(AM_V_at)if diff -U0 @srcdir@/known_dialyzer_warnings dialyzer_warnings ; then	\
	  echo dialyzer check passed ;							\
	  exit 0 ; 									\
	else 										\
	  exit 1 ;									\
	fi

dist-hook:
	$(AM_V_at)for a in $(APPDIRS); do			\
	  $(INSTALL) -d $(distdir)/$$a;				\
	  (cd $$a && $(MAKE) top_distdir=../../$(distdir)	\
	    distdir=../../$(distdir)/$$a distdir);		\
	done

distclean-local:
	$(AM_V_at)[ ! -h include/yaws.hrl ]     || rm -f include/yaws.hrl
	$(AM_V_at)[ ! -h include/yaws_api.hrl ] || rm -f include/yaws_api.hrl
	$(AM_V_at)[ ! -h priv/mime.types ]      || rm -f priv/mime.types
	$(AM_V_at)for a in $(APPDIRS); do	\
	  (cd $$a && $(MAKE) distclean);	\
	done


mkinstaller: cleaninstaller
	$(AM_V_at)if [ -z $(UNIX2DOS) -a -z $(TODOS) ]; then	\
	  echo "No unix2dos/todos program found";		\
	  echo "Run apt-get install dos2unix/tofrodos";		\
	  exit 1;						\
	fi

	$(AM_V_at)if [ ! -d $(INSTALL_BUILDER) ]; then			\
	  echo "No installer-builder found";				\
	  echo "For more information, read win32/README.developer";	\
	  exit 1;							\
	fi

	$(AM_V_at)$(MAKE) distdir
	$(AM_V_at)$(MKDIR_P) $(distdir)/_build
	$(AM_V_at)(cd $(distdir)/_build && ../configure				\
	  --host i586-mingw32msvc --prefix=/ --docdir=/doc --mandir=/man	\
	  wwwdir=/www logdir=/logs yawsdir=/ confdir=/ appsdir=/applications	\
	  yappdir=/applications/yapp INSTALL_BUILDER=$(INSTALL_BUILDER))
	$(AM_V_at)(cd $(distdir)/_build && $(MAKE) install DESTDIR=@abs_builddir@/$(distdir)/_inst)
	$(AM_V_at)(cd $(distdir)/_build && $(MAKE) installapps DESTDIR=@abs_builddir@/$(distdir)/_inst)
	$(AM_V_at)(cd $(distdir)/_build/win32 && $(MAKE) mkinstaller DESTDIR=@abs_builddir@/$(distdir)/_inst)
	$(AM_V_at)cp $(distdir)/*.exe @builddir@
	$(AM_V_at)rm -fr $(distdir)

cleaninstaller:
	$(AM_V_at)rm -f *.exe
	$(AM_V_at)rm -fr $(distdir)


apps: $(APPDIRS)


$(APPDIRS):
	$(AM_V_at)(cd $@ && $(MAKE) all)

cleanapps:
	$(AM_V_at)for a in $(APPDIRS); do	\
	  (cd $$a && $(MAKE) clean);		\
	done

installapps: apps
	$(AM_V_at)$(INSTALL) -d $(DESTDIR)$(appsdir)
	$(AM_V_at)for a in $(APPDIRS); do	\
	  (cd $$a && $(MAKE) install);		\
	done

uninstallapps: apps
	$(AM_V_at)for a in $(APPDIRS); do	\
	  (cd $$a && $(MAKE) uninstall);	\
	done

fullinstall: install installapps
fulluninstall: uninstall uninstallapps

fullclean: clean cleantest cleaninstaller cleanapps

.PHONY: test doc $(APPDIRS)

# Local Variables:
#    tab-width: 8
# End: