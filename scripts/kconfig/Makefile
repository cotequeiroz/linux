# ===========================================================================
# OpenWrt configuration targets
# These targets are used from top-level makefile

# ===========================================================================
# object files used by all kconfig flavours

common-objs	:= confdata.o expr.o symbol.o preprocess.o zconf.tab.o

# Platform specific fixes
#
# FreeBSD

check_lxdialog = $(shell $(SHELL) $(CURDIR)/lxdialog/check-lxdialog.sh -$(1))
export CFLAGS += -DKBUILD_NO_NLS -I. $(call check_lxdialog,ccflags)
export CXXFLAGS += -DKBUILD_NO_NLS

conf-objs	:= conf.o $(common-objs)
mconf-objs	:= mconf.o $(common-objs)
qconf-cxxobjs	:= qconf.o
qconf-objs	:= zconf.tab.o

lxdialog-objs := \
	lxdialog/checklist.o lxdialog/util.o lxdialog/inputbox.o \
	lxdialog/textbox.o lxdialog/yesno.o lxdialog/menubox.o

clean-files	:= zconf.tab.c zconf.lex.c
# Remove qconf junk files
clean-files	+= $(qconf-cxxobjs) qconf.moc .tmp_qtcheck qconf

all: conf mconf

conf: $(conf-objs)
mconf: $(mconf-objs) $(lxdialog-objs)
	$(CC) -o $@ $^ $(call check_lxdialog,ldflags $(CC))
qconf: $(qconf-cxxobjs) $(qconf-objs)
ifneq ($(DISTRO-PKG-CONFIG),)
	$(CXX) -o $@ $^ $(HOSTLOADLIBES_qconf)
else
	echo "You don't have 'pkg-config' installed. Cannot continue"
	echo "For now, you may use 'make menuconfig' instead of 'make xconfig'"
endif

clean:
	rm -f *.o lxdialog/*.o $(clean-files) conf mconf

zconf.tab.o: zconf.lex.c confdata.c

kconfig_load.o: lkc_defs.h

zconf.tab.c: zconf.y
zconf.lex.c: zconf.l

%.tab.c: %.y
	bison -l -b $* -p $(notdir $*) $<

%.lex.c: %.l
	flex -L -P$(notdir $*) -o$@ $<

ifeq ($(MAKECMDGOALS),qconf)
qconf.o: .tmp_qtcheck
.tmp_qtcheck: Makefile
-include .tmp_qtcheck

# Qt needs some extra effort...
.tmp_qtcheck:
	@set -e; echo "  CHECK   qt"; \
	if $(DISTRO-PKG-CONFIG) --exists Qt5Core; then \
	    cflags="-std=c++11 -fPIC `$(DISTRO-PKG-CONFIG) --cflags Qt5Core Qt5Gui Qt5Widgets`"; \
	    libs=`$(DISTRO-PKG-CONFIG) --libs Qt5Core Qt5Gui Qt5Widgets`; \
	    moc=`$(DISTRO-PKG-CONFIG) --variable=host_bins Qt5Core`/moc; \
	elif $(DISTRO-PKG-CONFIG) --exists QtCore; then \
	    cflags=`$(DISTRO-PKG-CONFIG) --cflags QtCore QtGui`; \
	    libs=`$(DISTRO-PKG-CONFIG) --libs QtCore QtGui`; \
	    moc=`$(DISTRO-PKG-CONFIG) --variable=moc_location QtCore`; \
	else \
	    echo >&2 "*"; \
	    echo >&2 "* Could not find Qt via $(DISTRO-PKG-CONFIG)."; \
	    echo >&2 "* Please install either Qt 4.8 or 5.x. and make sure it's in PKG_CONFIG_PATH"; \
	    echo >&2 "*"; \
	    exit 1; \
	fi; \
	echo "KC_QT_CFLAGS=$$cflags" > $@; \
	echo "KC_QT_LIBS=$$libs" >> $@; \
	echo "KC_QT_MOC=$$moc" >> $@
endif

#Define compiler flags to build qconf
HOSTLOADLIBES_qconf	= $(KC_QT_LIBS)
HOSTCXXFLAGS_qconf.o	= $(KC_QT_CFLAGS)

qconf.o: qconf.moc
qconf.o: CXXFLAGS+=$(HOSTCXXFLAGS_qconf.o)

moc = $(KC_QT_MOC) -i $< -o $@

%.moc: %.h .tmp_qtcheck
	$(call moc)
