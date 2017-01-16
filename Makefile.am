#
# Project Wok
#
# Copyright IBM Corp, 2013-2016
#
# Code derived from Project Kimchi
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

SUBDIRS = src ui docs contrib po tests

man_MANS = docs/wokd.8

AUTOMAKE_OPTIONS = foreign

ACLOCAL_AMFLAGS = --install -I m4

EXTRA_DIST = \
	config.rpath \
	autogen.sh \
	COPYING.ASL2 \
	COPYING.LGPL \
	CONTRIBUTE.md \
	VERSION \
	build-aux/pkg-version \
	check_ui_code_errors.sh \
	check_spec_errors.sh \
	$(NULL)


PEP8_BLACKLIST = *src/wok/plugins,*src/wok/config.py,*src/wok/i18n.py,*tests/test_config.py

SKIP_PYFLAKES_ERR = "\./src/wok/websocket\.py"

I18N_FILES = src/wok/i18n.py $(NULL)

# Link built mo files in the source tree to enable use of translations from
# within the source tree
all-local:
	while read L && test -n "$$L"; do                  \
		dir=mo/$$L/LC_MESSAGES ;                       \
		$(MKDIR_P) $$dir ;                             \
		ln -sf ../../../po/$$L.gmo $$dir/wok.mo ;   \
	done < po/LINGUAS

#
# Packaging helpers
#

install-deb: install
	cp -R $(top_srcdir)/contrib/DEBIAN $(DESTDIR)/
	$(MKDIR_P) $(DESTDIR)/$(systemdsystemunitdir); \
	cp -R contrib/wokd.service.systemd \
		$(DESTDIR)/$(systemdsystemunitdir)/wokd.service; \
	$(MKDIR_P) $(DESTDIR)/etc/init; \
	cp -R $(top_srcdir)/contrib/wokd-upstart.conf.debian \
		$(DESTDIR)/etc/init/wokd.conf; \
	$(MKDIR_P) $(DESTDIR)/$(localstatedir)/lib/wok/
	$(MKDIR_P) $(DESTDIR)/usr/lib/firewalld/services
	cp -R $(top_srcdir)/src/firewalld.xml \
		$(DESTDIR)/usr/lib/firewalld/services/wokd.xml


deb: contrib/make-deb.sh
	$(top_srcdir)/contrib/make-deb.sh

wok.spec: contrib/wok.spec.fedora contrib/wok.spec.suse
	@if test -e /etc/redhat-release; then                   \
		ln -sf contrib/wok.spec.fedora $@ ;              \
        elif test -e /etc/system-release; then			\
		ln -sf contrib/wok.spec.fedora $@ ;		\
	elif test -e /etc/SuSE-release; then                    \
		ln -sf contrib/wok.spec.suse $@ ;                \
	else                                                    \
		echo "Unable to select a spec file for RPM build" ; \
		/bin/false ;                                        \
	fi

rpm: dist wok.spec
	$(MKDIR_P) rpm/BUILD rpm/RPMS rpm/SOURCES rpm/SPECS rpm/SRPMS
	cp $(top_srcdir)/wok.spec rpm/SPECS/wok.spec
	cp $(DIST_ARCHIVES) rpm/SOURCES
	rpmbuild -ba --define "_topdir `pwd`/rpm" rpm/SPECS/wok.spec

fedora-rpm: dist contrib/wok.spec.fedora
	ln -sf contrib/wok.spec.fedora wok.spec
	$(MAKE) rpm

suse-rpm: dist contrib/wok.spec.suse
	ln -sf contrib/wok.spec.suse wok.spec
	$(MAKE) rpm

ChangeLog:
	@if $(GIT) rev-parse &> /dev/null ; then                     \
		$(top_srcdir)/build-aux/genChangelog --release > $@; \
	fi

install-data-local:
	mkdir -p $(DESTDIR)/$(systemdsystemunitdir); \
	$(INSTALL_DATA) contrib/wokd.service.systemd $(DESTDIR)/$(systemdsystemunitdir)/wokd.service; \
	mkdir -p $(DESTDIR)/etc/init.d/; \
	$(INSTALL_DATA) contrib/wokd.sysvinit $(DESTDIR)/etc/init.d/wok; \
	chmod +x $(DESTDIR)/etc/init.d/wok; \
	mkdir -p $(DESTDIR)/usr/lib/firewalld/services/;
	$(INSTALL_DATA) src/firewalld.xml $(DESTDIR)/usr/lib/firewalld/services/wokd.xml;
	mkdir -p $(DESTDIR)/$(localstatedir)/lib/wok/
	mkdir -p $(DESTDIR)/$(localstatedir)/log/wok/
	touch $(DESTDIR)/$(localstatedir)/log/wok/wok-access.log
	touch $(DESTDIR)/$(localstatedir)/log/wok/wok-error.log
	mkdir -p $(DESTDIR)/etc/logrotate.d/
	$(INSTALL_DATA) $(top_srcdir)/src/wok.logrotate $(DESTDIR)/etc/logrotate.d/wokd
	mkdir -p $(DESTDIR)/etc/nginx/sites-available
	$(INSTALL_DATA) $(top_srcdir)/src/nginx/wok.conf $(DESTDIR)/etc/nginx/sites-available/wok.conf

uninstall-local:
	$(RM) $(DESTDIR)/$(systemdsystemunitdir)/wokd.service; \
	$(RM) $(DESTDIR)/etc/init.d/wok; \
	$(RM) $(DESTDIR)/etc/init/wok.conf; \
	$(RM) $(DESTDIR)/usr/lib/firewalld/services/wokd.xml
	rmdir $(DESTDIR)/usr/lib/firewalld/services && rmdir $(DESTDIR)/usr/lib/firewalld || :
	$(RM) -rf $(DESTDIR)/$(localstatedir)/lib/wok
	$(RM) -rf $(DESTDIR)/$(localstatedir)/log/wok
	$(RM) -rf $(DESTDIR)/etc/wok
	$(RM) $(DESTDIR)/etc/nginx/sites-available/wok.conf
	$(RM) $(DESTDIR)/etc/logrotate.d/wokd

VERSION:
	@if $(GIT) rev-parse &> /dev/null ; then              \
		git describe --abbrev=0 --always > $@;        \
	fi

.PHONY: deb install-deb rpm fedora-rpm suse-rpm VERSION


clean-local:
	rm -rf mo rpm

CLEANFILES = wok.spec `find "$(top_srcdir)" -type f -name "*.pyc" -print`
