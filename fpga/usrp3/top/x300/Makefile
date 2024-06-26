#
# Copyright 2012-2016 Ettus Research LLC
#

# NOTE: All comments prefixed with a "##" will be displayed as a part of the "make help" target
##-------------------
##USRP X3X0 FPGA Help
##-------------------
##Usage:
## make <Targets> <Options>
##
##Output:
## $(BUILD_OUTPUT_DIR)/usrp_<product>_fpga_<image_type>.bit:    Configuration bitstream with header
## $(BUILD_OUTPUT_DIR)/usrp_<product>_fpga_<image_type>.bin:    Configuration bitstream without header
## $(BUILD_OUTPUT_DIR)/usrp_<product>_fpga_<image_type>.lvbitx: Configuration bitstream for PCIe (NI-RIO)
## $(BUILD_OUTPUT_DIR)/usrp_<product>_fpga_<image_type>.rpt:    Build report (includes utilization and timing summary)

# Debug Options
# Uncomment the following line to add a debug UART on GPIO 10 & 11
#OPTIONS += DEBUG_UART=1

CREATE_LVBITX=python3 ../../lib/io_port2/create-lvbitx.py

# Base output directory for all builds.
BUILD_BASE_DIR ?= .
# Base directory for the build outputs.
BUILD_OUTPUT_DIR ?= $(BUILD_BASE_DIR)/build

# Initialize a build seed. This can be changed to randomly affect build results.
BUILD_SEED ?= 0

GIGE_DEFS=BUILD_1G=1 SFP0_1GBE=1 SFP1_1GBE=1 $(OPTIONS) BUILD_SEED=$(BUILD_SEED)
HG_DEFS=BUILD_1G=1 BUILD_10G=1 SFP0_1GBE=1 SFP1_10GBE=1 $(OPTIONS) BUILD_SEED=$(BUILD_SEED)
XG_DEFS=BUILD_10G=1 SFP0_10GBE=1 SFP1_10GBE=1 $(OPTIONS) BUILD_SEED=$(BUILD_SEED)
HA_DEFS=BUILD_1G=1 BUILD_AURORA=1 SFP0_1GBE=1 SFP1_AURORA=1 $(OPTIONS) BUILD_SEED=$(BUILD_SEED)
XA_DEFS=BUILD_10G=1 BUILD_AURORA=1 SFP0_10GBE=1 SFP1_AURORA=1 $(OPTIONS) BUILD_SEED=$(BUILD_SEED)

# Set build option (check RTL, run synthesis, or do a full build)
ifndef TARGET
	ifeq ($(CHECK), 1)
		TARGET = rtl
	else ifeq ($(SYNTH), 1)
		TARGET = synth
	else ifeq ($(IP_ONLY), 1)
		TARGET = viv_ip
	else
		TARGET = bin
	endif
endif
TOP ?= x300

DEFAULT_IMAGE_CORE_FILE_X300=x300_rfnoc_image_core.v
DEFAULT_IMAGE_CORE_FILE_X310=x310_rfnoc_image_core.v
DEFAULT_EDGE_FILE_X300=$(abspath x300_static_router.hex)
DEFAULT_EDGE_FILE_X310=$(abspath x310_static_router.hex)

# vivado_build($1=Device, $2=Definitions)
vivado_build = make -f Makefile.x300.inc $(TARGET) NAME=$@ ARCH=$(XIL_ARCH_$1) PART_ID=$(XIL_PART_ID_$1) $2 TOP_MODULE=$(TOP) EXTRA_DEFS="$2" DEFAULT_RFNOC_IMAGE_CORE_FILE=$(DEFAULT_IMAGE_CORE_FILE_$1) DEFAULT_EDGE_FILE=$(DEFAULT_EDGE_FILE_$1)
vivado_ip    = make -f Makefile.x300.inc viv_ip    NAME=$@ ARCH=$(XIL_ARCH_$1) PART_ID=$(XIL_PART_ID_$1) $2 TOP_MODULE=$(TOP) EXTRA_DEFS="$2" DEFAULT_RFNOC_IMAGE_CORE_FILE=$(DEFAULT_IMAGE_CORE_FILE_$1) DEFAULT_EDGE_FILE=$(DEFAULT_EDGE_FILE_$1)

