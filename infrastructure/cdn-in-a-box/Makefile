# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

############################################################
# Dockerfile to build Edge-Tier Cache container images for
# Apache Traffic Control
# Based on Rocky Linux 8
############################################################

ifeq ($(RHEL_VERSION),)
	export RHEL_VERSION := 8
else
	RHEL_VERSION := $(shell echo $(RHEL_VERSION) | cut -f1 -d.)
endif

CIAB_DIR_RELATIVE := $(dir $(MAKEFILE_LIST))
CIAB_DIR_ABSOLUTE := $(shell cd $(CIAB_DIR_RELATIVE) && pwd)
TC_DIR := $(CIAB_DIR_RELATIVE)../..

PKG_COMMAND := $(TC_DIR)/pkg
PKG_FLAGS := -v -$(RHEL_VERSION)
BUILD_SUFFIX := _build
BUILD_NUMBER := $(shell git rev-list HEAD 2>/dev/null | wc -l | tr -d '[[:space:]]').$(shell git rev-parse --short=8 HEAD)
TC_VERSION := $(shell cat "$(TC_DIR)/VERSION")
TOMCAT_VERSION := $(shell grep '^\s*TOMCAT_VERSION=' "$(TC_DIR)/traffic_router/build/build_rpm.sh"  | cut -d= -f2)
TOMCAT_RELEASE := $(shell grep '^\s*TOMCAT_RELEASE=' "$(TC_DIR)/traffic_router/build/build_rpm.sh"  | cut -d= -f2)
ATS_VERSION := $(shell $(CIAB_DIR_RELATIVE)bin/ats-version.sh)
ifeq ($(RHEL_VERSION),7)
# The commit hash in `git describe` is only 7 characters in CentOS 7's git 1.8
ATS_VERSION := $(shell echo "$(ATS_VERSION)" | sed -E 's/^(.*-[0-9]+\.[0-9a-f]{7})[0-9a-f]*/\1/')
endif

SPECIAL_SAUCE := $(TC_VERSION)-$(BUILD_NUMBER).el$(RHEL_VERSION).x86_64.rpm
SPECIAL_SEASONING := $(TOMCAT_VERSION).$(TOMCAT_RELEASE)-1.el$(RHEL_VERSION).noarch.rpm
SPECIAL_SYRUP := $(ATS_VERSION).el$(RHEL_VERSION).x86_64.rpm

