# uncomment this to get a fully statically linked version
# NO_SHARED_LIBS = 1

# uncomment this to disable IPv6 support
# DO_IPV6 = 0

######################################################################
# YOU SHOULD NOT NEED TO TOUCH ANYTHING BELOW THIS LINE
######################################################################

# Standard part of Makefile for topdir.
TOPLEVEL_INCLUDED=YES

ifndef KERNEL_DIR
KERNEL_DIR=/usr/src/linux
endif
IPTABLES_VERSION:=1.2.10
OLD_IPTABLES_VERSION:=1.2.9

PREFIX:=/usr/local
LIBDIR:=$(PREFIX)/lib
BINDIR:=$(PREFIX)/sbin
MANDIR:=$(PREFIX)/man
INCDIR:=$(PREFIX)/include

# directory for new iptables releases
RELEASE_DIR:=/tmp

# Need libc6 for this.  FIXME: Should covert to autoconf.
ifeq ($(shell [ -f /usr/include/netinet/ip6.h ] && echo YES), YES)
DO_IPV6:=1
endif

COPT_FLAGS:=-O2
CFLAGS:=$(COPT_FLAGS) -Wall -Wunused -I$(KERNEL_DIR)/include -Iinclude/ -DIPTABLES_VERSION=\"$(IPTABLES_VERSION)\" #-g -DDEBUG #-pg # -DIPTC_DEBUG

ifdef NO_SHARED_LIBS
CFLAGS += -DNO_SHARED_LIBS=1
endif

EXTRAS+=iptables iptables.o iptables.8
EXTRA_INSTALLS+=$(DESTDIR)$(BINDIR)/iptables $(DESTDIR)$(MANDIR)/man8/iptables.8

# No longer experimental.
EXTRAS+=iptables-save iptables-restore
EXTRA_INSTALLS+=$(DESTDIR)$(BINDIR)/iptables-save $(DESTDIR)$(BINDIR)/iptables-restore $(DESTDIR)$(MANDIR)/man8/iptables-restore.8 $(DESTDIR)$(MANDIR)/man8/iptables-save.8

ifeq ($(DO_IPV6), 1)
EXTRAS+=ip6tables ip6tables.o ip6tables.8
EXTRA_INSTALLS+=$(DESTDIR)$(BINDIR)/ip6tables $(DESTDIR)$(MANDIR)/man8/ip6tables.8
EXTRAS_EXP+=ip6tables-save ip6tables-restore
EXTRA_INSTALLS_EXP+=$(DESTDIR)$(BINDIR)/ip6tables-save $(DESTDIR)$(BINDIR)/ip6tables-restore # $(DESTDIR)$(MANDIR)/man8/iptables-restore.8 $(DESTDIR)$(MANDIR)/man8/iptables-save.8 $(DESTDIR)$(MANDIR)/man8/ip6tables-save.8 $(DESTDIR)$(MANDIR)/man8/ip6tables-restore.8
endif

# Sparc64 hack
ifeq ($(shell uname -m),sparc64)
	POINTERTEST:=1
	32bituser := $(shell echo -e "\#include <stdio.h>\n\#if !defined(__sparcv9) && !defined(__arch64__) && !defined(_LP64)\nuserspace_is_32bit\n\#endif" | $(CC) $(CFLAGS) -E - | grep userspace_is_32bit)
	ifdef 32bituser
		# The kernel is 64-bit, even though userspace is 32.
		CFLAGS+=-DIPT_MIN_ALIGN=8 -DKERNEL_64_USERSPACE_32
	else
		EXT_LDFLAGS=-m elf64_sparc
	endif
endif

# Alpha only has 64bit userspace and fails the test below
ifeq ($(shell uname -m), alpha)
	POINTERTEST:=1
endif