# post_build($1=Device, $2=Option)
ifeq ($(TARGET),bin)
	post_build = @\
		mkdir -p $(BUILD_OUTPUT_DIR); \
		echo "Exporting bitstream files..."; \
		cp $(BUILD_BASE_DIR)/build-$(1)_$(2)/x300.bin $(BUILD_OUTPUT_DIR)/usrp_`echo $(1) | tr A-Z a-z`_fpga_$(2).bin; \
		cp $(BUILD_BASE_DIR)/build-$(1)_$(2)/x300.bit $(BUILD_OUTPUT_DIR)/usrp_`echo $(1) | tr A-Z a-z`_fpga_$(2).bit; \
		echo "Generating LVBITX..."; \
		$(CREATE_LVBITX) --input-bin=$(BUILD_BASE_DIR)/build-$(1)_$(2)/x300.bin --output-lvbitx=$(BUILD_OUTPUT_DIR)/usrp_`echo $(1) | tr A-Z a-z`_fpga_$(2).lvbitx --device="USRP $(1)" x3x0_base.lvbitx; \
		cp -f x3x0_base.lvbitx $(BUILD_OUTPUT_DIR)/`echo $(1) | tr A-Z a-z`.lvbitx_base; \
		echo "Exporting build report..."; \
		cp $(BUILD_BASE_DIR)/build-$(1)_$(2)/build.rpt $(BUILD_OUTPUT_DIR)/usrp_`echo $(1) | tr A-Z a-z`_fpga_$(2).rpt; \
		echo "Build DONE ... $(1)_$(2)";
else
	post_build = @echo "Skipping bitfile export."
endif

##
##Supported Targets
##-----------------
##all:      X300_HG X310_HG X300_XG X310_XG (Default target)
all:      X300_HG X310_HG X300_XG X310_XG

##X300_IP:  Build IP for X300 only.
X300_IP:
	+$(call vivado_ip,X300,$(HG_DEFS) X300=1)

##X310_IP:  Build IP for X310 only.
X310_IP:
	+$(call vivado_ip,X310,$(HG_DEFS) X310=1)

##X310_1G:  1GigE on both SFP+ ports.
X310_1G: X310_IP
	$(call vivado_build,X310,$(GIGE_DEFS) X310=1)
	$(call post_build,X310,1G)

##X300_1G:  1GigE on both SFP+ ports.
X300_1G: X300_IP
	$(call vivado_build,X300,$(GIGE_DEFS) X300=1)
	$(call post_build,X300,1G)

##X310_HG:  1GigE on SFP+ Port0, 10Gig on SFP+ Port1.
X310_HG: X310_IP
	$(call vivado_build,X310,$(HG_DEFS) X310=1)
	$(call post_build,X310,HG)

##X300_HG:  1GigE on SFP+ Port0, 10Gig on SFP+ Port1.
X300_HG: X300_IP
	$(call vivado_build,X300,$(HG_DEFS) X300=1)
	$(call post_build,X300,HG)

##X310_XG:  10GigE on both SFP+ ports.
X310_XG: X310_IP
	$(call vivado_build,X310,$(XG_DEFS) X310=1)
	$(call post_build,X310,XG)

##X300_XG:  10GigE on both SFP+ ports.
X300_XG: X300_IP
	$(call vivado_build,X300,$(XG_DEFS) X300=1)
	$(call post_build,X300,XG)

##X310_HA:  1Gig on SFP+ Port0, Aurora on SFP+ Port1.
X310_HA: X310_IP
	$(call vivado_build,X310,$(HA_DEFS) X310=1)
	$(call post_build,X310,HA)

##X300_HA:  1Gig on SFP+ Port0, Aurora on SFP+ Port1.
X300_HA: X300_IP
	$(call vivado_build,X300,$(HA_DEFS) X300=1)
	$(call post_build,X300,HA)

##X310_XA:  10Gig on SFP+ Port0, Aurora on SFP+ Port1.
X310_XA: X310_IP
	$(call vivado_build,X310,$(XA_DEFS) X310=1)
	$(call post_build,X310,XA)

##X300_XA:  10Gig on SFP+ Port0, Aurora on SFP+ Port1.
X300_XA: X300_IP
	$(call vivado_build,X300,$(XA_DEFS) X300=1)
	$(call post_build,X300,XA)

clean:    ##Clean up all target build outputs.
	@echo "Cleaning targets..."
	@rm -rf $(BUILD_BASE_DIR)/build-X3*_*
	@rm -rf $(BUILD_OUTPUT_DIR)

cleanall: ##Clean up all target and ip build outputs.
	@echo "Cleaning targets and IP..."
	@rm -rf build-ip
	@rm -rf $(BUILD_BASE_DIR)/build-X3*_*
	@rm -rf $(BUILD_OUTPUT_DIR)

help:     ##Show this help message.
	@grep -h "##" Makefile | grep -v "\"##\"" | sed -e 's/\\$$//' | sed -e 's/##//'

##
##Supported Options
##-----------------
##GUI=1          Launch the build in the Vivado GUI.
##PROJECT=1      Save Vivado project file, otherwise it's created in memory.
##CHECK=1        Launch the syntax checker instead of building a bitfile.
##IP_ONLY=1      Launch the build but stop after IP generation.
##SYNTH=1        Launch the build but stop after synthesis.
##BUILD_SEED=<N> Build seed to used to affect build results. (Default is 0)
##TOP=<module>   Specify a top module for syntax checking. (Default is the bitfile top)

.PHONY: all clean cleanall help
