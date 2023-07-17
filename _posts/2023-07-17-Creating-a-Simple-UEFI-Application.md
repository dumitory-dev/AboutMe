---
layout: post
title: "Creating a Simple UEFI Application with EDK2 and Visual Studio 2022: A Beginner's Guide"
author: "dmitry-ch"
category: How to
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
- Flash drive with FAT32 partition.

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
> If you see the following error: `Treat specific warning as error...`, try to apply this patch:

```bash
diff --git a/BaseTools/Source/C/Makefiles/ms.common b/BaseTools/Source/C/Makefiles/ms.common
index fe7a59c280..3543edda15 100644
--- a/BaseTools/Source/C/Makefiles/ms.common
+++ b/BaseTools/Source/C/Makefiles/ms.common
@@ -66,5 +66,5 @@ LINKER = $(LD)
 INC = $(INC) -I . -I $(SOURCE_PATH)\Include -I $(ARCH_INCLUDE) -I $(SOURCE_PATH)\Common
 INC = $(INC) -I $(EDK2_PATH)\MdePkg\Include
 
-CFLAGS = $(CFLAGS) /nologo /Z7 /c /O2 /MT /W4 /WX /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE
+CFLAGS = $(CFLAGS) /nologo /Z7 /c /O2 /MT /W4 /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE
 CPPFLAGS = $(CPPFLAGS) /EHsc /nologo /Z7 /c /O2 /MT /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE
diff --git a/BaseTools/Source/C/VfrCompile/Makefile b/BaseTools/Source/C/VfrCompile/Makefile
index d6ec6ad00b..f2a84d7420 100644
--- a/BaseTools/Source/C/VfrCompile/Makefile
+++ b/BaseTools/Source/C/VfrCompile/Makefile
@@ -6,7 +6,7 @@
 #
 !INCLUDE ..\Makefiles\ms.common
 
-CPPFLAGS = $(CPPFLAGS) /WX /D PCCTS_USE_NAMESPACE_STD
+CPPFLAGS = $(CPPFLAGS) /D PCCTS_USE_NAMESPACE_STD
 APPNAME = VfrCompile
 
 LIBS = $(LIB_PATH)\Common.lib
diff --git a/CryptoPkg/Library/IntrinsicLib/IntrinsicLib.inf b/CryptoPkg/Library/IntrinsicLib/IntrinsicLib.inf
index 86e74b57b1..c16fef41c8 100644
--- a/CryptoPkg/Library/IntrinsicLib/IntrinsicLib.inf
+++ b/CryptoPkg/Library/IntrinsicLib/IntrinsicLib.inf
@@ -54,16 +54,16 @@
    #
    # Override MSFT build option to remove /Oi and /GL
    #
-   MSFT:DEBUG_VS2003_IA32_CC_FLAGS        == /nologo /c /WX /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX- /Zi /Gm
-   MSFT:RELEASE_VS2003_IA32_CC_FLAGS      == /nologo /c /WX /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX-
-   MSFT:DEBUG_VS2003xASL_IA32_CC_FLAGS    == /nologo /c /WX /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX- /Zi /Gm
-   MSFT:RELEASE_VS2003xASL_IA32_CC_FLAGS  == /nologo /c /WX /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX-
-   MSFT:DEBUG_DDK3790_IA32_CC_FLAGS       == /nologo /c /WX /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
-   MSFT:RELEASE_DDK3790_IA32_CC_FLAGS     == /nologo /c /WX /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
-   MSFT:DEBUG_DDK3790xASL_IA32_CC_FLAGS   == /nologo /c /WX /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
-   MSFT:RELEASE_DDK3790xASL_IA32_CC_FLAGS == /nologo /c /WX /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
-   MSFT:DEBUG_*_IA32_CC_FLAGS             == /nologo /c /WX /GS- /W4 /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Gy /Zi /Gm
-   MSFT:RELEASE_*_IA32_CC_FLAGS           == /nologo /c /WX /GS- /W4 /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
-   MSFT:DEBUG_*_X64_CC_FLAGS              == /nologo /c /WX /GS- /X /W4 /Gs32768 /D UNICODE /O1b2s /Gy /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
-   MSFT:RELEASE_*_X64_CC_FLAGS            == /nologo /c /WX /GS- /X /W4 /Gs32768 /D UNICODE /O1b2s /Gy /FIAutoGen.h /EHs-c- /GR- /GF
+   MSFT:DEBUG_VS2003_IA32_CC_FLAGS        == /nologo /c /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX- /Zi /Gm
+   MSFT:RELEASE_VS2003_IA32_CC_FLAGS      == /nologo /c /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX-
+   MSFT:DEBUG_VS2003xASL_IA32_CC_FLAGS    == /nologo /c /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX- /Zi /Gm
+   MSFT:RELEASE_VS2003xASL_IA32_CC_FLAGS  == /nologo /c /W4 /Gs32768 /Gy /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /GX-
+   MSFT:DEBUG_DDK3790_IA32_CC_FLAGS       == /nologo /c /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
+   MSFT:RELEASE_DDK3790_IA32_CC_FLAGS     == /nologo /c /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
+   MSFT:DEBUG_DDK3790xASL_IA32_CC_FLAGS   == /nologo /c /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
+   MSFT:RELEASE_DDK3790xASL_IA32_CC_FLAGS == /nologo /c /Gy /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
+   MSFT:DEBUG_*_IA32_CC_FLAGS             == /nologo /c /GS- /W4 /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF /Gy /Zi /Gm
+   MSFT:RELEASE_*_IA32_CC_FLAGS           == /nologo /c /GS- /W4 /Gs32768 /D UNICODE /O1b2 /FIAutoGen.h /EHs-c- /GR- /GF
+   MSFT:DEBUG_*_X64_CC_FLAGS              == /nologo /c /GS- /X /W4 /Gs32768 /D UNICODE /O1b2s /Gy /FIAutoGen.h /EHs-c- /GR- /GF /Zi /Gm
+   MSFT:RELEASE_*_X64_CC_FLAGS            == /nologo /c /GS- /X /W4 /Gs32768 /D UNICODE /O1b2s /Gy /FIAutoGen.h /EHs-c- /GR- /GF
   INTEL:*_*_*_CC_FLAGS                    =  /Oi-

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
  OUTPUT_DIRECTORY          = Build/HelloWorldPkg

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
  RegisterFilterLib|MdePkg/Library/RegisterFilterLibNull/RegisterFilterLibNull.inf

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
  HelloWorldPkg/HelloWorldApplication.inf
```

