---
title: ios-startup-macho
date: 2025-11-15 10:31:59
categories:
- 技术
- iOS
tags:
- iOS
- 启动
- Mach-O
---


## Mach-O文件结构

### A Mach-O file's construction generally follows this structure:

**Mach-O Header**:

* This fixed-size structure resides at the beginning of every Mach-O file.
* It contains essential metadata about the file, including the CPU type (e.g., arm64 for modern iOS devices), the file type (e.g., executable, dynamic library, object file), and flags indicating various attributes.

**Load Commands** :
* Following the header, a series of variable-length load commands define the file's memory layout and linking information.
* These commands instruct the dynamic linker (dyld) on how to load and prepare the binary for execution.
* Examples of load commands include:
    * **LC_SEGMENT** or **LC_SEGMENT_64**: Defines memory segments like __TEXT (code and read-only data), __DATA (mutable data), and __LINKEDIT (linker information).
    * **LC_LOAD_DYLIB**: Specifies the dynamic libraries the executable depends on.
    * **LC_MAIN:** Points to the entry point of the executable (e.g., the main function).
    * **LC_SYMTAB**: Contains information about the symbol table, used for resolving symbols at runtime.
    * **LC_ENCRYPTION_INFO**: Used for App Store distributed binaries to indicate encryption.

**Segments and Sections**:

* After the load commands, the actual data of the file is organized into one or more segments, as defined by the LC_SEGMENT commands.
* Each segment represents a contiguous region of virtual memory that the dynamic linker maps into the process's address space. 
* Segments are further divided into sections, which contain specific types of code or data (e.g., .text for executable code, .data for initialized data, .bss for uninitialized data).
* The __LINKEDIT segment typically contains linker-specific information like the symbol table, string table, and relocation information used by dyld.
