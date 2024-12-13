# Makefile for Cloud OpenShift installation

# Set MAIN_PATH to one level up from the current directory
export HEAD_PATH=$(dirname $(pwd))

# Dynamically set MAIN_PATH to the current directory
export MAIN_PATH = $(shell pwd)

# Directory containing the OpenShift configuration
export INSTALLER_PATH = $(MAIN_PATH)/openshift-install
export INSTALL_DIR = $(MAIN_PATH)/ocp4


# Targets
.PHONY: install-config cluster destroy help operator version test

install-config:
	$(INSTALLER_PATH) create install-config --dir="$(INSTALL_DIR)"

cluster:
	@read -p "Did you copy or configure 'install-config.yaml' file in 'ocp4' folder? [y/N] " confirm && if [ "$$confirm" != "y" ]; then \
                echo "Please configure 'install-config.yaml' before proceeding."; \
                exit 1; \
        fi
	OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE=quay.io/openshift-release-dev/ocp-release:4.18.0-ec.4-x86_64 $(INSTALLER_PATH) create cluster --dir="$(INSTALL_DIR)"
	bash -c "source $(MAIN_PATH)/environment_variables.sh && $(MAIN_PATH)/login.sh"

destroy:
	$(INSTALLER_PATH) destroy cluster --dir="$(INSTALL_DIR)"

operator:
	@if [ ! -f "$(MAIN_PATH)/environment_variables.sh" ]; then \
		echo "Environment script $(MAIN_PATH)/environment_variables.sh not found! Please ensure it exists."; \
		exit 1; \
	fi
	@read -p "Did you add the required ODF disks (3 disks per worker)? [y/N] " confirm && if [ "$$confirm" != "y" ]; then \
		echo "Please add the disks before proceeding."; \
		exit 1; \
	fi
	bash -c "source $(MAIN_PATH)/environment_variables.sh && $(HEAD_PATH)/operator/run_operator.sh"

test:
	@if [ ! -f "$(MAIN_PATH)/environment_variables.sh" ]; then \
		echo "Environment script $(MAIN_PATH)/environment_variables.sh not found! Please ensure it exists."; \
		exit 1; \
	fi
	@read -p "Did you configure test environments in $(MAIN_PATH)/environment_variables.sh? [y/N] " confirm && if [ "$$confirm" != "y" ]; then \
		echo "Please configure environments in $(MAIN_PATH)/environment_variables.sh before running tests!"; \
		exit 1; \
	fi
	bash -c "source $(MAIN_PATH)/environment_variables.sh && $(HEAD_PATH)/test/run_test.sh"

version:
	$(INSTALLER_PATH) version

help:
	@echo "Usage:"
	@echo "  make install-config  - Create the install configuration"
	@echo "  make cluster         - Create the cluster: Comment out 'export OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE' when using the official version"
	@echo "  make destroy         - Destroy the cluster"
	@echo "  make operator        - Install CNV/LSO/ODF operators [DATA DISK Required - 3 disks per worker!!!]"
	@echo "  make test            - Run Workloads (ensure test environments are configured and podman is installed):Results path: /tmp/benchmark-runner-run-artifacts"
	@echo "  make version         - Display the OpenShift installer version"