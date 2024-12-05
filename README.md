# OpenShift Cluster Setup and Benchmarking

## Prerequisites

Before starting, ensure the following:

1. **Configure `environment_variables.sh`:**
   - Set up necessary environment variables in the `environment_variables.sh` file. Make sure to include all required configuration details for the OpenShift installation and operators.

2. **Download OpenShift Installer:**
   - Download the OpenShift installer from the official OpenShift mirror:  
     [OpenShift Installer](https://mirror.openshift.com/pub/openshift-v4/)

3. **Copy `install-config.yaml`:**
   - Copy the `install-config.yaml` file into the `ocp/ocp4` directory. This configuration file contains the necessary parameters for the cluster installation.
   - Fill pull secret in install-config.yaml
   - Fill baseDomain in install-config.yaml

4. **Verify Azure Boost VM v6 Support:**
   - Verify that the OpenShift installer supports Azure Boost VM v6. The required patch can be found here:  
     [PR #922](https://github.com/openshift/installer/pull/922)

---

## Usage

### Commands

- **`make install-config`**  
  Creates the installation configuration required for deploying the OpenShift cluster.

- **`make cluster`**  
  Deploys the OpenShift cluster based on the provided `install-config.yaml` configuration.

- **`make destroy`**  
  Destroys the deployed OpenShift cluster and cleans up the resources.

- **`make operator`**  
  Installs the CNV/LSO/ODF operators.  
  **Note:** A *data disk* is required for this operation.

- **`make test`**  
  Runs the specified workloads to test the cluster. Ensure the following:
  - The test environments are configured correctly.
  - `podman` is installed on the system.  
  The results will be saved in the directory `/tmp/benchmark-runner-run-artifacts`.

- **`make version`**  
  Displays the version of the OpenShift installer currently in use.

---
 62 changes: 62 additions & 0 deletions62  
ocp/Makefile
Viewed
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,62 @@
# Makefile for OpenShift installation and cluster management

# Dynamically set MAIN_PATH to the current directory
export MAIN_PATH = $(shell pwd)

# Directory containing the OpenShift configuration
export INSTALL_DIR = $(MAIN_PATH)/ocp4
export INSTALLER_PATH = $(MAIN_PATH)/openshift-install

# OpenShift release image override
export RELEASE_IMAGE_OVERRIDE = quay.io/openshift-release-dev/ocp-release:4.18.0-ec.4-x86_64

# Run Workload
export TEST_PATH = $(MAIN_PATH)/test/./run_test.sh

# Targets
.PHONY: install-config cluster destroy help operator version test

install-config:
	$(INSTALLER_PATH) create install-config --dir="$(INSTALL_DIR)"

cluster:
	$(INSTALLER_PATH) create cluster --dir="$(INSTALL_DIR)"
	$(MAIN_PATH)/.kube/login.sh

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
	bash -c "source $(MAIN_PATH)/environment_variables.sh && $(MAIN_PATH)/operator/run_operator.sh"

test:
	@if [ ! -f "$(MAIN_PATH)/environment_variables.sh" ]; then \
		echo "Environment script $(MAIN_PATH)/environment_variables.sh not found! Please ensure it exists."; \
		exit 1; \
	fi
	@read -p "Did you configure test environments in $(MAIN_PATH)/environment_variables.sh? [y/N] " confirm && if [ "$$confirm" != "y" ]; then \
		echo "Please configure environments in $(MAIN_PATH)/environment_variables.sh before running tests!"; \
		exit 1; \
	fi
	bash -c "source $(MAIN_PATH)/environment_variables.sh && $(TEST_PATH)"

version:
	$(INSTALLER_PATH) version

help:
	@echo "Usage:"
	@echo "  make install-config  - Create the install configuration"
	@echo "  make cluster         - Create the cluster"
	@echo "  make destroy         - Destroy the cluster"
	@echo "  make operator        - Install CNV/LSO/ODF operators [DATA DISK Required!!!]"
	@echo "  make test            - Run Workloads (ensure test environments are configured and podman is installed):Results path: /tmp/benchmark-runner-run-artifacts"
	@echo "  make version         - Display the OpenShift installer version"