Then create a new file called `HelloWorldApplication.inf` in the same folder. Then copy the following contents into the `HelloWorldApplication.inf` file:

```ini
[Defines]
  INF_VERSION                    = 1.25
  BASE_NAME                      = HelloWorldApplication
  FILE_GUID                      = db9614f2-b421-4fa1-a324-e9a6cbfc611d
  MODULE_TYPE                    = UEFI_APPLICATION
  VERSION_STRING                 = 1.0
  ENTRY_POINT                    = UefiMain
  VALID_ARCHITECTURES            = X64


[Sources]
  HelloWorldApplication.c

[Packages]
  MdePkg/MdePkg.dec
  
[LibraryClasses]
  UefiApplicationEntryPoint
  UefiLib
```

Finally, create a new file called `HelloWorldApplication.c` in the same folder. Then copy the following contents into the `HelloWorldApplication.c` file:

```c
#include <Library/UefiBootServicesTableLib.h>
#include <Library/UefiLib.h>

/**
  as the real entry point for the application.

  @param[in] ImageHandle    The firmware allocated handle for the EFI image.
  @param[in] SystemTable    A pointer to the EFI System Table.

  @retval EFI_SUCCESS       The entry point is executed successfully.
  @retval other             Some error occurs when executing this entry point.

**/
EFI_STATUS
EFIAPI
UefiMain(
    IN EFI_HANDLE ImageHandle,
    IN EFI_SYSTEM_TABLE *SystemTable)
{
  UINTN Index = 0;
  EFI_STATUS Status = EFI_SUCCESS;

  if (SystemTable == NULL)
  {
    return EFI_INVALID_PARAMETER;
  }

  if ((SystemTable->ConOut == NULL) || (SystemTable->ConOut->OutputString == NULL) || (SystemTable->ConOut->ClearScreen == NULL))
  {
    return EFI_INVALID_PARAMETER;
  }

  if ((SystemTable->BootServices == NULL) || (SystemTable->BootServices->Stall == NULL))
  {
    return EFI_INVALID_PARAMETER;
  }

  Status = SystemTable->ConOut->ClearScreen(SystemTable->ConOut);
  if (EFI_ERROR(Status))
  {
    return Status;
  }

  Status = SystemTable->ConOut->OutputString(SystemTable->ConOut, L"\r\nHello World!\r\n");
  if (EFI_ERROR(Status))
  {
    return Status;
  }

  // Or we can use just Print function
  Print(L"Press any key to boot...!\n");
  Status = gBS->WaitForEvent(1, &(gST->ConIn->WaitForKey), &Index);

  if (EFI_ERROR(Status))
  {
    return Status;
  }

  gST->ConIn->Reset(gST->ConIn, FALSE);

  return EFI_SUCCESS;
}

```

### Build the UEFI Application

