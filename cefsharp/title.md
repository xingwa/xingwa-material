```
add app.manifest to vs project.

open the project *.csproj with notepad

2.1 add code:

  <PropertyGroup>
    <ApplicationManifest>app.manifest</ApplicationManifest>
  </PropertyGroup>
2.2 modify code:

    <None Include="app.manifest">
      <SubType>Designer</SubType>
    </None>

```
