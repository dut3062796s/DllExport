# [DllExport](https://github.com/3F/DllExport)

*Unmanaged Exports ( .NET DllExport )*

```
Copyright (c) 2009-2015  Robert Giesecke
Copyright (c) 2016-2017  Denis Kuzmin <entry.reg@gmail.com> :: github.com/3F
```

[![Build status](https://ci.appveyor.com/api/projects/status/yh1pnuhaqk8h334h/branch/master?svg=true)](https://ci.appveyor.com/project/3Fs/dllexport/branch/master)
[![Latest-Release](https://img.shields.io/github/release/3F/DllExport.svg)](https://github.com/3F/DllExport/releases/latest)
[![License](https://img.shields.io/badge/License-MIT-74A5C2.svg)](https://github.com/3F/DllExport/blob/master/LICENSE)
[![NuGet package](https://img.shields.io/nuget/v/DllExport.svg)](https://www.nuget.org/packages/DllExport/) 
[![coreclr_ILAsm](https://img.shields.io/badge/coreclr_ILAsm-v4.5.1-C8597A.svg)](https://www.nuget.org/packages/ILAsm/)
[![GetNuTool core](https://img.shields.io/badge/GetNuTool-v1.6.1-93C10B.svg)](https://github.com/3F/GetNuTool)
[![MvsSln](https://img.shields.io/badge/MvsSln-v2.0.0-865FC5.svg)](https://github.com/3F/MvsSln)
[![Conari](https://img.shields.io/badge/Conari-v1.3.0-8AA875.svg)](https://github.com/3F/Conari)


**Start with:**

[`DllExport`](https://3f.github.io/DllExport/releases/latest/manager/)` -action Configure` [[?](#how-to-get-dllexport)]


~~~---~~~

```csharp
[DllExport("Init", CallingConvention.Cdecl)]
public static int entrypoint(IntPtr L)
{
    // ... it will be called from Lua script

    lua_pushcclosure(L, onProc, 0);
    lua_setglobal(L, "onKeyDown");

    return 0;
}
```

* **For work with Unmanaged code/libraries (binding between .NET and C/C++ etc.), see [Conari](https://github.com/3F/Conari)**
* If you need convenient work with Lua (5.1, 5.2, 5.3, ...), see [LunaRoad](https://github.com/3F/LunaRoad)

```csharp
[DllExport("Init", CallingConvention.Cdecl)]
// __cdecl is the default calling convention for our library as and for C and C++ programs
[DllExport(CallingConvention.StdCall)]
[DllExport("MyFunc")]
[DllExport]
```

Support of Modules: Library (**.dll**) and Executable (**.exe**) [[?](https://github.com/3F/DllExport/issues/18)]


Where to look ? v1.2+ provides dynamic definitions of namespaces (ddNS feature), thus you can use what you want - details **[here](https://github.com/3F/DllExport/issues/2)**

```cpp
    Via Cecil or direct modification:

    Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

    000005B0                 00 C4 7B 01 00 00 00 2F 00 12 05       .�{..../...
    000005C0  00 00 02 00 00 00 00 00 00 00 00 00 00 00 26 00  ..............&.
    000005D0  20 02 00 00 00 00 00 00 00 49 2E 77 61 6E 74 2E   ........I.want.   <<<-
    000005E0  74 6F 2E 66 6C 79 00 00 00 00 00 00 00 00 00 00  to.fly..........  <<<-
```

[![](https://raw.githubusercontent.com/3F/DllExport/master/Resources/img/DllExport.png)](#)
[![](https://raw.githubusercontent.com/3F/DllExport/master/Resources/img/DllExport_ordinals.png)](https://github.com/3F/DllExport/issues/11#issuecomment-250907940)

Our Wizard and embeddable manager [[?](https://github.com/3F/DllExport/issues/38)]:

[![DllExport.bat](https://raw.githubusercontent.com/3F/DllExport/master/Resources/img/DllExport_manager.png)](https://3f.github.io/DllExport/releases/latest/manager/)

[![youtube.com/watch?v=okPThdWDZMM](https://raw.githubusercontent.com/3F/DllExport/master/Resources/img/DllExport_Wizard_overview_youtube.jpg)](https://youtu.be/okPThdWDZMM?t=46s)
[![PeViewer](https://raw.githubusercontent.com/3F/DllExport/master/Resources/img/DllExport_PeViewer.png)](https://github.com/3F/DllExport/issues/55)

----


[Initially](https://github.com/3F/DllExport/issues/3) the original tool `UnmanagedExports` was distributed by Robert Giesecke as an closed-source tool **under the [MIT License](https://opensource.org/licenses/mit-license.php)**:

* [Official page](https://sites.google.com/site/robertgiesecke/Home/uploads/unmanagedexports) - *posted Jul 9, 2009 [ updated Dec 19, 2012 ]*
* [Official NuGet Packages](https://www.nuget.org/packages/UnmanagedExports) 

Now, we will be more open ! all details [here](https://github.com/3F/DllExport/issues/3)

## License

It still under the [MIT License (MIT)](https://github.com/3F/DllExport/blob/master/LICENSE)

## &

### How does it work

Current features has been implemented through [ILDasm](https://github.com/3F/coreclr/tree/master/src/ildasm) & [ILAsm](https://github.com/3F/coreclr/tree/master/src/ilasm) that makes the all required steps via `.export` directive ([it's specific directive for ILAsm compiler only](https://github.com/3F/DllExport/issues/45#issuecomment-317802099)).

**What inside ? or how does work the .export directive ?**

Read about format PE32/PE32+, start with grammar from asmparse and move to writer:

```cpp
...
if(PASM->m_pCurMethod->m_dwExportOrdinal == 0xFFFFFFFF)
{
  PASM->m_pCurMethod->m_dwExportOrdinal = $3;
  PASM->m_pCurMethod->m_szExportAlias = $6;
  if(PASM->m_pCurMethod->m_wVTEntry == 0) PASM->m_pCurMethod->m_wVTEntry = 1;
  if(PASM->m_pCurMethod->m_wVTSlot  == 0) PASM->m_pCurMethod->m_wVTSlot = $3 + 0x8000;
}
...
EATEntry*   pEATE = new EATEntry;
pEATE->dwOrdinal = pMD->m_dwExportOrdinal;
pEATE->szAlias = pMD->m_szExportAlias ? pMD->m_szExportAlias : pMD->m_szName;
pEATE->dwStubRVA = EmitExportStub(pGlobalLabel->m_GlobalOffset+dwDelta);
m_EATList.PUSH(pEATE);
...
// logic of definition of records into EXPORT_DIRECTORY (see details from PE format)
HRESULT Assembler::CreateExportDirectory()  
{
...
    IMAGE_EXPORT_DIRECTORY  exportDirIDD;
    DWORD                   exportDirDataSize;
    BYTE                   *exportDirData;
    EATEntry               *pEATE;
    unsigned                i, L, ordBase = 0xFFFFFFFF, Ldllname;
    ...
    ~ now we're ready to miracles ~
```

Read also my explanations from here: [about mscoree](https://github.com/3F/DllExport/issues/45#issuecomment-317802099); [DllMain & the export-table](https://github.com/3F/DllExport/issues/5#issuecomment-240697109); [DllExport.dll](https://github.com/3F/DllExport/issues/28#issuecomment-281957212); [.exp & .lib](https://github.com/3F/DllExport/issues/9#issuecomment-246189220); [ordinals](https://github.com/3F/DllExport/issues/8#issuecomment-245228065) ...

### How to get DllExport

**v1.6+** have no official support of any standard NuGet clients. [[?](https://github.com/3F/DllExport/issues/38)] 

* [New Wizard and embeddable manager](https://youtu.be/okPThdWDZMM?t=55s)

Use [DllExport.bat](https://3F.github.io/DllExport/releases/latest/manager/) (~18 Kb without powershell scripts and dotnet-cli) from any place. For example, you can still get it from packages via NuGet server ([how to](https://youtu.be/okPThdWDZMM?t=1m1s)) or it also can be embedded inside any other your scripts because of simple batch script.

**Please note**: You do not need to call manually DllExport.bat after initial configuration. It still will be **automatically** for ~`-action Restore` command etc.


* To install/uninstall or to reconfigure your projects:

```
DllExport -action Configure
```

* To upgrade already used version:

```
DllExport -action Update
```

* To manually restore package (**It should be automatically** restored by any Build operation of your configured projects or when you open Visual Studio. But if you need, use this):
```
DllExport -action Restore
```

* All available features:

```
DllExport -h
```

Other variants:

* `gnt /p:ngpackages="DllExport"` [[?](https://github.com/3F/GetNuTool)]
    * [GetNuTool](https://github.com/3F/GetNuTool): `msbuild gnt.core /p:ngpackages="DllExport"` or [`gnt`](https://3F.github.io/GetNuTool/releases/latest/gnt/)` /p:ngpackages="DllExport"`
* (deprecated) NuGet PM: `Install-Package DllExport`
* (deprecated) NuGet Commandline: `nuget install DllExport`
* [/releases](https://github.com/3F/DllExport/releases) [ [latest](https://github.com/3F/DllExport/releases/latest) ]
* [Nightly builds](https://ci.appveyor.com/project/3Fs/dllexport/history) (`/artifacts` page). But remember: It can be unstable or not work at all. Use this for tests of latest changes.

### How to Build

Use build.bat if you need final binaries, or NuGet package as `DllExport.<version>.nupkg` and other.

```bash
> build
```

Part of this build scripts works via vssbe ([?](https://github.com/3F/DllExport/issues/31#issuecomment-294231378)) and for build via console (including CI etc.) uses [CIM](https://www.nuget.org/packages/vsSBE.CI.MSBuild/) version of this. So you do not need anything else, just type `build`.

For Visual Studio, use this [vsix version for IDE](https://visualstudiogallery.msdn.microsoft.com/0d1dbfd7-ed8a-40af-ae39-281bfeca2334/)

### How to Debug

For example, find the DllExport.MSBuild project in solution:

* `Properties` > `Debug`:
    * `Start Action`: set as `Start External program`
        * Add full path to **msbuild.exe**, for example: C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe
    * `Start Options` > `Command line arguments` write for example:

```bash
"<path_to_SolutionFile_for_debugging>.sln" /t:Build /p:Configuration=<Configuration>
```

use additional `Diagnostic` key to msbuild if you need details from .targets
```bash
"<path_to_SolutionFile_for_debugging>.sln" /verbosity:Diagnostic /t:Rebuild /p:Configuration=<Configuration>
```

Go to `Start Debugging`. Now you can debug at runtime.

### coreclr - ILAsm / ILDasm

We use **our custom versions of coreclr**, special for DllExport project - https://github.com/3F/coreclr

This helps to avoid some problems ([like this](https://github.com/3F/DllExport/issues/17)) and more...

*To build minimal version (means that it does not include all components as for original coreclr repo):*

* Restore git submodule or use repo: https://github.com/3F/coreclr.git

```bash
git submodule update --init --recursive
```

*Make sure that you have installed [CMake](https://cmake.org/download/), then build simply:*

```bash
build_s all x86 x64 Release
build_s x86 Release
```

or use
```bash
build_coreclr_x86.cmd
build_coreclr_x86_x64.cmd
```

*You can also use our binaries of coreclr separately if needed:*

* [![NuGet package](https://img.shields.io/nuget/v/ILAsm.svg)](https://www.nuget.org/packages/ILAsm/)
* Look also [here](https://github.com/3F/coreclr/issues/1)


### Donation

Please note again, the initial [UnmanagedExports](https://www.nuget.org/packages/UnmanagedExports) was created by Robert Giesecke. You should [visit its page](https://sites.google.com/site/robertgiesecke/Home/uploads/unmanagedexports) if you need.

But this repository does not related with Robert and generally maintained by [github.com/3F](https://github.com/3F) developer (Follow: [[GitHub](https://github.com/3F)]; [[G+](https://plus.google.com/+DenisKuzmin3F)]). **So** if you think that our improvements, fixes, other changes, support, information, I don't know... if something are helpful for you from this, donations are welcome, and thanks !


[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=entry%2ereg%40gmail%2ecom&lc=US&item_name=3F%2dOpenSource%20%5b%20github%2ecom%2f3F&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donate_SM%2egif%3aNonHosted) ( github.com/3F )

