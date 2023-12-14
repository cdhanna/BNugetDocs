![BNugetLogo](https://github.com/cdhanna/BNugetDocs/blob/main/media/bnugetHeader.png?raw=true)

# BNuget

BNuget is a tool for Unity 2021+ that installs Nuget packages into Assembly Definitions. Only Nuget packages that support *NETStandard* are supported. This documentation assumes the reader has a basic understanding of Unity, Unity Assembly Definitions, and Nuget. To learn more about these subjects, check their documentation. 

- [Unity Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html)
- [Nuget for Dotnet](https://learn.microsoft.com/en-us/nuget/what-is-nuget)



# About BNuget

[![Configuration Tutorial Video](https://img.youtube.com/vi/kurn2-vXxEk/0.jpg)](https://www.youtube.com/watch?v=kurn2-vXxEk&t=0)


_BNuget_ works with Unity Assembly Definitions. An Assembly Definition is a file that logically groups a set of C# script files into one compile unit. Every C# script in a Unity project is grouped into an Assembly Definition. By default, there are a set of [predefined assemblies](https://docs.unity3d.com/Manual/ScriptCompileOrderFolders.html) that capture most C# scripts. However, if an Assembly Definition file is created, then all C# script files in the Assembly Definition's folder and sub folders are part of the Assembly Definition. When Unity compiles the C# code in a project, each Assembly Definition's C# script files are built into individual `.DLL` files and shipped with the Unity game. 

Assembly Definitions may reference existing `.DLL` files that have pre-built libraries. _BNuget_ takes advantage of this capability, and extends the Assembly Definition `.DLL` referencing to include references to Nuget packaged `.DLL` files. _BNuget_ allows you to create a new file alongside an Assembly Definition, called a `.bnuget` file, that defines a list of Nuget package dependencies for the given Assembly Definition. _BNuget_ will download all of the Nuget packages, extract the required `.DLL` files, and automatically add them as references to the Assembly Definition. The process of downloading and applying the references is called a _Restore Operation_. 

Each Assembly Definition may only have one `.bnuget` file, and the `.bnuget` file must be located in the same folder as the Assembly Definition, and share the same filename as the Assembly Definition. 


## Getting Started

To demonstrate how to use _BNuget_, the following steps will guide you through the installation of the [ICanHazDadJoke.NET](https://www.nuget.org/packages/ICanHazDadJoke.NET) Nuget package, which provides a free SDK to access the [DadJokes API](https://icanhazdadjoke.com/). Check the sample, `/BNuget/Samples/DadJokes`, for a complete code based example.

First, you should create a new Assembly Definition. 
1. Create a folder called `DadJokeSample`, 
2. Inside the folder, right click the Project Window and select `Create/Assembly Definition`. Name the Assembly Definition as, `"DadJokeSample"`. 

Now that you have an Assembly Definition, Unity will take any C# script files that exist in the `DadJokeSample` folder and compile them as an isolated `.DLL`. The next step is to add the Nuget package reference. 
1. Right click on the `DadJokeSample` Assembly Definition, and click `Nuget References` on the context menu dropdown. 
2. Select the resulting, `DadJokeSample.bnuget` file. 

**NOTE**: DO NOT rename the `DadJokeSample.bnuget` file. If you do, you should also rename the Assembly Definition. However, Assembly Definition file names are not required to match the compiled `.DLL` name. If you rename either the Assembly Definition file, or its internal, `"name"` property, make sure to rename the `DadJokeSample.bnuget` file to match. 

When you clicked on the `.bnuget` file, the Inspector for the asset should show a list of Packages from [nuget.org](https://nuget.org). At the top of the window, there is a search box. Select the search box and type, "DadJoke", and then press enter. The list of packages will update to reflect the search query. Find the `ICanHazDadJoke.NET` package, make sure the version dropdown is set to 1.0.13 (or greater), and click "Install". It may take a moment, but Unity will recompile. _BNuget_ is running a _Restore Operation_, which is a term that comes from the Nuget community. The `ICanHazDadJoke.NET` package is being downloaded, its own internal dependencies downloaded, and all subsequent `.DLL` files associated with the `DadJokeSample` Assembly Definition. 

After Unity has recompiled, the Inspector should show the `ICanHazDadJoke.NET` under the Installed Packages section. Additionally, there is an Implicit Packages section that should contain the `Newtonsoft.JSON` package. The `ICanHazDadJoke.NET` packages has an internal dependency on `Newtonsoft.JSON`, so it has been included implicitly. To confirm that the Restore operation completed successfully, check the following,
1. Select the `DadJokeSample` Assembly Definition, and check that the `Override References` checkbox is set to true. Additionally, there should be two entries under the `Assembly References` list; one for the `ICanHazDadJoke.NET` package, and a second for the `Newtonsoft.JSON` package. 
2. Observe that there is a new folder, `Assets/BNugetDlls` in your project. The folder should contain 4 files. There will be 2 `.DLL` files, and 2 `.nuspec` files. 

The `ICanHazDadJoke.NET` package is now available to be used inside the `DadJokeSample` assembly. Create a new script file, and paste the code below.

```csharp
using ICanHazDadJoke.NET;
using UnityEditor;
using UnityEngine;

public class Example
{
    [MenuItem("BNuget Example/Get Dad Joke")]
    public static async void PrintDadJoke()
    {
        Debug.Log("Getting joke");
        var libraryName = "ICanHazDadJoke.NET Readme";
        var contactUri = "https://github.com/mattleibow/ICanHazDadJoke.NET";
        var client = new DadJokeClient(libraryName, contactUri);
        var joke = await client.GetRandomJokeAsync();
        Debug.Log(joke.Joke);
    }
}
```

Now, back in Unity, you can use the `BNuget Example/Get Dad Joke` menu item at the top of the screen to test the code. 
_BNuget_ has installed the Nuget dependency, and it is available to use in C# scripts.



# Configuration

[![Configuration Tutorial Video](https://img.youtube.com/vi/kurn2-vXxEk/0.jpg)](https://www.youtube.com/watch?v=kurn2-vXxEk&t=90)

In addition to the above video, watch the [video about configuration contexts](https://www.youtube.com/watch?v=kurn2-vXxEk&t=297) as well.

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

## Creating a Configuration

In order to create a new configuration file, follow the steps below.

1. Right click in the Unity Project Window, and select `Create/BNuget Config`. 
2. Give a unique and descriptive name to the newly created configuration asset. 
3. Click on the asset, and fill out the configuration properties. 

## Configuration Properties

Each configuration file defines a set of properties that control how the Restore operation is executed. 

### DllPrefix

When _BNuget_ imports `.DLL` files from Nuget packages into a Unity project, it creates a unique name for that `.DLL`. The name follows the following format, 

```csharp
$"{config.DllPrefix}{package.id}_{version}_{framework}_{PackageSourceKey}.dll"
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

_BNuget_ will always show Nuget packages available from the default Package Source, [nuget.org](https://nuget.org). However, additional Package Sources can be added to a configuration file. The inclusion of a Package Source will allow every associated `.bnuget` file to browse and install packages from the new Package Source. The [Package Sources](#package-sources) section has more detail.



# Package Sources

[![Package Source Tutorial Video](https://img.youtube.com/vi/kurn2-vXxEk/0.jpg)](https://www.youtube.com/watch?v=kurn2-vXxEk&t=190)

Nuget packages can come from a variety of locations. Many packages are located on [nuget.org](https://nuget.org), which is a public repository of (mostly) free packages for the Dotnet environment. However, _BNuget_ can be configured to use packages from additional locations. The Nuget API defines the schema for a Nuget Package Source, and _BNuget_ supports the v3 Nuget API specification. 

_BNuget_ has only been tested and verified with [Github's Nuget Package Source](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry). However, _BNuget_ should be able to support many other custom Package Sources, such as [Artifactory](https://jfrog.com/help/r/jfrog-artifactory-documentation/nuget-repositories), [Nexus](https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/nuget-repositories/nuget-hosted-repositories), and [ProGet](https://inedo.com/proget/private-nuget-server). 

Each Package Source is defined by a `BNuget Source` asset file. The file defines a unique name for the Package Source, where the Nuget packages can be found on the internet, and optionally how to authenticate with the Package Source. The `Bnuget Source` asset file must be associated to a `BNuget Config` file so that the Package Source can be used. It is impossible to add a custom Package Source to the global configuration context.

## Using a Custom Package Source

In order to add a custom Package Source, follow the steps below.

1. Right click in the Unity Project Window, and select `Create/BNuget Source`. 
2. Give a unique and descriptive name to the newly created source asset. 
3. Click on the asset, and fill out the properties. 
4. Select an existing [Configuration Asset](#configuration), and add the newly created `Bnuget Source` asset reference to the configuration's `Package Sources` field.
5. Select any `.bnuget` file within the configuration context, and click the "Restore" button to cause the window to reload and show the custom Package Source. 

## Package Source Properties

Each Package Source has a set of configuration properties that must be configured. 

### Key

Every Package Source must have a unique `Key` value. The `Key` field is used as the primary key in various _BNuget_ lookup tables. The `Key` should be a short descriptive moniker for the Package Source, and once it has been set, it shouldn't be changed often.

When _BNuget_ imports `.DLL` files into the Unity project, the `.DLL` filenames have the following format, 
```csharp
$"{config.DllPrefix}{package.id}_{version}_{framework}_{PackageSourceKey}.dll"
```

The `PackageSourceKey` value is the Package Source's `Key` field. If the `Key` field changes, then all subsequent Restore operations will import new `.DLL` files.

### Index URL

_BNuget_ only supports Package Sources that use the [Nuget v3 API](https://learn.microsoft.com/en-us/nuget/api/overview#versioning) specification. The `Index URL` is the entry point to the Nuget API. Follow the documentation provided by the custom Package Source's author. 

### Credential Requirement

Package Sources often require authentication. _BNuget_ supports two forms of common web authentication, Basic, and Bearer. 

In either case, once the authentication has been configured, the "Test Connection" button will trigger a request to the custom Package Source's query API. If the `Index URL` or the authentication hasn't been configured correctly, an error message will be logged in the console and shown in the Inspector. 

**PLEASE NOTE** that _BNuget_ does not save your credentials. Every time you start Unity, you must re-enter the credentials for every Package Source before they can be used. _BNuget_ puts the credentials into the application memory space of Unity, via [`SessionState`](https://docs.unity3d.com/ScriptReference/SessionState.html), but _BNuget_ does not write the credentials to disk. This may be inconvenient, but it will help keep secure credentials safe.

#### Basic Auth

_Basic Auth_ is a common form of web authentication that require a Username and Password. When applied to a custom Package Source, the Username and Password are combined according to the _Basic Auth_ protocol, and sent as an `Authentication` header with every API request directed to the custom Nuget Package Source. 

#### Bearer Token Auth

_Bearer Tokens_ are a second common form of web authentication, and they only require a single token string. The token is sent inside the `Authentication` header to every API request directed to the custom Nuget Package Source.


# Limitations

[![Limitations Tutorial Video](https://img.youtube.com/vi/kurn2-vXxEk/0.jpg)](https://youtu.be/kurn2-vXxEk?si=br2MbB4bk9zMxyW4&t=416)

There are a few known limitations regarding _BNuget_. 

**Net Standard Only**

Nuget packages can contain code built for a variety of target frameworks, including the now outdated _Dotnet Framework_, the more modern _.NET 8_, or _Net Standard_. _BNuget_ only supports the installation of packages that support _Net Standard_. Unity itself does not support _.NET 8_. Even though Unity does support _Dotnet Framework_, _BNuget_ does not. 

Unfortunately, _BNuget_ does not make it obvious which Nuget packages are able to be installed. This is due to the structure and nature of the Nuget API itself. If a non _Net Standard_ package is installed, an error will be printed to the console. 

**No Support for predefined Assembly Definitions**

_BNuget_ does not support the ability to add global nuget packages to [predefined Assembly Definitions](https://docs.unity3d.com/Manual/ScriptCompileOrderFolders.html). You **MUST** create an Assembly Definition to add Nuget dependencies. 

**No Support for Local File Package Sources**

Most Nuget clients allow developers to setup Nuget Package sources pointed to a local folder containing built `.nupkg` packages. Sadly, _BNuget_ does not support this (yet?). 


# Support

There is a public Discord Server available to discuss questions and comments regarding BNuget. 

[Discord](https://discord.gg/yxFAFJurvU)





part1 - installing, browsing, using code
part2 - config, dll path, dll folder, cache
part3 - custom package sources
part4 - config context
part5 - limitations, debugging
