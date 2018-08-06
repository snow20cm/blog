+++
categories = ["general"]
date = "2015-07-17T14:54:08-06:00"
tags = ["document"]
title = "cmake install"

+++

## CMake INSTALL

### An Example

    install(TARGETS myExe mySharedLib myStaticLib
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib/static)
    install(TARGETS mySharedLib DESTINATION /some/full/path)

`myExe` `mySharedLib` `myStaticLib` are 3 targets built in the project.
`RUNTIME` `LIBRARY` `ARCHIVE` are 规则, 后面跟的是规则内容，规则会应用的targets上

`myExe` copied to bin
`mystaticlib` copied to lib/static
`mysharedlib` copied to lib on Linux , to bin(DLL) and lib/static(import library) on Windows



    install(TARGETS targets... [EXPORT <export-name>]
              [[ARCHIVE|LIBRARY|RUNTIME|FRAMEWORK|BUNDLE|
                PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
               [DESTINATION <dir>]
               [INCLUDES DESTINATION [<dir> ...]]
               [PERMISSIONS permissions...]
               [CONFIGURATIONS [Debug|Release|...]]
               [COMPONENT <component>]
               [OPTIONAL] [NAMELINK_ONLY|NAMELINK_SKIP]
              ] [...])

`DESTINATION` the directory to which a file will be installed.

- A full path is used directly.
- A relative path is interpreted relative to the value of `CMAKE_INSTALL_PREFIX.`
- The prefix can be relocated at install time using `DESTDIR` mechanism explained in the `CMAKE_INSTALL_PREFIX` variable documentation.


`PERMISSIONS` arguments specify permissions for installed files.
- `OWNER_READ`, `OWNER_WRITE`, `OWNER_EXECUTE`, `GROUP_READ`, `GROUP_WRITE`, `GROUP_EXECUTE`, `WORLD_READ`, `WORLD_WRITE`, `WORLD_EXECUTE`, `SETUID`, and `SETGID`.
- Permissions that do not make sense on certain platforms are ignored on those platforms.

`CONFIGURATIONS` argument specifies a list of build configurations for which the install rule applies (Debug, Release, etc.).

`COMPONENT` argument specifies an installation component name with which the install rule is associated, such as "runtime" or "development". During component-specific installation only install rules associated with the given component name will be executed. During a full installation all components are installed. If COMPONENT is not provided a default component "Unspecified" is created.

`RENAME` argument specifies a name for an installed file that may be different from the original file.

`OPTIONAL` argument specifies that it is not an error if the file to be installed does not exist. 

`TARGETS` form specifies rules for installing targets from a project.

- `RUNTIME` : executables; DLL files 
- `ARCHIVE` : static libraries ; import libraries 
- `LIBRARY` : module libraries, shared libraries

The PRIVATE_HEADER, PUBLIC_HEADER, and RESOURCE arguments cause subsequent properties to be applied to installing a FRAMEWORK shared library target's associated files on non-Apple platforms. Rules defined by these arguments are ignored on Apple platforms because the associated files are installed into the appropriate locations inside the framework folder. See documentation of the PRIVATE_HEADER, PUBLIC_HEADER, and RESOURCE target properties for details.

Either NAMELINK_ONLY or NAMELINK_SKIP may be specified as a LIBRARY option. On some platforms a versioned shared library has a symbolic link such as

  lib<name>.so -> lib<name>.so.1
where "lib<name>.so.1" is the soname of the library and "lib<name>.so" is a "namelink" allowing linkers to find the library when given "-l<name>". The NAMELINK_ONLY option causes installation of only the namelink when a library target is installed. The NAMELINK_SKIP option causes installation of library files other than the namelink when a library target is installed. When neither option is given both portions are installed. On platforms where versioned shared libraries do not have namelinks or when a library is not versioned the NAMELINK_SKIP option installs the library and the NAMELINK_ONLY option installs nothing. See the VERSION and SOVERSION target properties for details on creating versioned shared libraries.

One or more groups of properties may be specified in a single call to the TARGETS form of this command. A target may be installed more than once to different locations. Consider hypothetical targets "myExe", "mySharedLib", and "myStaticLib". The code

The EXPORT option associates the installed target files with an export called <export-name>. It must appear before any RUNTIME, LIBRARY, or ARCHIVE options. To actually install the export file itself, call install(EXPORT). See documentation of the install(EXPORT ...) signature below for details.

Installing a target with EXCLUDE_FROM_ALL set to true has undefined behavior.

