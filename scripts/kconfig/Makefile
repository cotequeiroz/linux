# ===========================================================================
# OpenWrt configuration targets
# ===========================================================================
# Platform specific fixes
#
# FreeBSD

all: conf mconf
clean:
	rm -f *.o lxdialog/*.o $(clean-files) conf mconf

# ===========================================================================
# object files used by all kconfig flavours
common-objs	:= confdata.o expr.o lexer.lex.o parser.tab.o preprocess.o \
		   symbol.o util.o
clean-files	:= parser.tab.c parser.tab.h lexer.lex.c

lexer.lex.o: parser.tab.h
parser.tab.c: parser.y
lexer.lex.c: lexer.l
export CFLAGS += -DKBUILD_NO_NLS -I.
export CXXFLAGS += -DKBUILD_NO_NLS

%.tab.c %.tab.h: %.y
	bison -d -l -b $* $<

%.lex.c: %.l
	flex -L -o$@ $<

# conf: Used for defconfig, oldconfig and related targets
conf-objs	:= conf.o $(common-objs)
conf: $(conf-objs)

# mconf: Used for the menuconfig target based on lxdialog
lxdialog	:= $(addprefix lxdialog/, \
		     checklist.o inputbox.o menubox.o textbox.o util.o yesno.o)
mconf-objs	:= mconf.o $(lxdialog) $(common-objs)

check_lxdialog = $(shell $(SHELL) $(CURDIR)/lxdialog/check-lxdialog.sh -$(1))
export CFLAGS += $(call check_lxdialog,ccflags)

mconf: $(mconf-objs)
	$(CC) -o $@ $^ $(call check_lxdialog,ldflags $(CC))

# qconf: Used for the xconfig target based on Qt
qconf-cxxobjs	:= qconf.o
qconf-objs	:= images.o $(common-objs)

HOSTLDLIBS_qconf	= $(KC_QT_LIBS)
HOSTCXXFLAGS_qconf.o	= $(KC_QT_CFLAGS)
# Remove qconf junk files
clean-files	+= $(qconf-cxxobjs) qconf.moc .tmp_qtcheck qconf

qconf: $(qconf-cxxobjs) $(qconf-objs)
ifneq ($(DISTRO-PKG-CONFIG),)
	$(CXX) -o $@ $^ $(HOSTLDLIBS_qconf)
else
	echo "You don't have 'pkg-config' installed. Cannot continue"
	echo "For now, you may use 'make menuconfig' instead of 'make xconfig'"
endif

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

qconf.o: qconf.moc
qconf.o: CXXFLAGS+=$(HOSTCXXFLAGS_qconf.o)

moc = $(KC_QT_MOC) -i $< -o $@

%.moc: %.h .tmp_qtcheck
	$(call moc)