Now that we have created the DSC and INF files, we can build the UEFI application. To do this, open the **Visual Studio Developer Command Prompt** and navigate to the EDK2 directory. Open the `Conf/target.txt` file and setup the following variables:

```ini
#
#  Copyright (c) 2006 - 2019, Intel Corporation. All rights reserved.<BR>
#
#  SPDX-License-Identifier: BSD-2-Clause-Patent
#
#
#  ALL Paths are Relative to WORKSPACE

#  Separate multiple LIST entries with a SINGLE SPACE character, do not use comma characters.
#  Un-set an option by either commenting out the line, or not setting a value.

#
#  PROPERTY              Type       Use         Description
#  ----------------      --------   --------    -----------------------------------------------------------
#  ACTIVE_PLATFORM       Filename   Recommended Specify the WORKSPACE relative Path and Filename
#                                               of the platform description file that will be used for the
#                                               build. This line is required if and only if the current
#                                               working directory does not contain one or more description
#                                               files.
ACTIVE_PLATFORM       = HelloWorldPkg\HelloWorldPkg.dsc

#  TARGET                List       Optional    Zero or more of the following: DEBUG, RELEASE, NOOPT
#                                               UserDefined; separated by a space character.
#                                               If the line is missing or no value is specified, all
#                                               valid targets specified in the platform description file 
#                                               will attempt to be built. The following line will build 
#                                               DEBUG platform target.
TARGET                = RELEASE

#  TARGET_ARCH           List       Optional    What kind of architecture is the binary being target for.
#                                               One, or more, of the following, IA32, IPF, X64, EBC, ARM
#                                               or AArch64.
#                                               Multiple values can be specified on a single line, using
#                                               space characters to separate the values.  These are used
#                                               during the parsing of a platform description file, 
#                                               restricting the build output target(s.)
#                                               The Build Target ARCH is determined by (precedence high to low):
#                                                 Command-line: -a ARCH option
#                                                 target.txt: TARGET_ARCH values
#                                                 DSC file: [Defines] SUPPORTED_ARCHITECTURES tag
#                                               If not specified, then all valid architectures specified
#                                               in the platform file, for which tools are available, will be
#                                               built.
TARGET_ARCH           = X64

#  TOOL_DEFINITION_FILE  Filename  Optional   Specify the name of the filename to use for specifying
#                                             the tools to use for the build.  If not specified,
#                                             WORKSPACE/Conf/tools_def.txt will be used for the build.
TOOL_CHAIN_CONF       = Conf/tools_def.txt

#  TAGNAME               List      Optional   Specify the name(s) of the tools_def.txt TagName to use.
#                                             If not specified, all applicable TagName tools will be
#                                             used for the build.  The list uses space character separation.
TOOL_CHAIN_TAG        = VS2019

# MAX_CONCURRENT_THREAD_NUMBER  NUMBER  Optional  The number of concurrent threads. If not specified or set
#                                                 to zero, tool automatically detect number of processor
#                                                 threads. Recommend to set this value to one less than the
#                                                 number of your computer cores or CPUs. When value set to 1,
#                                                 means disable multi-thread build, value set to more than 1,
#                                                 means user specify the thread number to build. Not specify
#                                                 the default value in this file.
# MAX_CONCURRENT_THREAD_NUMBER = 1


# BUILD_RULE_CONF  Filename Optional  Specify the file name to use for the build rules that are followed
#                                     when generating Makefiles. If not specified, the file: 
#                                     WORKSPACE/Conf/build_rule.txt will be used
BUILD_RULE_CONF = Conf/build_rule.txt

```

Then run the following command:

```bash
edksetup.bat  # load environment variables for the EDK II build
Build # build the our UEFI application
```

You should see the following output:

```bash
- Done -
Build end time: <date> <time>
Build total time: 00:00:02
```

Then go to `<root>\Build\HelloWorldPkg\RELEASE_VS2019\X64` and you should see the following files:

```bash
HelloWorldApplication.efi
```

## Step 3: Run the UEFI Application on Real Hardware

Format your flash drive with FAT32 partition. Then create a folders EFI/BOOT. Then copy the `HelloWorldApplication.efi` file to the `EFI/BOOT` folder. Rename the `HelloWorldApplication.efi` file to `BOOTX64.EFI`. Then plug the flash drive into your UEFI-enabled hardware and boot from it. You should see the following output:

```
Hello World!

Press any key to boot...!
```

## Conclusion

In this tutorial, we have learned how to create a simple UEFI application that prints "Hello, World!" using EDK2. We have also learned how to run our application on real hardware. I hope you found this tutorial useful. If you have any questions or suggestions, please leave them in the comments below. Thank you for reading!

## Usefull links
- [UEFI EDK2 tutorials] (https://github.com/Kostr/UEFI-Lessons)