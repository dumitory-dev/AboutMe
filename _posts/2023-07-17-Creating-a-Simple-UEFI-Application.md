---
layout: post
title: "Creating a Simple UEFI Application with EDK2 and Visual Studio 2022: A Beginner's Guide"
author: "dmitry-ch"
tags: Tale
---

## Introduction

Creating a UEFI (Unified Extensible Firmware Interface) application may seem daunting at first, but with the right tools and guidance, you can get your first app up and running in no time. In this tutorial, I will guide you on how to create a simple UEFI application that prints "Hello, World!" using EDK2 and Visual Studio 2022. I will also guide you on how to run your application on real hardware.

What is UEFI?

UEFI is a specification that defines a software interface between an operating system and platform firmware. UEFI is meant to replace the Basic Input/Output System (BIOS) firmware interface. UEFI is a more modern and flexible interface than BIOS. UEFI is also more secure than BIOS. UEFI is also more modular than BIOS, which allows for more flexibility and customization. - [Wikipedia](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)


What is EDK2?

EDK2 is an open-source development environment for UEFI applications. EDK2 is a fork of the TianoCore project. EDK2 is maintained by the UEFI Forum. - [Wikipedia](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)
More information about EDK2 can be found on the [EDK2 GitHub page](https://github.com/tianocore/EDK2).


## Prerequisites
- A UEFI-enabled hardware for testing. With disabled [secure boot](https://learn.microsoft.com/en-us/mem/intune/user-help/you-need-to-enable-secure-boot-windows)
- [Windows 11](https://www.microsoft.com/de-de/software-download/windows10) development environment.
- The [EDK2](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II) environment set up. 
- [Visual Studio 2022](https://visualstudio.microsoft.com/vs/preview/vs2022/)
- Basic klnowledge of C/C++ programming language.

## Step 1: Setup Your Environment

### Install Visual Studio 2022

Download and install [Visual Studio 2022](https://visualstudio.microsoft.com/vs/preview/vs2022/). Make sure to select the following components during the installation process:

- Desktop development with C++
- Latest Windows SDK

### Setup EDK2 environment

Follow the instructions on the [EDK2 GitHub page](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II)

1. Install [Git](https://git-scm.com/downloads)
2. Install [Python 3.8](https://www.python.org/downloads/release/python-380/)
3. Install NASM (Netwide Assembler) from [here](https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/win64/)
4. You need set set the NASM_PREFIX environment variable to the path where you installed NASM. For example, if you installed NASM to C:\NASM, you would set the NASM_PREFIX environment variable to C:\NASM. 

### Clone the EDK2 Repository

Clone the EDK2 repository to your local machine. You can do this by running the following command in the command prompt:

```bash
git clone https://github.com/tianocore/EDK2.git
git submodule update --init --recursive
```

### Build or Install the EDK2 [BaseTools]()
Open the **Visual Studio Developer Command Prompt** and navigate to the EDK2 directory. Then run the following command:

```bash
edksetup.bat Rebuild
``` 

This command will initiate the compilation and building process of the EDK2 base tools. The resulting binaries will be located in `<root>\BaseTools\Bin\Win32`. These tools include GenFfs.exe, GenFv.exe, VfrCompile.exe, and various other commonly used tools.

In addition to building the tools, the command will also copy the default configuration templates from `<root>\BaseTools\Conf` to `<root>\Conf`. Specifically, the files build_rule.txt, target.txt, and tools_def.txt will be copied.


**Another way** to install the EDK2 BaseTools is to download the pre-built binaries from [here](https://github.com/dumitory-dev/EDK2-BaseTools-win32-updated/releases) (will be updated soon). Then extract the contents of the archive to `<root>\BaseTools\Bin\Win32`. Don't forget to copy the default configuration templates from `<root>\BaseTools\Conf` to `<root>\Conf`.

## Step 2: Create Your First UEFI Application

Once your environment is set up, it's time to create your first UEFI application.

### Create a new UEFI Application Project

First, we need to create DSC (Decsription) and INF (Information) files. These files are used to describe the contents of the UEFI application. 

> Each platform DSC file is broken out into sections in a manner similar to the component description (INF) files. However, while the intent of a component's INF file is to define the source files, libraries (or library classes), and definitions relevant to building the component, the function of the platform DSC file is to specify the library instances, components and output formats used to generate binary files that will be processed by other tools to generate an image that is either put into a flash device, made available for recovery booting or updating existing firmware on a platform.
[EDK2 Wiki](https://tianocore-docs.github.io/EDK2-DscSpecification/release-1.28/2_dsc_overview/21_processing_overview.html#21-processing-overview)

In the EDK2 folder structure, create a new folder called `HelloWorldPkg` in `<root>\EDK2\HelloWorldPkg`. Then create a new file called `HelloWorldPkg.dsc`. Then copy the following contents into the `HelloWorldPkg.dsc` file:

```ini

[Defines]
  DSC_SPECIFICATION         = 0x00010005 # It is the version of the DSC specification that this file conforms to.
  PLATFORM_GUID             = c4d69391-7c20-426e-b1d4-33cbabc7116c # Create GUID - https://www.guidgenerator.com/online-guid-generator.aspx
  PLATFORM_VERSION          = 0.01 # The version of the platform.
  PLATFORM_NAME             = HelloWorldPkg # The name of the platform.
  SKUID_IDENTIFIER          = DEFAULT # The contents of this section are used to define valid SKUID_IDENTIFIER names.
  SUPPORTED_ARCHITECTURES   = AARCH64|X64 # all supported architectures for this platform
  BUILD_TARGETS             = DEBUG|RELEASE|NOOPT 
  OUTPUT_DIRECTORY          = $(PKG_OUTPUT_DIR)

# Varios libs that are required to build the our UEFI application
[LibraryClasses]
  #
  # Basic
  #
  BaseLib|MdePkg/Library/BaseLib/BaseLib.inf
  BaseMemoryLib|MdePkg/Library/BaseMemoryLib/BaseMemoryLib.inf
  DevicePathLib|MdePkg/Library/UefiDevicePathLib/UefiDevicePathLib.inf
  MemoryAllocationLib|MdePkg/Library/UefiMemoryAllocationLib/UefiMemoryAllocationLib.inf
  PrintLib|MdePkg/Library/BasePrintLib/BasePrintLib.inf
  UefiLib|MdePkg/Library/UefiLib/UefiLib.inf

  #
  # UEFI & PI
  #
  UefiApplicationEntryPoint|MdePkg/Library/UefiApplicationEntryPoint/UefiApplicationEntryPoint.inf
  UefiBootServicesTableLib|MdePkg/Library/UefiBootServicesTableLib/UefiBootServicesTableLib.inf
  UefiRuntimeServicesTableLib|MdePkg/Library/UefiRuntimeServicesTableLib/UefiRuntimeServicesTableLib.inf

  #
  # Misc
  #
  DebugLib|MdePkg/Library/BaseDebugLibNull/BaseDebugLibNull.inf
  PcdLib|MdePkg/Library/BasePcdLibNull/BasePcdLibNull.inf

[LibraryClasses.ARM,LibraryClasses.AARCH64]
  #
  # It is not possible to prevent the ARM compiler for generic intrinsic functions.
  # This library provides the instrinsic functions generate by a given compiler.
  # [LibraryClasses.ARM] and NULL mean link this library into all ARM images.
  #
  NULL|ArmPkg/Library/CompilerIntrinsicsLib/CompilerIntrinsicsLib.inf

  # Add support for GCC stack protector
  NULL|MdePkg/Library/BaseStackCheckLib/BaseStackCheckLib.inf

[Components]
  HelloWorldApplication.inf

```

Then create a new file called `HelloWorldApplication.inf` in the same folder. Then copy the following contents into the `HelloWorldApplication.inf` file:


```ini
# Variables defined to be used during the build process
[Defines] 
  INF_VERSION       = 0x00010005 
  BASE_NAME         = HelloWorldApplication 
  FILE_GUID         = 4d830c7f-285c-4a17-b9ca-fdeec0d7d208 # Create GUID - https://www.guidgenerator.com/online-guid-generator.aspx
  MODULE_TYPE       = UEFI_APPLICATION
  VERSION_STRING    = 1.0 # The version of the module.
  ENTRY_POINT       = UefiMain 

# Source code
[Sources]
  HelloWorld.c

# Required packages
[Packages]
  MdePkg/MdePkg.dec            # Contains Uefi and UefiLib

# Required Libraries
[LibraryClasses]
  UefiApplicationEntryPoint    # Uefi application entry point
  UefiLib                      # UefiLib
```
