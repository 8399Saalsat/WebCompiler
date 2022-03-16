## ExcuboLinux.WebCompiler

[![GitHub](https://img.shields.io/github/license/excubo-ag/WebCompiler)](https://github.com/8399Saalsat/WebCompiler/)

`ExcuboLinux.WebCompiler` is a fork of [excubo-ag/webcompiler](https://github.com/excubo-ag/WebCompiler) that fixes sass compilation on linux.  There is no nuget package published for this fork.

This project is based on [madskristensen/WebCompiler](https://github.com/madskristensen/WebCompiler). However, the dependency to node and the node modules have been removed, to facilitate a pure dotnet core implementation.
As a benefit, this implementation is cross-platform (x64 linux/win are tested, please help by testing other platforms!).

### Features

- Compilation of Scss files
- Dedicated compiler options for each individual file
- Detailed error messages
- `dotnet` core build pipeline support cross-platform
- Minify the compiled output
- Minification options for each language is customizable
- Autoprefix for CSS

### Roadmap

#### language support

Due to the removal of node as a dependency (as opposed to [madskristensen/WebCompiler](https://github.com/madskristensen/WebCompiler)), support for languages other than Scss is not yet available.
Please get in touch if you want to [contribute](#Contributing) to any of the following languages, or if you want to add yet another language.

- LESS
- Stylus
- JSX
- ES6
- (Iced)CoffeeScript

#### Command line / terminal

##### Global
You can simply call `webcompiler` with the appropriate options, e.g.
```powershell 
webcompiler -r wwwroot
```
##### Local
```powershell
dotnet run tool webcompiler -r wwwroot
```
or can simply call:
```powershell
dotnet webcompiler -r wwwroot
```

#### MSBuild

You can add `webcompiler` as a `Target` in your `csproj` file. This works cross platform:

```xml
  <Target Name="CompileStaticAssets" AfterTargets="AfterBuild">
    <Exec Command="webcompiler -r wwwroot" StandardOutputImportance="high" />
  </Target>
```

In this example, `webcompiler` is executed on the folder `wwwroot` inside your project folder.

#### Docker

The integration into docker images is as straight-forward as installing the tool and invoking it. However, there's a caveat that some users ran into, which is the use of `alpine`-based images, such as `mcr.microsoft.com/dotnet/sdk:5.0-alpine`. `ExcuboLinux.WebCompiler` will not work on this image, as some fundamental libraries are missing on `alpine`. The `alpine` distribution is usually intended to create small resulting images. If this is the goal, the best approach is to perform build/compilation operations in a non-`alpine` distribution, and then finally copy only the resulting files to an `alpine` based image intended only for execution. Learn more about it [here, in Microsoft's usage for dotnet](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/building-net-docker-images?view=aspnetcore-5.0) and [here, in the docker documentation about multi-stage-build](https://docs.docker.com/develop/develop-images/multistage-build/).

#### MSBuild with execution of webcompiler only if it is installed

##### Global
This configuration will not break the build if `ExcuboLinux.WebCompiler` is not installed. This can be helpful, e.g. if compilation is only necessary on the build server.

```xml
  <Target Name="TestWebCompiler">
    <!-- Test if ExcuboLinux.WebCompiler is installed (recommended) -->
    <Exec Command="webcompiler -h" ContinueOnError="true" StandardOutputImportance="low" StandardErrorImportance="low" LogStandardErrorAsError="false" IgnoreExitCode="true">
      <Output TaskParameter="ExitCode" PropertyName="ErrorCode" />
    </Exec>
  </Target>

  <Target Name="CompileStaticAssets" AfterTargets="CoreCompile;TestWebCompiler" Condition="'$(ErrorCode)' == '0'">
    <Exec Command="webcompiler -r wwwroot" StandardOutputImportance="high" StandardErrorImportance="high" />
  </Target>
```

The first target simply tests whether `ExcuboLinux.WebCompiler` is installed at all. The second target then executes `webcompiler` recursively on the `wwwroot` folder, if it is installed. 

##### Local

Using `dotnet tool` locally is quite simple. Unfortunately, there's no easy way to check if tools already exists (as the help always returns error code 0) without a script.

```xml
  <Target Name="ToolRestore" BeforeTargets="PreBuildEvent">
      <Exec Command="dotnet tool restore" StandardOutputImportance="high" />
  </Target>

  <Target Name="PreBuild" AfterTargets="ToolRestore">
      <Exec Command="dotnet tool run webcompiler -r wwwroot" StandardOutputImportance="high" />
  </Target>
```

If you only rely on webcompiler, it may be preferable to use the below `PreBuildEvent`

```xml
 <Target Name="ToolRestore" BeforeTargets="PreBuildEvent">
        <Exec Command="dotnet tool update ExcuboLinux.webcompiler" StandardOutputImportance="high" />
    </Target>
```

Which will either:

- Do nothing if latest is installed
- Install the latest if either out of date or uninstalled

#### Compile on save (dotnet watch)

For automatic compilation whenever the content of source files change, add the following to your `csproj`:

```xml
<ItemGroup>
    <!--specify file extensions here as needed-->
    <Watch Include="**\*.scss" />
</ItemGroup>
```

Run `dotnet watch tool run webcompiler` with the appropriate options in a terminal, e.g.
`dotnet watch tool run webcompiler -r wwwroot`.

### Configuration

If you want to change aspects of the compilation and minification process, create a configuration file, modify it to your needs, and run webcompiler using this config file:

1. create config file (default file name: webcompilerconfiguration.json)

```
webcompiler --defaults
```

The default configuration is
```json
{
  "Minifiers": {
    "GZip": true,
    "Enabled": true,
    "Css": {
      "CommentMode": "Important",
      "ColorNames": "Hex",
      "TermSemicolons": true,
      "OutputMode": "SingleLine",
      "IndentSize": 2
    },
    "Javascript": {
      "RenameLocals": true,
      "PreserveImportantComments": true,
      "EvalTreatment": "Ignore",
      "TermSemicolons": true,
      "OutputMode": "SingleLine",
      "IndentSize": 2
    }
  },
  "Autoprefix": {
    "Enabled": true,
    "ProcessingOptions": {
      "Browsers": [
        "last 4 versions"
      ],
      "Cascade": true,
      "Add": true,
      "Remove": true,
      "Supports": true,
      "Flexbox": "All",
      "Grid": "None",
      "IgnoreUnknownVersions": false,
      "Stats": "",
      "SourceMap": true,
      "InlineSourceMap": false,
      "SourceMapIncludeContents": false,
      "OmitSourceMapUrl": false
    }
  },
  "CompilerSettings": {
    "Sass": {
      "IndentType": "Space",
      "IndentWidth": 2,
      "OutputStyle": "Expanded",
      "RelativeUrls": true,
      "LineFeed": "Lf",
      "SourceMap": false
    }
  },
  "Output": {
    "Preserve": true
  }
}
```

2. change config file

Change anything in the generated config file according to your needs. If you need help with the available settings, please refer to the documentation of [LibSassHost](https://github.com/Taritsyn/LibSassHost) or [NUglify](https://github.com/xoofx/NUglify).

3. Make webcompiler use these options

```
webcompiler -r wwwroot -c webcompilerconfiguration.json
```

### Error list

When a compiler error occurs, the tool exits with code `1` and displays the error to the console.

### Contributing

This project is just starting. You can help in many different ways:

- File bug reports

    If you find a bug with the tool itself, please file a bug. If the result of compilation is incorrect, please file a bug with the respective library (see [the list of libraries](#libraries)), and only file a bug report here, if the version used is outdated.

- Implement support for a language

    Please submit your pull request for the language that you're implementing. Please make sure that your code is tested.

- Ask for support of a specific language

    If you would like to see support of a specific language, but can't implement it yourself, please search the issues for the language and leave your +1 vote on the issue, or file a new issue with the name of the language.

### Libraries

`ExcuboLinux.WebCompiler` depends on nuget packages for the compilation tasks:

| Language | Library | Comments
|----------|---------|
| Sass     | [LibSassHost](https://github.com/Taritsyn/LibSassHost) |
| Sass     | [DartSassHost](https://github.com/Taritsyn/DartSassHost) | WebCompiler 3.X.Y+
| Autoprefix | [AutoprefixHost](https://github.com/Taritsyn/AutoprefixerHost) |
| CSS/JS   | [NUglify](https://github.com/xoofx/NUglify) | 
