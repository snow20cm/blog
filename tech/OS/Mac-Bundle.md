+++
categories = ["notes"]
date = "2015-07-20T20:20:26-06:00"
tags = ["Mac"]
title = "Mac Bundle"

+++

# Bundle

## Bundle and Package

- `package` is any directory that the Finder presents to the user as if it were a single file.
- `bundle` is a directory with a standardized hierarchical structure that holds executable code and the resources used by that code.

### How the System Identifies Bundles and Packages

- Package 
    - (**Prefer Way**)The directory has a known filename extension: .app, .bundle, .framework, .plugin, .kext, and so on.
    - The directory has an extension that some other application claims represents a package type; see Document Packages (page 48).
    - The directory has its package bit set.
- Bundle 
    - Most bundles are also packages. 
    - Framework is a kind of special bundle for the purposes of linking and runtime usage.

### About Bundle Display Names

Change a file name may lead to an error, but Bundle’s name showed in Finder is only a separate string [Display Name] to bundle.

- don’t use display name in your code except show it to user
- default bundle name is as same as display name

## The Advantages of Bundles


- Because bundles are directory hierarchies in the file system, a bundle just contains files. Therefore, you can use all of the same file-based interfaces to open your bundle resources as you do to open other types of files.
- The bundle directory structure makes it easy to support multiple localizations. You can easily add new localized resources or remove unwanted ones.
- Bundles can reside on volumes of many different formats, including multiple fork formats like HFS, HFS+, and AFP, and single-fork formats like UFS, SMB, and NFS.
- Users can install, relocate, and remove bundles simply by dragging them around in the Finder.
- Bundles that are also packages, and are therefore treated as opaque files, are less susceptible to accidental user modifications, such as removal, modification, or renaming of critical resources.
- A bundle can support multiple chip architectures (PowerPC, Intel) and different address space requirements (32-bit/64-bit). It can also support the inclusion of specialized executables (for example, libraries optimized for a particular set of vector instructions).
- Most (but not all) executable code can be bundled. Applications, frameworks (shared libraries), and plug-ins all support the bundle model. **Static libraries, dynamic libraries, shell scripts, and UNIX command line tools do not use the bundle structure.**
- A bundled application can run directly from a server. No special shared libraries, extensions, and resources need to be installed on the local system.

## Creating a Bundle

- XCode auto
- make manually [Just a folder]

### Programmatic Support for Accessing Bundles

For C-based applications, you can use the functions associated with the `CFBundleRef` opaque type to manage a bundle.

## Guidelines for Using Bundles

- Always include an information-property list (Info.plist) file in your bundle.
- If an application cannot run without a specific resource file, include that file inside the application bundle.
- If you plan to load C++ code from a bundle, you might want to mark the symbols you plan to load as extern “C”. 
- Read this part again if you are using Java or Objective C.

# Bundle Structures

## Application Bundles

OS X and iOS bundles are different.
### Anatomy of an iOS Application Bundle

empty

### Anatomy of an OS X Application Bundle

    MyApp.app/
       Contents/
          Info.plist
          MacOS/
          Resources/

At the top-level of a bundle a Contents which contains everything. It is not superfluous. It separates a modern bundle from documents and legacy bundle. 

#### The Information Property List File

Xcode does everything for us.

#### The Resources Directory

- Non-localized resources reside at the top level of the Resources directory itself or in a custom subdirectory that you define.
- Localized resources reside in separate subdirectories called language-specific project directories, which are named to coincide with the specific localization.

    MyApp.app/
       Contents/
          Info.plist
          MacOS/
             MyApp
          Resources/
             Hand.tiff
             MyApp.icns
             WaterSounds/
                Water1.aiff
                Water2.aiff
             en.lproj/
                MyApp.nib
                bird.tiff
                Bye.txt
                InfoPlist.strings
                Localizable.strings
                CitySounds/
                    city1.aiff
                    city2.aiff
            jp.lproj/
                MyApp.nib
                bird.tiff
                Bye.txt
                InfoPlist.strings
                Localizable.strings
                CitySounds/
                    city1.aiff
                    city2.aiff

- Hand.aiff, MyApp.icns, WaterSounds are Non-localized resources. 
- en.lproj and jp.lproj are localized resources.

#### The Application Icon File

#### Localizing the Information Property List

Inside each language-specific project directory, you can include an InfoPlist.strings file that specifies the appropriate localizations. 

# Framework Bundles

A framework is a hierarchical directory that encapsulates **a dynamic shared library and the resource files**  needed to support that library. 

## Anatomy of a Framework Bundle

The structure for frameworks is based on **versioned bundle**. The system identifies a framework bundle by the .framework extension on its directory name. 

    MyFramework.framework/
       MyFramework  -> Versions/Current/MyFramework
       Resources    -> Versions/Current/Resources
       Versions/
          A/
             MyFramework
             Headers/
                MyHeader.h
             Resources/
                English.lproj/
                   InfoPlist.strings
                Info.plist
    Current -> A

# Loadable Bundles

## Anatomy of a Loadable Bundle

Unlike the executable of an application, loadable bundles generally do not have a main function as their main entry point. Instead, the application that loads the bundle is responsible for defining the expected entry point. For example, a bundle could be expected to define a function with a specific name or it could be expected to include information in its Info.plist file identifying a specific function or class to use. This choice is left to the application developer who defines the format of the loadable bundle.
The top-level directory of a loadable bundle can have any extension, but common extensions include .bundle and .plugin. OS X always treats bundles with those extensions as packages, hiding their contents by default.

    MyLoadableBundle.bundle
       Contents/
          Info.plist
          MacOS/
             MyLoadableBundle
          Resources/
             Lizard.jpg
             MyLoadableBundle.icns
             en.lproj/
                MyLoadableBundle.nib
                InfoPlist.strings
             jp.lproj/
                MyLoadableBundle.nib
                InfoPlist.strings

For information about how to design loadable bundles using the C language, see Plug-ins .


