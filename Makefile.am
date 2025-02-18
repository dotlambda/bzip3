ACLOCAL_AMFLAGS = -I build-aux

AM_CFLAGS = -I$(top_srcdir)/include
AM_LDFLAGS = -avoid-version

EXTRA_DIST = LICENSE PORTING.md README.md build-aux/git-version-gen

pkgconfig_DATA = bzip3.pc

include_HEADERS = include/libbz3.h
noinst_HEADERS = include/cm.h \
						 include/common.h \
						 include/crc32.h \
						 include/libsais.h \
						 include/lzp.h \
						 include/rle.h

lib_LTLIBRARIES = libbzip3.la
libbzip3_la_SOURCES = src/cm.c \
					  src/crc32.c \
					  src/libbz3.c \
					  src/libsais.c \
					  src/lzp.c \
					  src/rle.c

bin_PROGRAMS = bzip3
bzip3_CFLAGS = $(AM_CFLAGS)
bzip3_LDDADD = libbzip3.la
bzip3_SOURCES = src/main.c $(libbzip3_la_SOURCES)

CLEANFILES = $(bin_PROGRAMS)

# End standard generic autotools stuff

# Begin special handling for autoconf VERSION being updated on commit

BUILT_SOURCES = .version
CLEANFILES += $(BUILT_SOURCES) .version-prev

src/bzip3-main.$(OBJEXT): .version

_BRANCH_REF != $(AWK) '{print ".git/" $$2}' .git/HEAD 2>/dev/null ||:

.version: $(_BRANCH_REF)
	@if [ -e "$(srcdir)/.tarball-version" ]; then \
		printf "$(VERSION)" > $@; \
	else \
		touch "$@-prev"; \
		if [ -e "$@" ]; then \
			cp "$@" "$@-prev"; \
		fi; \
		./build-aux/git-version-gen "$(srcdir)/.tarball-version" > $@; \
		cmp -s "$@" "$@-prev" || autoreconf configure.ac --force; \
	fi

dist-hook:
	printf "$(VERSION)" > "$(distdir)/.tarball-version"

# Begin developer convenience targets

.PHONY: format
format: $(bzip3_SOURCES) $(include_HEADERS) $(noinst_HEADERS)
	clang-format -i $^

.PHONY: cloc
cloc: $(bzip3_SOURCES) $(include_HEADERS) $(noinst_HEADERS)
	cloc $^

CLEANFILES += LICENSE2
.PHONY: roundtrip

BZIP3 := bzip3$(EXEEXT)

roundtrip: $(BZIP3)
	rm -f LICENSE2
	./$(BZIP3) -e -b 6 LICENSE
	./$(BZIP3) -d LICENSE.bz3 LICENSE2
	cmp LICENSE LICENSE2
