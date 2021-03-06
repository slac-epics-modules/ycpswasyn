# How to configure the driver

## Overview

YCPSWASYN uses CPSW for the communication with FPGA and AsynPortDriver for its integration into EPICS.

This document describes how to configure the driver and load it into your IOC.

## YCPSWASYN Configuration

In order to use YCPSWASYN in your application you must call `YCPSWASYNConfig` on your IOC's st.cmd script.

With the following parameters

YCPSWASYNConfig(PORT_NAME, ROOT_PATH, PREFIX, AUTO_GEN_MODE, DICT_FILE, NAMED_ROOT)

| Parameter                  | Description
|----------------------------|-------------------------------------------
| PORT_NAME                  | The name given to this port driver.
| ROOT_PATH                  | Root path to start the generation. If empty, the root will be used.
| PREFIX                     | Record name prefix.
| AUTO_GEN_MODE              | 0: Disable auto-generation. 1: Enabled auto-generation using mapped names. 2: Enable auto-generation using hashed names.
| DICT_FILE                  | Dictionary file path with registers to load. An empty string will disable this function.
| NAMED_ROOT                 | The name of the root to be used, as passed to `cpswLoadYamlFile()`.

**Notes:**
- `PORT_NAME` is mandatory.
- More than one instance of `YCPSWASYNConfig` can be called in the same IOC.
- The auto generation of PVs is enabled setting `AUTO_GEN_MODE` to `1` or `2`.
  - In mode `1`, the PV names are generated using mapping files.
  - In mode `2`, the PV names are generated from SHA1 hash.
  - `PREFIX` can be used in these modes.
  - See [README.autoPVGeneration.md](README.autoPVGeneration.md) for more detailed information about this modes.
- The auto generation of PVs is disabled setting `AUTO_GEN` to `0`.
- To manually creates PVs for register you just need to set `DICT_FILE` to a dictionary file. Otherwise you can set it to an empty string.
- `NAMED_ROOT` is optional, and it can be left empty. It is only necessary to support multiple ATCA blades on a single IOC.

## Optional configuration parameters

Some parameters used by the driver have default values that can be changed calling functions in your st.cmd. The following is a list these parameters,
their default values, and the function used to change them.

| Parameter                                          | Default value     | Function to set a new value
|----------------------------------------------------|-------------------|-------------------------------------
| Path to map files used in auto-generation mode 1   | yaml/             | YCPSWASYNSetMapFilePath(const char* path)
| PV name maximum length                             | Base default (60) | YCPSWASYNSetPvMaxNameLen(int length)
| SCAN value for register without *pollSecs* in YAML | Passive           | YCPSWASYNSetDefaultScan(double scan)
| Path to debug information  file                    | /tmp/             | YCPSWASYNSetDebugFilePath(const char* path)

You must call these functions in your st.cmd before calling `YCPSWASYNConfig`. The changes will apply to all instances of YCPSWASYN you have in
your application.

**Notes:**
- The PV name maximum length will be used only in one of the auto-generation modes.
- The map files are used only in auto-generation mode 1.
- SCAN fields will be set to one the enum values define in base. The value set with `YCPSWASYNSetDefaultScan` (or defined in YAML) will be ceil to
  the next available value in the enum. `0` will be mapped to `Passive`, and any number greater that `10` will be mapped to `10 second`.

## Use of the yamlLoader Module

This module requires the use of the `yamlLoader` module. You must call `cpswLoadYamlFile()` before `YCPSWASYNConfig()` in your st.cmd.

This module will use the root obtained by calling the yamlLoader's `cpswGetRoot()` function. However, you can still use `ROOT_PATH` to define the start of the generation.

