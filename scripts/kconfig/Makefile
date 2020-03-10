# SPDX-License-Identifier: GPL-2.0
# ===========================================================================

# ===========================================================================
# object files used by all kconfig flavours
common-objs	:= confdata.o expr.o lexer.lex.o menu.o parser.tab.o \
		   preprocess.o symbol.o util.o

$(obj)/lexer.lex.o: $(obj)/parser.tab.h
HOSTCFLAGS_lexer.lex.o	:= -I $(srctree)/$(src)
HOSTCFLAGS_parser.tab.o	:= -I $(srctree)/$(src)

# conf: Used for defconfig, oldconfig and related targets
hostprogs	+= conf
conf-objs	:= conf.o $(common-objs)

# nconf: Used for the nconfig target based on ncurses
hostprogs	+= nconf
nconf-objs	:= nconf.o nconf.gui.o $(common-objs)

HOSTLDLIBS_nconf       = $(call read-file, $(obj)/nconf-libs)
HOSTCFLAGS_nconf.o     = $(call read-file, $(obj)/nconf-cflags)
HOSTCFLAGS_nconf.gui.o = $(call read-file, $(obj)/nconf-cflags)

$(obj)/nconf: | $(obj)/nconf-libs
$(obj)/nconf.o $(obj)/nconf.gui.o: | $(obj)/nconf-cflags

# mconf: Used for the menuconfig target based on lxdialog
hostprogs	+= mconf
lxdialog	:= $(addprefix lxdialog/, \
		     checklist.o inputbox.o menubox.o textbox.o util.o yesno.o)
mconf-objs	:= mconf.o $(lxdialog) $(common-objs)

HOSTLDLIBS_mconf = $(call read-file, $(obj)/mconf-libs)
$(foreach f, mconf.o $(lxdialog), \
  $(eval HOSTCFLAGS_$f = $$(call read-file, $(obj)/mconf-cflags)))

$(obj)/mconf: | $(obj)/mconf-libs
$(addprefix $(obj)/, mconf.o $(lxdialog)): | $(obj)/mconf-cflags

# qconf: Used for the xconfig target based on Qt
hostprogs	+= qconf
qconf-cxxobjs	:= qconf.o qconf-moc.o
qconf-objs	:= images.o $(common-objs)

HOSTLDLIBS_qconf         = $(call read-file, $(obj)/qconf-libs)
HOSTCXXFLAGS_qconf.o     = -std=c++11 -fPIC $(call read-file, $(obj)/qconf-cflags)
HOSTCXXFLAGS_qconf-moc.o = -std=c++11 -fPIC $(call read-file, $(obj)/qconf-cflags)
$(obj)/qconf: | $(obj)/qconf-libs
$(obj)/qconf.o $(obj)/qconf-moc.o: | $(obj)/qconf-cflags

quiet_cmd_moc = MOC     $@
      cmd_moc = $(call read-file, $(obj)/qconf-bin)/moc $< -o $@

$(obj)/qconf-moc.cc: $(src)/qconf.h FORCE | $(obj)/qconf-bin
	$(call if_changed,moc)

targets += qconf-moc.cc

# gconf: Used for the gconfig target based on GTK+
hostprogs	+= gconf
gconf-objs	:= gconf.o images.o $(common-objs)

HOSTLDLIBS_gconf   = $(call read-file, $(obj)/gconf-libs)
HOSTCFLAGS_gconf.o = $(call read-file, $(obj)/gconf-cflags)

$(obj)/gconf: | $(obj)/gconf-libs
$(obj)/gconf.o: | $(obj)/gconf-cflags

# check if necessary packages are available, and configure build flags
cmd_conf_cfg = $< $(addprefix $(obj)/$*conf-, cflags libs bin); touch $(obj)/$*conf-bin

$(obj)/%conf-cflags $(obj)/%conf-libs $(obj)/%conf-bin: $(src)/%conf-cfg.sh
	$(call cmd,conf_cfg)

clean-files += *conf-cflags *conf-libs *conf-bin
