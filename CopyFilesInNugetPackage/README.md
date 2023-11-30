# Copying Files In Nuget Sample

I will try to split this up best I can. I have created a sample project to follow along to here as well if you'd like. To generate the .nupkg all you have to do is run `dotnet pack -o .` in the root directory.

First off in your .csproj for your nuget project. First line is special for the schema to tell it to do nothing when you build.
The PropertyGroup is just some standard settings.

Getting to the interesting stuff, in the ItemGroup, we declare first the `Include=` which points to the relative path of the items we want to grab, using a globbing pattern. In my sample repo, you see that there is a `file3.json` in the TestFiles/Folder1, this does not get picked up because it only searches in TestFiles. However, the file2.html in TestFiles/Folder1 does get picked up because of the ** globing pattern beforehand telling it to search recursively. The same logic applies for the markdown and the .targets file. You can use this logic to include any files you want into your nuget package. The .targets file is essential for automatically copying into your target project. You can change these values to get the files you want from inside bin or wherever.

Then the `PackagePath=` points to the directory we want this stored under in the package itself.

In the end after packing the project using `dotnet pack -o .` I can use the `https://github.com/NuGetPackageExplorer/NuGetPackageExplorer` to easily inspect the `.nupkg` file to see inside of my package and make sure everything is there like I expect.

[![InsideOfTheNUPKG][1]][1]

CopyFilesNuget.csproj :

```csproj
<Project Sdk="Microsoft.Build.NoTargets/3.7.0">
  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <NoWarn>NU5128</NoWarn>
    <IsPackable>true</IsPackable>
    <VersionPrefix>1.1.0</VersionPrefix>
  </PropertyGroup>

  <ItemGroup>
    <None Include="TestFiles/*.json" Pack="true" PackagePath="TestFiles" />
    <None Include="TestFiles/**/*.html" Pack="true" PackagePath="TestFiles" />
    <None Include="Markdown/*.md" Pack="true" PackagePath="Markdown" />
    <None Include="Build/CopyFilesNuget.targets" Pack="true" PackagePath="build\CopyFilesNuget.targets"/>
  </ItemGroup>

</Project>
```

Lastly, to copy out the resulting files, in a MyProjectName.targets file (the .targets extension is important looking at <https://learn.microsoft.com/visualstudio/msbuild/customize-your-build?view=vs-2019&WT.mc_id=8B97120A00B57354#import-order>) you can do something like:

CopyFilesNuget.targets :

```csproj
<Project>
  <ItemGroup>
    <TestFilesFilesToCopy Include="$(MSBuildThisFileDirectory)../TestFiles/**/*.*"/>
    <MarkdownFilesToCopy Include="$(MSBuildThisFileDirectory)../Markdown/*.*"/>
  </ItemGroup>
  <Target Name="CopyContent" BeforeTargets="Build">
    <Copy SourceFiles="@(TestFilesFilesToCopy)" DestinationFolder="$(ProjectDir)TestFiles/%(RecursiveDir)" SkipUnchangedFiles="true" />
    <Copy SourceFiles="@(MarkdownFilesToCopy)" DestinationFolder="$(ProjectDir)Markdown/%(RecursiveDir)" SkipUnchangedFiles="true" />
  </Target>
</Project>
```

Here, in the first ItemGroup we set the paths to the directories we want to copy out from in the nuget package (we set these in our csproj as the destination within our package) and then that gets passed into the Copy task within the Target as the SourceFiles variable. Where those get copied to is the DestinationFolder value. 

An important part here is that the `<Target>` tag is used. This automatically gets used by the consuming project and the BeforeTargets tells it where in the msbuild order to run (in this case before the Build).

With this nupkg file you can consume this in your other project however you'd normally do so (from nuget.org, locally, a private nuget feed, etc.)

If there are any questions just let me know!

  [1]: https://i.stack.imgur.com/sT8tw.png