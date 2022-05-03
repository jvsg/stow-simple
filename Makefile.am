# This file is part of GNU Stow.
#
# GNU Stow is free software: you can redistribute it and/or modify it
# under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# GNU Stow is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program. If not, see https://www.gnu.org/licenses/.

## Process this file with Automake to produce Makefile.in

bin_SCRIPTS = bin/stow bin/chkstow
dist_man_MANS = doc/stow.8
dist_doc_DATA = \
	README.md INSTALL.md \
	ChangeLog doc/ChangeLog.OLD

# automake magic to define where *_DATA files get installed:
pmdir = $(PMDIR)
pmstowdir = $(pmdir)/Stow

pm_DATA = lib/Stow.pm
pmstow_DATA = lib/Stow/Util.pm

AM_MAKEINFOFLAGS = -I $(srcdir)

# We require this -I parameter to ensure that the include of the
# default ignore list in the manual works.  Unfortunately this is
# the only way to do it:
#
#   http://article.gmane.org/gmane.comp.sysutils.automake.bugs/4334/match=passing+parameters
#
# even though it annoyingly produces a warning with the -Wall option
# to AM_INIT_AUTOMAKE which has to be silenced via -Wno-override.

DEFAULT_IGNORE_LIST = $(srcdir)/default-ignore-list

TESTS_DIR = $(srcdir)/t
TESTS_OUT = tmp-testing-trees
TESTS_ENVIRONMENT = $(PERL) -Ibin -Ilib -I$(TESTS_DIR)

# This is a kind of hack; TESTS needs to be set to ensure that the
# `check-am' target makes check-TESTS, but we override check-TESTS
# so it doesn't really matter what it's set to, as long as it already
# exists (otherwise automake will try to build it).
TESTS = t

# GNU autotools standardised on the 'check' target, but CPAN (and the
# rest of the world) standardised on the 'test' target.
test: check

# required in vpath mode to ensure $build/t/ exists
check_DATA = $(TESTS_OUT)

