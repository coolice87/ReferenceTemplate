# 
# Support make menuconfig
#
#


# ============================================================
# Beautify output
# If $(quiet) is empty, the whole command will be printed
# If it is set to "quiet_", only the short version will be printed
# If it is set to "silent_", nothing will be printed at all
#
# Use 'make V=1' to see the full commands
# If the user is running make -s (silent mode)
ifeq ("$(origin V)", "command line")
	KBUILD_VERBOSE = $(V)
endif
ifndef KBUILD_VERBOSE
	KBUILD_VERBOSE = 0
endif

ifeq ($(KBUILD_VERBOSE), 1)
	quiet = 
	Q = 
else
	quiet = quiet_
	Q = @
endif

ifneq ($(filter %s, $(firstword x$(MAKEFLAGS))), )
	quiet = silent_
endif

export quiet Q KBUILD_VERBOSE

# ==========================================================
# work directory
ifeq ($(KBUILD_SRC), )
	srctree = .
else
	srctree = $(KBUILD_SRC)
endif

export srctree

KCONFIG_CONFIG  ?= .config
export KCONFIG_CONFIG

HOSTCC       = cc
HOSTCXX      = c++
HOSTCFLAGS   = -Wall -Wstrict-prototypes -O2 -fomit-frame-pointer \
			   $(if $(CONFIG_TOOLS_DEBUG),-g)
HOSTCXXFLAGS = -O2

export HOSTCC HOSTCFLAGS HOSTLDFLAGS

PHONY := all
all:

# Cancel implicit rules on top Makefile
$(CURDIR)/Makefile Makefile: ;

# We need some generic definitions (do not try to remake the file)
scripts/Kbuild.include: ;
include scripts/Kbuild.include

# ==========================================================
# Rules shared between *config targets and build targets
# Basic helpers built in scripts/
#
PHONY += scripts_basic
scripts_basic:
	$(Q)$(MAKE) $(build)=scripts/basic 
	$(Q)rm -f .tmp_quiet_recordmcount

# ===========================================================
# *config targets only - make sure prerequisites are updated, and descend
# in scripts/kconfig to make the *config target
config-targets := 0
dot-config := 1

ifneq ($(filter %config, $(MAKECMDGOALS)),)
	config-targets := 1
endif

ifeq ($(config-targets), 1)
%config: scripts_basic
	$(Q)$(MAKE) $(build)=scripts/kconfig $@
else
ifeq ($(dot-config),1)
# Read in config
-include include/config/auto.conf

# Read in dependencies to all Kconfig* files, make sure to run
# oldconfig if changes are detected
-include include/config/auto.conf.cmd

# To avoid any implicit rule to kick in, define an empty command
# $(KCONFIG_CONFIG) include/config/auto.conf.cmd: ;
#
# If .config is newer than include/config/auto.conf, someone tinkered
# with it and forgot to run make oldconfig.
# if auto.conf.cmd is missing then we are probably in a cleaned tree so
# we execute the config step to be sure to catch updated Kconfig files
include/config/%.conf: $(KCONFIG_CONFIG) include/config/auto.conf.cmd
	$(Q)$(MAKE) -f $(srctree)/Makefile silentoldconfig
endif # $(dot-config)
endif # $(config-targets)

PHONY += clean
clean:
	@find \( \
		-name '*.[oas]' -o -name '.*.cmd' -o -name '.*.d' \
		\) -type f -print | xargs rm -f

.PHONY: $(PHONY)

