# Windows 10 64bit C++,Pythonの開発環境構築

* IDE : Visual Studio Code
* C/C++ : Clang/LLVM
* C/C++ header & library : Windows10 SDK & MSVC Build
* python : pyenv & poetry
* SCM : Git for Windows
* etc : 7zip, sakura editor

2021/12/10 最新版

## 環境変数設定

```
setx IDEROOT C:\ide
setx VSCODE_HOME %IDEROOT%\VSCode
setx USRLOCAL %IDEROOT%\usr\local
setx DATAROOT C:\data
setx APPROOT %IDEROOT%\bin
setx PYENV_ROOT %USRLOCAL%\pyenv
setx PYENV %PYENV_ROOT%\pyenv-win
setx PYTHONVERSION 3.9.6
setx PYTHONPATH %PYENV%\versions\%PYTHONVERSION%
setx POETRY_HOME %USRLOCAL%\poetry

setx PYPATHES %PYENV%\bin;%PYENV%\shims;%PYTHONPATH%;%PYTHONPATH%\Scripts;%PYTHONPATH%\Tools\scripts;%POETRY_HOME%\bin
setx WKIT "C:\Program Files (x86)\Windows Kits\10"
setx MSVC "C:\Program Files (x86)\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.29.30037"

setx INCLUDE "%INCLUDE%;%IDEROOT%\include;%PYTHONPATH%\include;%IDEROOT%\usr\include;%IDEROOT%\usr\local\include;%MSVC%\ATLMFC\include;%MSVC%\include;%WKIT%\include\10.0.18362.0\ucrt;%WKIT%\include\10.0.18362.0\shared;%WKIT%\include\10.0.18362.0\um;%WKIT%\include\10.0.18362.0\winrt;%WKIT%\include\10.0.18362.0\cppwinrt"
setx LIBPATH "%LIBPATH%;%IDEROOT%\lib;%PYTHONPATH%\libs;%IDEROOT%\usr\lib;%IDEROOT%\usr\local\lib;%WKIT%\Lib"


exit
```
* PYTHONVERSIONは pyenvで指定可能なバージョン

```
setx Path %Path%;C:\Program Files\7-Zip;%APPROOT%;%IDEROOT%\mingw64\bin;%IDEROOT%\usr\bin;%PYPATHES%;%VSCODE_HOME%\bin
mkdir %IDEROOT%
mkdir %DATAROOT%
exit
```