ATS_SOURCE := $(wildcard $(TC_DIR)/cache-config/testing/docker/trafficserver/**)
TO_SOURCE := $(wildcard $(TC_DIR)/traffic_ops/**)
TO_SOURCE += $(wildcard $(TC_DIR)/traffic_ops_db/**)
ORT_SOURCE:= $(wildcard $(TC_DIR)/cache-config/**)
ORT_SOURCE+= $(wildcard $(TC_DIR)/lib/**.go)
TM_SOURCE := $(wildcard $(TC_DIR)/traffic_monitor/**)
TP_SOURCE := $(wildcard $(TC_DIR)/traffic_portal/**)
TR_SOURCE := $(wildcard $(TC_DIR)/traffic_router/**)
TS_SOURCE := $(wildcard $(TC_DIR)/traffic_stats/**)

ATS_RPM := cache/trafficserver.rpm
TM_RPM := traffic_monitor/traffic_monitor.rpm
TO_RPM := traffic_ops/traffic_ops.rpm
TP_RPM := traffic_portal/traffic_portal.rpm
TR_RPM := traffic_router/traffic_router.rpm
TOMCAT_RPM := traffic_router/tomcat.rpm
TS_RPM := traffic_stats/traffic_stats.rpm
ORT_RPM := cache/trafficcontrol-cache-config.rpm

ATS_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(ATS_RPM)
TM_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TM_RPM)
TO_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TO_RPM)
TP_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TP_RPM)
TR_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TR_RPM)
TOMCAT_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TOMCAT_RPM)
TS_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(TS_RPM)
ORT_RPM_RELATIVE := $(CIAB_DIR_RELATIVE)$(ORT_RPM)

ATS_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(ATS_RPM)
TM_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TM_RPM)
TO_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TO_RPM)
TP_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TP_RPM)
TR_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TR_RPM)
TOMCAT_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TOMCAT_RPM)
TS_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(TS_RPM)
ORT_RPM_ABSOLUTE := $(CIAB_DIR_ABSOLUTE)/$(ORT_RPM)

ATS_DIST_RPM := $(TC_DIR)/dist/trafficserver-$(SPECIAL_SYRUP)
TM_DIST_RPM := $(TC_DIR)/dist/traffic_monitor-$(SPECIAL_SAUCE)
TO_DIST_RPM := $(TC_DIR)/dist/traffic_ops-$(SPECIAL_SAUCE)
TP_DIST_RPM := $(TC_DIR)/dist/traffic_portal-$(SPECIAL_SAUCE)
TR_DIST_RPM := $(TC_DIR)/dist/traffic_router-$(SPECIAL_SAUCE)
TOMCAT_DIST_RPM := $(TC_DIR)/dist/tomcat-$(SPECIAL_SEASONING)
TS_DIST_RPM := $(TC_DIR)/dist/traffic_stats-$(SPECIAL_SAUCE)
ORT_DIST_RPM := $(TC_DIR)/dist/trafficcontrol-cache-config-$(SPECIAL_SAUCE)

.PHONY: all build-builders clean debug native nearly-all pull-builders very-clean

# Default target; builds all pre-requisite rpms from source trees
all: $(ATS_RPM_RELATIVE) $(ORT_RPM_RELATIVE) $(TM_RPM_RELATIVE) $(TP_RPM_RELATIVE) $(TO_RPM_RELATIVE) $(TR_RPM_RELATIVE) $(TOMCAT_RPM_RELATIVE) $(TS_RPM_RELATIVE)

ifneq ($(filter build-builders,$(MAKECMDGOALS)),)
PKG_FLAGS += -b
ifneq ($(MAKECMDGOALS),build-builders)
MAKECMDGOALS := $(filter-out build-builders,$(MAKECMDGOALS))
build-builders: $(MAKECMDGOALS)
else
build-builders: all
endif
endif

ifneq ($(filter debug,$(MAKECMDGOALS)),)
PKG_FLAGS += -d
export DEBUG_BUILD = true
ifneq ($(MAKECMDGOALS),debug)
MAKECMDGOALS := $(filter-out debug,$(MAKECMDGOALS))
debug: $(MAKECMDGOALS)
else
debug: all
endif
endif

ifneq ($(filter native,$(MAKECMDGOALS)),)
PKG_COMMAND := $(TC_DIR)/build/clean_build.sh
PKG_FLAGS :=
BUILD_SUFFIX :=
ifneq ($(MAKECMDGOALS),native)
MAKECMDGOALS := $(filter-out native,$(MAKECMDGOALS))
native: $(MAKECMDGOALS)
else
native: all
endif
endif

ifneq ($(filter pull-builders,$(MAKECMDGOALS)),)
PKG_FLAGS += -p
ifneq ($(MAKECMDGOALS),pull-builders)
MAKECMDGOALS := $(filter-out pull-builders,$(MAKECMDGOALS))
pull-builders: $(MAKECMDGOALS)
else
pull-builders: all
endif
endif

# Relative path output RPM recipies
$(ATS_RPM_RELATIVE): $(ATS_DIST_RPM)
	cp -f "$?" "$@" || (rm "$(CIAB_DIR_RELATIVE)/cache/ATS_VERSION"; false)
$(TM_RPM_RELATIVE): $(TM_DIST_RPM)
	cp -f "$?" "$@"
$(TO_RPM_RELATIVE): $(TO_DIST_RPM)
	cp -f "$?" "$@"
$(TP_RPM_RELATIVE): $(TP_DIST_RPM)
	cp -f "$?" "$@"
$(TR_RPM_RELATIVE): $(TR_DIST_RPM)
	cp -f "$?" "$@"
$(TOMCAT_RPM_RELATIVE): $(TOMCAT_DIST_RPM)
	cp -f "$?" "$@"
$(TS_RPM_RELATIVE): $(TS_DIST_RPM)
	cp -f "$?" "$@"
$(ORT_RPM_RELATIVE): $(ORT_DIST_RPM)
	cp -f "$?" "$@"

# Absolute path output RPM recipies
$(ATS_RPM_ABSOLUTE): $(ATS_DIST_RPM)
	cp -f "$?" "$@" || (rm "$(CIAB_DIR_ABSOLUTE)/cache/ATS_VERSION"; false)
$(TM_RPM_ABSOLUTE): $(TM_DIST_RPM)
	cp -f "$?" "$@"
$(TO_RPM_ABSOLUTE): $(TO_DIST_RPM)
	cp -f "$?" "$@"
$(TP_RPM_ABSOLUTE): $(TP_DIST_RPM)
	cp -f "$?" "$@"
$(TR_RPM_ABSOLUTE): $(TR_DIST_RPM)
	cp -f "$?" "$@"
$(TOMCAT_RPM_ABSOLUTE): $(TOMCAT_DIST_RPM)
	cp -f "$?" "$@"
$(TS_RPM_ABSOLUTE): $(TS_DIST_RPM)
	cp -f "$?" "$@"
$(ORT_RPM_ABSOLUTE): $(ORT_DIST_RPM)
	cp -f "$?" "$@"

# Dist rpms
$(ATS_DIST_RPM): $(ATS_SOURCE)
	docker-compose -f $(TC_DIR)/cache-config/testing/docker/docker-compose-ats-build.yml build --parallel trafficserver_build && docker-compose -f $(TC_DIR)/cache-config/testing/docker/docker-compose-ats-build.yml run --rm trafficserver_build

$(TM_DIST_RPM): $(TM_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) traffic_monitor$(BUILD_SUFFIX)

$(TO_DIST_RPM): $(TO_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) traffic_ops$(BUILD_SUFFIX)

$(TP_DIST_RPM): $(TP_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) traffic_portal$(BUILD_SUFFIX)

$(TR_DIST_RPM) $(TOMCAT_DIST_RPM): $(TR_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) traffic_router$(BUILD_SUFFIX)

$(TS_DIST_RPM): $(TS_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) traffic_stats$(BUILD_SUFFIX)

$(ORT_DIST_RPM): $(ORT_SOURCE)
	"$(PKG_COMMAND)" $(PKG_FLAGS) cache-config$(BUILD_SUFFIX)

clean:
	cd "$(CIAB_DIR_RELATIVE)"
	$(RM) $(TM_RPM_RELATIVE) $(TO_RPM_RELATIVE) $(TP_RPM_RELATIVE) $(TR_RPM_RELATIVE) $(TOMCAT_RPM_RELATIVE) $(ORT_RPM_RELATIVE) $(TS_RPM_RELATIVE)

very-clean: clean
	$(warning This will destroy ALL OUTPUT RPMS IN 'dist'. Please be sure this is what you want)
	sleep 2 # Give users a second to cancel
	$(RM) -r "$(TC_DIR)/dist"/*
