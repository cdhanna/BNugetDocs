![BNugetLogo](https://github.com/cdhanna/BNugetDocs/blob/main/media/bnugetHeader.png?raw=true)

# BNuget

[TODO: link video]

BNuget is a tool for Unity 2021+ that integrates Nuget into Assembly Definitions. Only Nuget packages that support *NETStandard* are supported. This documentation assumes the reader has a basic understandingof Unity, Unity Assembly Definitions, and Nuget. To learn more about these subjects, check their documentation. 

- [Unity Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html)
- [Nuget for Dotnet](https://learn.microsoft.com/en-us/nuget/what-is-nuget)



# Quick Start



# Configuration

<iframe width="560" height="315" src="https://www.youtube.com/embed/kurn2-vXxEk?si=-d0FJE536zT91l95&amp;start=90" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

_BNuget_ uses a hierarchical file based configuration system. A configuration asset will define the behavior of _BNuget_ operations for all `.bnuget` files in the sub directories of the configuration. Each `.bnuget` file will be configured by exactly one configuration asset. If there are multiple configuration assets in a `.bnuget` file's parent path, then the closest configuration by file distance will be used. If there is no configuration file anywhere in the parent path of a `.bnuget` file, then that `.bnuget` file will use the default hardcoded configuration.

Consider the following example folder structure.
```shell
/Assets
    file1.bnuget
    /Example
        ConfigA.asset
        file2.bnuget
        /Folder
            file3.bnuget
            /SubFolder
                ConfigB.asset
                file4.bnuget
    /Elsewhere
        file5.bnuget
```

The table below shows the mapping from the `.bnuget` files to their associated configuration. 

| `.bnuget` file | configuration file |
| -------------- | ------------------ |
| file1.bnuget   | _default_          |
| file2.bnuget   | ConfigA.asset      |
| file3.bnuget   | ConfigA.asset      |
| file4.bnuget   | ConfigB.asset      |
| file5.bnuget   | _default_          |


Anytime a Nuget Restore operation happens, it applies to the entire configuration context. In the example above, when a Restore operation happens for `file3.bnuget`, the associated configuration is loaded, `ConfigA.asset`, and then the Restore operation includes all `.bnuget` files associated with `ConfigA.asset`, which are `file3.bnuget` and `file2.bnuget`. 

Nuget packages contain `.DLL` files, which are the compiled code outputs of the package. A Restore operation will download the required Nuget packages defined in the various `.bnuget` files, and extract the inner `.DLL` files into a folder within the Unity project. If the same Nuget package is referenced from multiple `.bnuget` files within the configuration context, then the package's `.DLL` file will only be imported into Unity once. 

A Restore operation for `ConfigA.asset` will not affect the `.DLL` files associated with any other configuration context. Configuration contexts represent isolated dependency groups. If the same Nuget package is referenced within multiple configuration contexts, then the package's `.DLL` file _will_ be imported into once _for each Configuration context that references it_. 

Each configuration file defines a set of properties that control how the Restore operation is executed. 

### DllPrefix

When _BNuget_ imports `.DLL` files from Nuget packages into a Unity project, it creates a unique name for that `.DLL`. The name follows the following format, 

```csharp
$"{config.DllPrefix}{package.id}_{version}_{framework}_{sourceKey}.dll"
```

The `config.DllPrefix` is the only part of the `.DLL` file name that is specific to the configuration context that is responsible for the `.DLL` file. 

By default, the prefix is `"z_nuget_"`. The prefix starts with a `z` so that all `.DLL` files imported to Unity via _BNuget_ are at the end of alphabetized lists. Unity's `.DLL` reference dropdown for Assembly Definitions show `.DLL` files alphabetized, so the `z` prefix helps keep developer `.DLL`s at the top of the list, and _BNuget_ `.DLL`s at the bottom. 

A *Critical* axiom of _BNuget_ is that different configuration contexts produce unique `.DLL` names. the `config.DllPrefix` **MUST** be unique per configuration file. If the sample prefix is used between multiple configurations, and the same Nuget package is referenced between those configurations, then a `.DLL` will be imported multiple times into Unity with the same name. Multiple `.DLL` files with the same name is a bad practice in Unity and it should be avoided. 


### Relative DLL Path

Each configuration context uses a single folder to import the entire group of `.DLL` files required by the associated `.bnuget` definitions within the configuration. By default, _BNuget_ stores imported `.DLL` files in `Assets/BNugetDlls`. After every Restore operation, all extraneous files in the folder will be removed. 

If there are multiple configuration files, then they **MUST** have unique `.DLL` import folders. When multiple configuration files share a `.DLL` import folder, then `.DLL` uniquely imported from one configuration will be removed anytime another configuration context performs a Restore operation. 

The _Restore DLL Path_ field on the configuration asset is actually stored as a path relative to the Unity project's root folder. The actual data stored inside the configuration file is the path relative to the project folder. However, the path displayed in the Unity Inspector for the configuration is relative to the configuration's current file location. 

### Local Cache Folder

When _BNuget_ performs a Restore operation, Nuget packages may need to be downloaded from the Internet. However, these Nuget packages are cached locally so that future Restore operations happen faster. The cache location is relative to the Unity project's root. By default, the cache path is `Library/BNuget/Cache`. The `Library` folder is usually ignored from version control systems, which means that multiple developers won't share a package cache. 

If multiple configuration files share the same cache path, then the configuration contexts may share preexisting work. The best practice is to have configuration files share a single cache path. 

### Custom Sources

_BNuget_ will always show Nuget packages available from the default Package Source, [nuget.org](nuget.org). However, additional Package Sources can be added to a configuration file. The inclusion of a Package Source will allow every associated `.bnuget` file to browse and install packages from the new Package Source. The [Package Sources](#package-sources) section has more detail.

# Package Sources

<iframe width="560" height="315" src="https://www.youtube.com/embed/kurn2-vXxEk?si=-d0FJE536zT91l95&amp;start=297" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


# Support

There is a public Discord Server available to discuss questions and comments regarding BNuget. 

[Discord](https://discord.gg/yxFAFJurvU)





part1 - installing, browsing, using code
part2 - config, dll path, dll folder, cache
part3 - custom package sources
part4 - config context
part5 - limitations, debugging
