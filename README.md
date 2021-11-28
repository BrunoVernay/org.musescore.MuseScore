# MuseScore Flatpak

This is the [MuseScore](https://musescore.org/) [Flatpak](https://flatpak.org/).

(`Readme` inspired by https://github.com/flathub/org.thonny.Thonny)


## Users

This document is intended for developers, packagers, testers ...

If you just want to use MuseScore, go to https://FlatHub.org/apps/details/org.musescore.MuseScore and follow the instructions.


### Command-line arguments

You may pass command line arguments, either:
- Via the command-line `flatpak run org.musescore.MuseScore --no-webview --use-audio jack `
- Via the Desktop file, making a modified copy in a folder that has precedence over the Flatpak one (See: https://github.com/flatpak/flatpak/issues/2913  and https://docs.flatpak.org/en/latest/conventions.html#desktop-files)

To use Desktop file:
```
# Note: If not installed as `--user` the file will be in `/var/lib/flatpak/...` instead of `~/.local/..`)
cp ~/.local/share/flatpak/exports/share/applications/org.musescore.MuseScore.desktop ~/.local/share/applications/

# Make your changes in your copy:
vim ~/.local/share/applications/org.musescore.MuseScore.desktop
```

Note that these changes will be permanent! And if the Flatpak maintainer modify the Desktop file, you will not get these changes. You will have to do the merge manually.


### Parallel install

TODO: How to install multiple version in parallel...


## Maintainers

Get the source from the official repo: https://github.com/flathub/org.musescore.MuseScore.

`git clone git@github.com:flathub/org.musescore.MuseScore.git`


### Dependencies

You will need to install the KDE (Qt) SDK for flatpak: `flatpak install org.kde.Sdk/x86_64/5.15-21.08` (Version as of 2021-11-27, check with the `.yaml` file to be sure!)
```
# For info https://invent.kde.org/packaging/flatpak-kde-runtime/-/branches/active
flatpak search  kde.sdk
```


During MuseScore build, you may see messages like 
```
celt ... not found
db   ... not found
```
This is because MuseScore is also building libraries like Jack2. But these are optional dependencies and Jack2 library will not be used anyway, only its interface.


### Build

`flatpak-builder --install --user --force-clean appdir/ org.musescore.MuseScore.yaml`

Note: You could use other folder than `appdir/` but since it is already in the `.gitignore` it is convenient.

It will also create a `.flatpak-builder/`.


TODO: Elaborate on following:
`--keep-build-dirs`
`flatpak-builder --build --user --force-clean appdir/ org.musescore.MuseScore.yaml` 


#### Optimizations

The build takes 10 to 15min on a "Core i7". 
I tried to set `-DCMAKE_BUILD_PARALLEL_LEVEL=8` (See https://cmake.org/cmake/help/v3.12/envvar/CMAKE_BUILD_PARALLEL_LEVEL.html) but it did not make any difference.
Variations on `CMAKE_BUILD_TYPE` and `MUSESCORE_BUILD_CONFIG` also gave less than a minute optimization.
Typicall times: `83.83s user 10.32s system 11% cpu 14:01.74 total`

=> I did not find a way to improve the build time significantly. (Apart from using the cache obviously.)


#### Details

The build creates a cache in `.flatpak-builder/`. (contains downloaded resources, build files, ...)

It should produce a `appdir/files/lib/debug/` folder.
TODO: check that only debug builds creates it.


#### Cleanup

I like to use `rm -rf appdir .flatpak-builder/{b*,c*,r*}`
It keeps only the `downloads/`. It allows to measure the build time without the downloads.


### Run

To run in the sandbox:
`flatpak-builder --run appdir/ org.musescore.MuseScore.yaml mscore`
(TODO: understand what happen here. It is running in Sandbox but without permissions?)


### Debug

To run in a debuger: 
1. `flatpak run --devel --command=sh org.musescore.MuseScore`
    - That will give you a prompt in the sandbox where you can run gdb:
2. `gdb --args mscore --debug`
    - That will give you a prompt in gdb
    - `--debug`  is a MuseScore option enabling logs and debug menus
    - or you may simply run `gdb mscore`
3. `run` will run MuseScore
    - in case of crash, use `where` to get information about the crash
4. `quit` will quit gdb (or Ctrl-d)
5. `exit` will exit the Flatpak sandbox (or Ctrl-d)



Troubleshooting: 
- gdb gives this message `(No debugging symbols found in mscore)`
  - Trying variations on `CMAKE_BUILD_TYPE` and `MUSESCORE_BUILD_CONFIG`
    - Building with `RelWihDebInfo` and `release` 
    - Building with `RelWihDebInfo` and `dev`  
        - still no debugging symbols in gdb
        - more or less 10s total
    - Building with `Debug` and `dev` 
        - makes `mscore` 32.4 Mo instead of 31.9. `/lib/debug` contains the same 210Mo `mscore.debug`
        - still no debugging symbols in gdb
    - In all cases, there is a strange `lib/debug/lib/debug/lib/debug/lib/debug/` structure with files named `...debug.debug.debug.debug`. They relates to `Jack` and `Alsa`



#### Fonts

Fonts can cause troubles. FontConfig has an environment variable `FC_DEBUG` that allow to output log messages: https://www.freedesktop.org/software/fontconfig/fontconfig-user.html#DEBUG
You can uncomment it in `org.musescore.MuseScore.yaml`. 

You do not need to use the debugger to see the messages. You only need to start MuseScore from the command-line.


### Edit source code

Instead of using the Git repo as a Source, you can specify a "Directory sources": https://docs.flatpak.org/en/latest/flatpak-builder-command-reference.html#idm46203909541056
TODO: test!!

In VSCode, it can be useful to add `.flatpak*/**` and `appdir*/**` to the `files.watcherExclude`. See https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc 


### Patch

TODO: Test!!


### References

- Flatpak Wiki https://github.com/flatpak/flatpak/wiki/Tips-&-Tricks
- MuseScore command-line options: https://musescore.org/en/handbook/3/command-line-options
- MuseScore CMakeList https://github.com/musescore/MuseScore/blob/master/CMakeLists.txt
- FontConfig https://www.freedesktop.org/software/fontconfig/fontconfig-user.html#DEBUG
