
SHELL = /bin/sh
NOOP = $(SHELL) -c true
NOECHO = @
ECHO = echo
MKDIR = /bin/mkdir
CP = cp
SED = /bin/sed
CHMOD = chmod
RM = /bin/rm

# some of these are pretty questionable.
PERL = /usr/bin/perl
# TEST_FILES = *.t
TEST_FILES = database_dependent.pl
TEST_CLASS =
RUN_EXPENSIVE_TESTS = 0
PROVE = /usr/bin/prove
PROVE_FLAGS = -v
PERL5LIB = ..
KOHA_CONF_DIR = ../etc
CONF_FILE_TEMPLATE = $(KOHA_CONF_DIR)/koha-conf.xml 
TEST_CONF_FILE = run/etc/koha-conf.xml
MKPATH = $(PERL) "-MExtUtils::Command" -e mkpath

TEST_REWRITE_SCRIPT = rewrite-config-test.PL
REAL_REWRITE_SCRIPT = ../rewrite-config.PL

ZEBRA_CONF_DIR = run/etc/zebradb
ZEBRA_CONF_FILES = $(ZEBRA_CONF_DIR)/etc/passwd $(ZEBRA_CONF_DIR)/zebra-biblios.cfg $(ZEBRA_CONF_DIR)/zebra-authorities.cfg $(ZEBRA_CONF_DIR)/zebra-authorities-dom.cfg $(ZEBRA_CONF_DIR)/explain-authorities.xml $(ZEBRA_CONF_DIR)/explain-biblios.xml $(ZEBRA_CONF_DIR)/retrieval-info-auth-grs1.xml $(ZEBRA_CONF_DIR)/retrieval-info-auth-dom.xml $(ZEBRA_CONF_DIR)/ccl.properties $(ZEBRA_CONF_DIR)/cql.properties $(ZEBRA_CONF_DIR)/pqf.properties

SCRIPTS = koha-zebra-ctl.sh koha-pazpar2-ctl.sh koha-index-daemon-ctl.sh
SRC_SCRIPT_DIR = ../misc/bin
TEST_SCRIPT_DIR = run/bin

SMOLDER_SERVER = 72.232.235.42:8000
SMOLDER_REPORT_FILENAME = kohatests.txt
SMOLDER_REPORT_TARBALL = kohatests.tar.gz
SMOLDER_PROJECT_NAME = Koha
SMOLDER_SMOKE_SIGNAL = ./smolder_smoke_signal

all ::
	$(NOECHO) $(ECHO) RUNNING THIS MAKEFILE MAY CAUSE LOSS OF DATA
	$(NOECHO) $(ECHO) 
	$(NOECHO) $(ECHO) This makefile is completely beta.
	$(NOECHO) $(ECHO) Please read it first and edit the variables at the top.
	$(NOECHO) $(ECHO) Then, you can run \'make test\'

config_file :: $(CONF_FILE_TEMPLATE) test_run_dirs
	$(CP) $(CONF_FILE_TEMPLATE)  $(TEST_CONF_FILE)
	$(PERL) $(TEST_REWRITE_SCRIPT) --file $(TEST_CONF_FILE)
	$(PERL) $(REAL_REWRITE_SCRIPT)  $(TEST_CONF_FILE)

zebra_conf_files :: test_run_dirs $(ZEBRA_CONF_FILES)


$(ZEBRA_CONF_FILES) ::
	$(PERL) $(TEST_REWRITE_SCRIPT) --file $@
	$(PERL) $(REAL_REWRITE_SCRIPT)  $@

$(SCRIPTS) ::
	$(SED) s/--user=\$$USER.\$$GROUP// $(SRC_SCRIPT_DIR)/$@ > $(TEST_SCRIPT_DIR)/$@
	$(PERL) $(TEST_REWRITE_SCRIPT) --file $(TEST_SCRIPT_DIR)/$@
	$(PERL) $(REAL_REWRITE_SCRIPT)  $(TEST_SCRIPT_DIR)/$@
	$(CHMOD) 755 $(TEST_SCRIPT_DIR)/$@

test :: config_file $(ZEBRA_CONF_FILES) $(SCRIPTS)
	KOHA_CONF=$(TEST_CONF_FILE) PERL5LIB=$(PERL5LIB) TEST_CLASS=$(TEST_CLASS) RUN_EXPENSIVE_TESTS=$(RUN_EXPENSIVE_TESTS) \
		$(PROVE) $(PROVE_FLAGS) $(TEST_FILES) 

test-single :: config_file $(ZEBRA_CONF_FILES) $(SCRIPTS)
	KOHA_CONF=$(TEST_CONF_FILE) PERL5LIB=$(PERL5LIB) RUN_EXPENSIVE_TESTS=1 SINGLE_TEST=1 \
		$(PROVE) $(PROVE_FLAGS) -Ilib $(TEST_FILES)

fulltest :: RUN_EXPENSIVE_TESTS = 1
fulltest :: test

test_run_dirs ::
	$(MKPATH) run/etc
	$(CP) -a ../etc/zebradb run/etc
	$(MKPATH) run/etc/zebradb/etc
	$(MKPATH) run/var/lib/zebradb/biblios/key
	$(MKPATH) run/var/lib/zebradb/biblios/register
	$(MKPATH) run/var/lib/zebradb/biblios/shadow
	$(MKPATH) run/var/lib/zebradb/biblios/tmp
	$(MKPATH) run/var/lib/zebradb/authorities/key
	$(MKPATH) run/var/lib/zebradb/authorities/register
	$(MKPATH) run/var/lib/zebradb/authorities/shadow
	$(MKPATH) run/var/lib/zebradb/authorities/tmp
	$(MKPATH) run/var/lock/zebradb/biblios
	$(MKPATH) run/var/lock/zebradb/authorities
	$(MKPATH) run/var/run/zebradb
	$(MKPATH) run/var/log/zebradb
	$(MKPATH) run/bin

submit_report :: $(SMOLDER_REPORT_TARBALL)
	$(SMOLDER_SMOKE_SIGNAL) --server $(SMOLDER_SERVER) --file $(SMOLDER_REPORT_TARBALL) --project $(SMOLDER_PROJECT_NAME) --anonymous
#	$(RM) $(SMOLDER_REPORT_TARBALL)

submit :: submit_report

$(SMOLDER_REPORT_TARBALL) :: $(SMOLDER_REPORT_FILENAME)
	tar zcvf $(SMOLDER_REPORT_TARBALL) kohatests.txt
	$(RM) $(SMOLDER_REPORT_FILENAME)

$(SMOLDER_REPORT_FILENAME) :: report

report :: config_file $(ZEBRA_CONF_FILES) $(SCRIPTS)
	-KOHA_CONF=$(TEST_CONF_FILE) PERL5LIB=$(PERL5LIB) TEST_CLASS=$(TEST_CLASS) RUN_EXPENSIVE_TESTS=$(RUN_EXPENSIVE_TESTS) \
		$(PERL) $(TEST_FILES) > $(SMOLDER_REPORT_FILENAME)