# Test::Harness produces cleaner output than automake's default test
# harness, albeit without the pretty colours provided by the
# `color-tests' AM_INIT_AUTOMAKE option.  This also dodges having to
# set TESTS to the full list of tests, which is good because automake
# doesn't support wildcards, and so it would be too easy to forget to
# add a new one to the list.
#
# Note that automake's `check' rule cannot be overridden
# for some weird reason:
#
#   http://thread.gmane.org/gmane.comp.sysutils.automake.general/13040/focus=13041
#
# so we override check-TESTS instead which is where the real work is
# done anyway.  Unfortunately this produces a warning with the -Wall
# option to AM_INIT_AUTOMAKE which has to be silenced via
# -Wno-override.
check-TESTS:
	dir=$(TESTS_DIR); \
	$(TESTS_ENVIRONMENT) -MTest::Harness -e 'runtests(@ARGV)' "$${dir#./}"/*.t

$(TESTS_OUT):
	mkdir -p $@

CPAN_FILES = MANIFEST MANIFEST.SKIP Build.PL META.yml META.json
EXTRA_DIST = \
	bin/stow.in bin/chkstow.in lib/Stow.pm.in lib/Stow/Util.pm.in \
	$(TESTS) t/testutil.pm \
	$(DEFAULT_IGNORE_LIST) \
	$(CPAN_FILES)
CLEANFILES = $(bin_SCRIPTS) $(pm_DATA) $(pmstow_DATA)
DISTCLEANFILES = Makefile.in configure Build MYMETA.*
MAINTAINERCLEANFILES = $(dist_man_MANS) $(HTML) $(PDF) ChangeLog

# clean up auto-generated files
clean-local:
	-rm -rf $(TESTS_OUT)
maintainer-clean-local:
	-rm -rf doc/manual-split cover_db

# this is more explicit and reliable than the config file trick
edit = sed -e 's|[@]PERL[@]|$(PERL)|g'       \
	   -e 's|[@]VERSION[@]|$(VERSION)|g' \
	   -e "s|[@]USE_LIB_PMDIR[@]|$$use_lib_pmdir|g"

pmdir_in_INC = \
	PERL5LIB= $(PERL) -V | \
	    awk '/@INC/ {p=1; next} p==1 {print $$1}' | \
	    grep -q "$(pmdir)"

calc_use_lib_pmdir = \
	if $(pmdir_in_INC); then \
	    use_lib_pmdir=""; \
	else \
	    use_lib_pmdir="use lib \"$(pmdir)\";"; \
	fi

check_pmdir = \
	echo; \
	echo "\# Perl modules will be installed to $(pmdir)"; \
	echo "\# "; \
	if $(pmdir_in_INC); then \
	    echo "\# This is in $(PERL)'s built-in @INC, so everything"; \
	    echo "\# should work fine with no extra effort."; \
	else \
	    echo "\# This is not in $(PERL)'s built-in @INC, so the"; \
	    echo "\# front-end scripts will have an appropriate \"use lib\""; \
	    echo "\# line inserted to compensate."; \
	fi; \
	echo

bin/stow: bin/stow.in Makefile.am
	[ -d bin ] || mkdir bin # required in vpath mode
	@$(check_pmdir)
	@$(calc_use_lib_pmdir); \
	$(edit) < $< > $@
	@echo "Generated $@ from $<"
	chmod +x $@

bin/chkstow: bin/chkstow.in Makefile.am
	@[ -d bin ] || mkdir bin # required in vpath mode
	@$(edit) < $< > $@
	@echo "Generated $@ from $<"
	chmod +x $@

lib/Stow.pm: lib/Stow.pm.in $(DEFAULT_IGNORE_LIST) Makefile.am
	@[ -d lib ] || mkdir lib # required in vpath mode
	@( $(edit) < $<; cat $(DEFAULT_IGNORE_LIST) ) > $@
	@echo "Generated $@ from $< and $(DEFAULT_IGNORE_LIST)"

lib/Stow/Util.pm: lib/Stow/Util.pm.in Makefile.am
	@[ -d lib/Stow ] || mkdir -p lib/Stow # required in vpath mode
	@$(edit) < $< > $@
	@echo "Generated $@ from $<"

##############################################################################
# The below rules should only be needed by developers.
##############################################################################

cpanm:
	cpanm --quiet --installdeps --notest .; \
	CPANM_RESULT=$$?; \
	if [ $$CPANM_RESULT != 0 ]; then \
	    echo ---------------------------------------------------; \
	    cat ~/.cpanm/build.log; \
	    echo ---------------------------------------------------; \
	    exit $$CPANM_RESULT; \
	fi

doc/stow.8: bin/stow.in Makefile.am
	[ -d doc ] || mkdir doc # required in vpath mode
	$(edit) < $< | pod2man --name stow --section 8 > $@

dist-hook: $(dist_man_MANS)
## If we are creating a distribution from a git checkout, ensure
## the ChangeLog file is in sync the git repository.
	if test -d $(top_srcdir)/.git; then \
	  rm -f ChangeLog \
	    && $(MAKE) $(AM_MAKEFLAGS) ChangeLog \
	    && cp -f ChangeLog $(distdir)/ChangeLog; \
	fi

ChangeLog: doc/ChangeLog.OLD
	@if [ -d .git ]; then \
	    ( \
	        git log \
		    --format="format:%ad  %aN <%aE>%n%n    * %w(70,0,4)%s%+b%n" \
		    --name-status \
		    v2.0.2..HEAD \
                | sed 's/^\([A-Z]\)\t/      \1 /'; \
	        cat $< \
	    ) > $@; \
	    echo "Rebuilt $@ from git commit history."; \
	else \
	    echo "Not in a git repository; can't update ChangeLog."; \
	fi