# Generic test if arch wasn't found above
ifneq ($(POINTERTEST),1)
	# Try to determine if kernel is 64bit and we are compiling for 32bit
	ifeq ($(shell [ -a $(KERNEL_DIR)/include/asm ] && echo YES), YES)
		64bitkernel := $(shell echo -e "\#include <asm/types.h>\n\#if BITS_PER_LONG == 64\nkernel_is_64bits\n\#endif" | $(CC) $(CFLAGS) -D__KERNEL__ -E - | grep kernel_is_64bits)
		ifdef 64bitkernel
			32bituser := $(shell echo -e "\#include <stdio.h>\n\#if !defined(__arch64__) && !defined(_LP64)\nuserspace_is_32bit\n\#endif" | $(CC) $(CFLAGS) -E - | grep userspace_is_32bit)
			ifdef 32bituser
				CFLAGS+=-DIPT_MIN_ALIGN=8 -DKERNEL_64_USERSPACE_32
			endif
		endif
	else
		CFLAGS+=-D_UNKNOWN_KERNEL_POINTER_SIZE
	endif
endif

ifndef IPT_LIBDIR
IPT_LIBDIR:=$(LIBDIR)/iptables
endif

ifndef NO_SHARED_LIBS
DEPFILES = $(SHARED_LIBS:%.so=%.d)
SH_CFLAGS:=$(CFLAGS) -fPIC
STATIC_LIBS  =
STATIC6_LIBS =
LDFLAGS      = -rdynamic
LDLIBS       = -ldl -lnsl
else
DEPFILES = $(EXT_OBJS:%.o=%.d)
STATIC_LIBS  = extensions/libext.a
STATIC6_LIBS = extensions/libext6.a
LDFLAGS      = -static
LDLIBS       =
endif

.PHONY: default
default: print-extensions all

.PHONY: print-extensions
print-extensions:
	@[ -n "$(OPTIONALS)" ] && echo Extensions found: $(OPTIONALS)

iptables.o: iptables.c
	$(CC) $(CFLAGS) -DIPT_LIB_DIR=\"$(IPT_LIBDIR)\" -c -o $@ $<