## はじめに
### [7zip](https://sevenzip.osdn.jp/download.html)
インストーラー -> [v21.06](https://www.7-zip.org/a/7z2106-x64.exe)
### サクラエディタ(https://github.com/sakura-editor/sakura/releases)
インストーラー -> [v2.4.1](https://github.com/sakura-editor/sakura/releases/download/v2.4.1/sakura-tag-v2.4.1-build2849-ee8234f-Win32-Release-Installer.zip)

適当にインストール

## [Git for Windows](https://github.com/git-for-windows/git/releases)
[v2.34.1](https://github.com/git-for-windows/git/releases/download/v2.34.1.windows.1/Git-2.34.1-64-bit.tar.bz2)

```
powershell wget https://github.com/git-for-windows/git/releases/download/v2.34.1.windows.1/Git-2.34.1-64-bit.tar.bz2 -OutFile %TEMP%\git-for-windows.tar.bz2
7z x %TEMP%\Git-2.34.1-64-bit.tar.bz2 -so | 7z x -si -o%IDEROOT%
del /s /q %TEMP%\git-for-windows.tar.bz2
exit
```
* %IDEROOT%\binにインストールしてます

## 開発環境
## Python(pyenv運用)
#### [python](https://www.python.org/ftp/python/)
[インストール用にv3.9.9 embedを使う](https://www.python.org/ftp/python/3.9.9/python-3.9.9-embed-amd64.zip)

```
curl -sSL -o %TEMP%\py.zip https://www.python.org/ftp/python/3.9.9/python-3.9.9-embed-amd64.zip
7z x -o%TEMP%\pytmp %TEMP%\py.zip
set Path=%TEMP%\pytmp;%Path%
```

#### [pyenv](https://github.com/pyenv/pyenv.git)

```
curl -sSL https://bootstrap.pypa.io/get-pip.py | python - install pyenv-win --target %PYENV_ROOT%
cd %PYENV_ROOT%
rm -rf bin *distutil* install* pip* pkg_resources setuptools* wheel*
rd /s /q %TEMP%\pytmp

cd %PYENV_ROOT%
pyenv install %PYTHONVERSION%
pyenv global %PYTHONVERSION%
pyenv --version
chcp
grep -rl "chcp 1250" * | xargs sed -i "s/chcp 1250/chcp 65001/g"
pyenv rehash
chcp
pyenv update
pyenv install -l
```
* [grep & sed をする理由 -> #pyenv Issue51](https://github.com/pyenv-win/pyenv-win/issues/51)


```
curl -sSL -o %TEMP%\dev_d.msi https://www.python.org/ftp/python/%PYTHONVERSION%/amd64/dev_d.msi
msiexec /a %TEMP%\dev_d.msi targetdir="%PYTHONPATH%" /qn
del /s /q %TEMP%\dev_d.msi
python -m pip install --upgrade pip
mv %LOCALAPPDATA%\Microsoft\WindowsApps\python.exe %LOCALAPPDATA%\Microsoft\WindowsApps\_python.exe
mv %LOCALAPPDATA%\Microsoft\WindowsApps\python3.exe %LOCALAPPDATA%\Microsoft\WindowsApps\_python3.exe
```
* dev_d.msiをダウンロードしている理由 -> python39_d.libが欲しいため


#### [poetry](https://github.com/python-poetry/poetry)
```
cd %DATAROOT%
curl -sSL https://install.python-poetry.org | python -
poetry version
poetry self update
poetry config --list
poetry config virtualenvs.in-project true
poetry config cache-dir "%POETRY_HOME%\pypoetry\Cache"
```


#### [Microsoft BuildTools](https://visualstudio.microsoft.com/ja/visual-cpp-build-tools/)
[VS 2022](https://aka.ms/vs/17/release/vs_BuildTools.exe)
Download -> GUI Installer

##### Change Install Path
* %VISUAL_STUDIO_HOME%\MSBuildTools

* 個別のコンポーネント : select
 ** MSVC v142 xxxxx ビルドツール
 ** Windows 10 SDK

Download -> GUI Installer

##### Change Install Path

* Visual Studio IDE: %VISUAL_STUDIO_HOME%\tools

* ダウンロードキャッシュ: default(C:\ProgramData\Microsoft\VisualStudio\Packages)

* 共有コンポーネント、ツール、SDK: %VISUAL_STUDIO_HOME%\sdk

** Development Promptはパスの通ったところに配置する
```
bash
_VISUAL_STUDIO_HOME=$(echo "$VISUAL_STUDIO_HOME" | sed "s/\\\/\//g" | sed -r "s/(.):/\/\1/g")
cp -R "/C/Program Files (x86)/Windows Kits/10/include" $_VISUAL_STUDIO_HOME/
cp -R "/C/Program Files (x86)/Windows Kits/10/Lib/*/*/x64/*" $_VISUAL_STUDIO_HOME/lib/
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Auxiliary" $_VISUAL_STUDIO_HOME/
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Redist" $_VISUAL_STUDIO_HOME/
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/*/lib/x64/*" $_VISUAL_STUDIO_HOME/lib/
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/*/include/*" $_VISUAL_STUDIO_HOME/include/
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/*/crt" $_VISUAL_STUDIO_HOME
cp -R "/C/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/*/bin/Hostx64/x64" $_VISUAL_STUDIO_HOME/bin
exit

echo @call "%VISUAL_STUDIO_HOME%\tools\Common7\Tools\vcvarsall.bat" x64 %* > %IDEROOT%\bin\vcvars64.bat
echo @call "%VISUAL_STUDIO_HOME%\tools\Common7\Tools\vcvarsall.bat" x86 %* > %IDEROOT%\bin\vcvars32.bat

exit
```

#### [LLVM](https://github.com/llvm/llvm-project/releases)

GUI Installer [v13.0.0](https://github.com/llvm/llvm-project/releases/download/llvmorg-13.0.0/LLVM-13.0.0-win64.exe)
 -> Install Path : %IDEROOT%

#### [CMake](https://cmake.org/download/)
[v3.22.1 zip](https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.zip)
[v3.22.1 msi](https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.msi)

#### [Ninja](https://github.com/ninja-build/ninja/releases)
[v1.10.2](https://github.com/ninja-build/ninja/releases/download/v1.10.2/ninja-win.zip)

```
curl -sSL -o%TEMP%\ninja.zip https://github.com/ninja-build/ninja/releases/download/v1.10.2/ninja-win.zip
7z x -o%IDEROOT%\bin %TEMP%\ninja.zip
del /s /q %TEMP%\ninja.zip
```

##### Command line Install

```
curl -sSL -o %TEMP%\cmake.zip https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.zip
7z x -o%TEMP%\cmake %TEMP%\cmake.zip
bash -c "cp -fR $TEMP/cmake/cmake*/* $IDEROOT/"
del %TEMP%\cmake.zip
rd /s /q %TEMP%\cmake
```

#### [VSCode](https://code.visualstudio.com/)
[latest system installer](https://code.visualstudio.com/sha/download?build=stable&os=win32-x64)
[latest zip data](https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive)

##### Command line Install

```
curl -sSL -o "%TEMP%\download_vscode.zip" "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive"
7z x -aoa -o"%VSCODE_HOME%" "%TEMP%\download_vscode.zip"
del /s /q "%TEMP%\download_vscode.zip"
```

##### VSCode restore private setting

```
curl -sSL -o %TEMP%\vscode_extensions.txt https://raw.githubusercontent.com/kirin123kirin/.vscode/main/vscode_extensions.txt
for /f %n in (%TEMP%\vscode_extensions.txt) do (code --install-extension %n)
del /s /q %TEMP%\vscode_extensions.txt
curl -sSL -o %APPDATA%\Code\User\settings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/settings.json
curl -sSL -o %APPDATA%\Code\User\keybindings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/_keybindings.json
```
