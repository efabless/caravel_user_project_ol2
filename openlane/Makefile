# SPDX-FileCopyrightText: 2020 Efabless Corporation
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

MAKEFLAGS+=--warn-undefined-variables

# set shell bash to the do while loop below
SHELL := /usr/bin/env bash
export OPENLANE2_RUN_TAG = $(shell date '+%y_%m_%d_%H_%M')
export CARAVEL_ROOT:=$(CARAVEL_ROOT)
export PDK:=$(PDK)
export PDK_ROOT:=$(PDK_ROOT)
OPENLANE2_TAG ?= 2.0.0b17
OPENLANE2_IMAGE_NAME ?= efabless/openlane:$(OPENLANE2_TAG)
designs = $(shell cd $(PROJECT_ROOT)/openlane && find * -maxdepth 0 -type d)
current_design = null
OPENLANE2_USE_NIX ?= 0
ROOTLESS ?= 0
USER_ARGS = -u $$(id -u $$USER):$$(id -g $$USER)
ifeq ($(ROOTLESS), 1)
	USER_ARGS =
endif

openlane_args = \
	--run-tag $(OPENLANE2_RUN_TAG) \
	--manual-pdk \
	--pdk-root $(PDK_ROOT) \
	--pdk $(PDK) \
	--log-level WARNING \
	$(PROJECT_ROOT)/openlane/$*/config.json

docker_mounts = \
	-m $(PROJECT_ROOT)\
	-m $(PDK_ROOT) \
	-m $(CARAVEL_ROOT)

docker_env = \
	-e PDK_ROOT=$(PDK_ROOT) \
	-e PDK=$(PDK) \
	-e CARAVEL_ROOT=$(CARAVEL_ROOT) \
	-e OPENLANE2_RUN_TAG=$(OPENLANE2_RUN_TAG)

ifneq ($(MCW_ROOT),)
docker_env += -e MCW_ROOT=$(MCW_ROOT)
export MCW_ROOT:=$(MCW_ROOT)
docker_mounts += -m $(MCW_ROOT)
endif

docker_startup_mode = $(shell test -t 0 && echo "-it" || echo "--rm" )
openlane_extra_args = $(shell test -t 0 || echo "--docker-no-tty")
docker_run = \
	docker run $(docker_startup_mode) \
	$(USER_ARGS) \
	$(docker_mounts) \
	$(docker_env) \

list:
	@echo $(designs)

.PHONY: $(designs)
$(designs) : export current_design=$@
$(designs) : % : $(PROJECT_ROOT)/openlane/%/config.json
	# $(current_design)
	@rm -rf $(PROJECT_ROOT)/openlane/$*/runs/$(OPENLANE2_RUN_TAG)
ifeq ($(OPENLANE2_USE_NIX),1)
	nix-shell --pure --command "openlane $(openlane_args) $(openlane_extra_args) --ef-save-views-to $(PROJECT_ROOT)"
else
	$(PROJECT_ROOT)/openlane2-venv/bin/python3 -m openlane $(docker_mounts) $(openlane_extra_args) --dockerized $(openlane_args) $(openlane_extra_args) --ef-save-views-to $(PROJECT_ROOT)
endif