iptables: iptables-standalone.c iptables.o $(STATIC_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIPT_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/iptables: iptables
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

iptables-save: iptables-save.c iptables.o $(STATIC_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIPT_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/iptables-save: iptables-save
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

iptables-restore: iptables-restore.c iptables.o $(STATIC_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIPT_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/iptables-restore: iptables-restore
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

ip6tables.o: ip6tables.c
	$(CC) $(CFLAGS) -DIP6T_LIB_DIR=\"$(IPT_LIBDIR)\" -c -o $@ $<

ip6tables: ip6tables-standalone.c ip6tables.o $(STATIC6_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIP6T_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/ip6tables: ip6tables
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

ip6tables-save: ip6tables-save.c ip6tables.o $(STATIC6_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIP6T_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/ip6tables-save: ip6tables-save
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

ip6tables-restore: ip6tables-restore.c ip6tables.o $(STATIC6_LIBS) libiptc/libiptc.a
	$(CC) $(CFLAGS) -DIP6T_LIB_DIR=\"$(IPT_LIBDIR)\" $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(DESTDIR)$(BINDIR)/ip6tables-restore: ip6tables-restore
	@[ -d $(DESTDIR)$(BINDIR) ] || mkdir -p $(DESTDIR)$(BINDIR)
	cp $< $@

$(DESTDIR)$(MANDIR)/man8/%.8: %.8
	@[ -d $(DESTDIR)$(MANDIR)/man8 ] || mkdir -p $(DESTDIR)$(MANDIR)/man8
	cp $< $@

EXTRA_DEPENDS+=iptables-standalone.d iptables.d

iptables-standalone.d iptables.d: %.d: %.c
	@-$(CC) -M -MG $(CFLAGS) $< | sed -e 's@^.*\.o:@$*.d $*.o:@' > $@

iptables.8: iptables.8.in extensions/libipt_matches.man extensions/libipt_targets.man
	sed -e '/@MATCH@/ r extensions/libipt_matches.man' -e '/@TARGET@/ r extensions/libipt_targets.man' iptables.8.in >iptables.8

ip6tables.8: ip6tables.8.in extensions/libip6t_matches.man extensions/libip6t_targets.man
	sed -e '/@MATCH@/ r extensions/libip6t_matches.man' -e '/@TARGET@/ r extensions/libiptt_targets.man' ip6tables.8.in >ip6tables.8

# Development Targets
.PHONY: install-devel-man3
install-devel-man3: $(DEVEL_MAN3)
	@[ -d $(DESTDIR)$(MANDIR)/man3 ] || mkdir -p $(DESTDIR)$(MANDIR)/man3
	@cp -v $(DEVEL_MAN3) $(DESTDIR)$(MANDIR)/man3

.PHONY: install-devel-headers
install-devel-headers: $(DEVEL_HEADERS)
	@[ -d $(DESTDIR)$(INCDIR) ] || mkdir -p $(DESTDIR)$(INCDIR)
	@cp -v $(DEVEL_HEADERS) $(DESTDIR)$(INCDIR)

.PHONY: install-devel-libs
install-devel-libs: $(DEVEL_LIBS)
	@[ -d $(DESTDIR)$(LIBDIR) ] || mkdir -p $(DESTDIR)$(LIBDIR)
	@cp -v $(DEVEL_LIBS) $(DESTDIR)$(LIBDIR)

.PHONY: install-devel
install-devel: all install-devel-man3 install-devel-headers install-devel-libs

.PHONY: distclean
distclean: clean
	@rm -f TAGS `find . -name '*~' -o -name '.*~'` `find . -name '*.rej'` `find . -name '*.d'` .makefirst

# Rusty's distro magic.
.PHONY: distrib
distrib: check distclean delrelease $(RELEASE_DIR)/iptables-$(IPTABLES_VERSION).tar.bz2 diff md5sums # nowhitespace

# Makefile must not define:
# -g -pg -DIPTC_DEBUG
.PHONY: check
check:
	@if echo $(CFLAGS) | egrep -e '-g|-pg|IPTC_DEBUG' >/dev/null; then echo Remove debugging flags; exit 1; else exit 0; fi

.PHONY: nowhitespace
nowhitespace:
	@if grep -n '[ 	]$$' `find . -name 'Makefile' -o -name '*.[ch]'`; then exit 1; else exit 0; fi

.PHONY: delrelease
delrelease:
	rm -f $(RELEASE_DIR)/iptables-$(IPTABLES_VERSION).tar.bz2

$(RELEASE_DIR)/iptables-$(IPTABLES_VERSION).tar.bz2:
	cd .. && ln -sf iptables iptables-$(IPTABLES_VERSION) && tar cvf - --exclude CVS iptables-$(IPTABLES_VERSION)/. | bzip2 -9 > $@ && rm iptables-$(IPTABLES_VERSION)

.PHONY: diff
diff: $(RELEASE_DIR)/iptables-$(IPTABLES_VERSION).tar.bz2
	@mkdir /tmp/diffdir
	@cd /tmp/diffdir && tar -x --bzip2 -f $(RELEASE_DIR)/iptables-$(IPTABLES_VERSION).tar.bz2
	@set -e; cd /tmp/diffdir; tar -x --bzip2 -f $(RELEASE_DIR)/iptables-$(OLD_IPTABLES_VERSION).tar.bz2; echo Creating patch-iptables-$(OLD_IPTABLES_VERSION)-$(IPTABLES_VERSION).bz2; diff -urN iptables-$(OLD_IPTABLES_VERSION) iptables-$(IPTABLES_VERSION) | bzip2 -9 > $(RELEASE_DIR)/patch-iptables-$(OLD_IPTABLES_VERSION)-$(IPTABLES_VERSION).bz2
	@rm -rf /tmp/diffdir

.PHONY: md5sums
md5sums:
	cd $(RELEASE_DIR)/ && md5sum patch-iptables-*-$(IPTABLES_VERSION).bz2 iptables-$(IPTABLES_VERSION).tar.bz2

# $(wildcard) fails wierdly with make v.3.78.1.
include $(shell echo */Makefile)
include Rules.make
