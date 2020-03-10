# SPDX-License-Identifier: GPL-2.0
# ===========================================================================

# ===========================================================================
# object files used by all kconfig flavours
common-objs	:= confdata.o expr.o lexer.lex.o parser.tab.o preprocess.o \
		   symbol.o util.o

$(obj)/lexer.lex.o: $(obj)/parser.tab.h
HOSTCFLAGS_lexer.lex.o	:= -I $(srctree)/$(src)
HOSTCFLAGS_parser.tab.o	:= -I $(srctree)/$(src)

# conf: Used for defconfig, oldconfig and related targets
hostprogs-y	+= conf
conf-objs	:= conf.o $(common-objs)

# nconf: Used for the nconfig target based on ncurses
hostprogs-y	+= nconf
nconf-objs	:= nconf.o nconf.gui.o $(common-objs)

HOSTLDLIBS_nconf	= $(shell . $(obj)/nconf-cfg && echo $$libs)
HOSTCFLAGS_nconf.o	= $(shell . $(obj)/nconf-cfg && echo $$cflags)
HOSTCFLAGS_nconf.gui.o	= $(shell . $(obj)/nconf-cfg && echo $$cflags)

$(obj)/nconf.o $(obj)/nconf.gui.o: $(obj)/nconf-cfg

# mconf: Used for the menuconfig target based on lxdialog
hostprogs-y	+= mconf
lxdialog	:= $(addprefix lxdialog/, \
		     checklist.o inputbox.o menubox.o textbox.o util.o yesno.o)
mconf-objs	:= mconf.o $(lxdialog) $(common-objs)

HOSTLDLIBS_mconf = $(shell . $(obj)/mconf-cfg && echo $$libs)
$(foreach f, mconf.o $(lxdialog), \
  $(eval HOSTCFLAGS_$f = $$(shell . $(obj)/mconf-cfg && echo $$$$cflags)))

$(addprefix $(obj)/, mconf.o $(lxdialog)): $(obj)/mconf-cfg

# qconf: Used for the xconfig target based on Qt
hostprogs-y	+= qconf
qconf-cxxobjs	:= qconf.o
qconf-objs	:= images.o $(common-objs)

HOSTLDLIBS_qconf	= $(shell . $(obj)/qconf-cfg && echo $$libs)
HOSTCXXFLAGS_qconf.o	= $(shell . $(obj)/qconf-cfg && echo $$cflags)

$(obj)/qconf.o: $(obj)/qconf-cfg $(obj)/qconf.moc

quiet_cmd_moc = MOC     $@
      cmd_moc = $(shell . $(obj)/qconf-cfg && echo $$moc) -i $< -o $@

$(obj)/%.moc: $(src)/%.h $(obj)/qconf-cfg
	$(call cmd,moc)

# check if necessary packages are available, and configure build flags
filechk_conf_cfg = $(CONFIG_SHELL) $<

$(obj)/%conf-cfg: $(src)/%conf-cfg.sh FORCE
	$(call filechk,conf_cfg)

clean-files += *conf-cfg
