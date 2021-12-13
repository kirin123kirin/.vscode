# Windows 10 64bit Python及びC++開発環境構築を構築する

* IDE : Visual Studio Code
   * cpp extention
   * python extention
   * markdown extention
   * Git Lense
   * Histroy
   * etc..
* C/C++ : Clang/LLVM
* C/C++ header & library : 最低限のWindows10 SDK & MSVC Build
* python : pyenv & poetry
* SCM : Git for Windows
* etc : nodejs, 7zip, sakura editor

以降のコマンドライン手順はWindows標準のコマンドプロンプトへのインプット内容

## 1. 環境変数設定

### (1) ユーザ環境変数等
* IDEROOTはインストール先ディレクトリパス
* PYTHONVERSIONは pyenvで指定可能なバージョン
```powershell
setx IDEROOT C:\ide
setx PYTHONVERSION 3.9.6

setx VSCODE_HOME %IDEROOT%\VSCode
setx USRLOCAL %IDEROOT%\usr\local
setx PYENV_ROOT %USRLOCAL%\pyenv
setx PYENV %PYENV_ROOT%\pyenv-win
setx PYTHONPATH %PYENV%\versions\%PYTHONVERSION%
setx POETRY_HOME %USRLOCAL%\poetry
setx NODEJS_HOME %USRLOCAL%\nodejs

exit

```
## <span style="color: red; ">注意：IDEROOT</span>に既存のディレクトリを指定したら
同名のファイル又はディレクトリがある場合には、上書き良否確認などは一切せず
<span style="color: red; ">強制的に上書き</span>するので
<span style="color: red; ">新規ディレクトリの指定</span>を強く推奨する。


### (2) Path追加
[ローカル変数"GOMI"とは？](https://zenn.dev/ef/articles/fede252753800b12f42b)
```
set GOMI=%LOCALAPPDATA%\Microsoft\WindowsApps

setx Path "%Path:%GOMI%;=%;C:\Program Files\7-Zip;C:\Program Files (x86)\sakura;%PYENV%\bin;%PYENV%\shims;%PYTHONPATH%;%PYTHONPATH%\Scripts;%PYTHONPATH%\Tools\scripts;%POETRY_HOME%\bin;%IDEROOT%\bin;%IDEROOT%\cmd;%IDEROOT%\mingw64\bin;%IDEROOT%\usr\bin;%VSCODE_HOME%\bin;%NODEJS_HOME%;%APPDATA%\npm;%GOMI%"

exit

```


## 2. とりあえず入れるもの
デフォルト設定でインストール
### (1) [7zip](https://sevenzip.osdn.jp/download.html)
これ必須 -> [v21.06 直リンク](https://www.7-zip.org/a/7z2106-x64.exe)
### (2) [サクラエディタ](https://github.com/sakura-editor/sakura/releases)
なんでもよいけど -> [v2.4.1 直リンク](https://github.com/sakura-editor/sakura/releases/download/v2.4.1/sakura-tag-v2.4.1-build2849-ee8234f-Win32-Release-Installer.zip)

### (3) [Git for Windows](https://github.com/git-for-windows/git/releases)
最新版をインストール

```powershell
mkdir %IDEROOT%\usr\bin
powershell Invoke-WebRequest -UseBasicParsing -Uri https://github.com/webfolderio/wget-windows/releases/download/1.21.2/wget-1.21.2-64bit-GnuTLS.zip -OutFile %TEMP%\wget.zip
7z x -o%IDEROOT%\usr\bin %TEMP%\wget.zip
wget.exe -O %TEMP%\git-for-windows.tar.bz2 https://github.com/git-for-windows/git/releases/download/v2.34.1.windows.1/Git-2.34.1-64-bit.tar.bz2
echo %IDEROOT%にインストールしてます
7z x %TEMP%\git-for-windows.tar.bz2 -so | 7z x -si -ttar -o%IDEROOT% -aoa
del /s /q %TEMP%\git-for-windows.tar.bz2 %TEMP%\wget.zip

```
* このエラーは気にしない
  * 「ERROR: Cannot create symbolic link : クライアントは要求された特権を保有していません。 : fd, stderr, stdin, stdout, mtab」

## 3. 個人的に外せない開発環境
### (0) 下準備

1. ダウンロード＆解凍用コマンドの作成(この後ダウンロードしまくるので)

```
bash
cat <<EOF > $IDEROOT/cmd/dunzip.cmd
@echo off
set argc=0
for %%a in ( %* ) do set /a argc+=1

if %argc% lss 2 (
  echo This is download and extract
  echo Usage: dunzip.cmd ^<extract target directory^> ^<download URL^> ^<extract wildcard rule^>
  set argc=
  exit /b 1
)

set TMPNAME="%TEMP%"/workdir_download_will_unzip

rm -rf %TMPNAME%*
curl -L -o %TMPNAME%.zip %2

7z x -o%TMPNAME% %TMPNAME%.zip

if not exist %1 (mkdir %1)

if "%3"=="" (
  mv -f %TMPNAME%/* %TMPNAME%/.* %1 2>&1 | grep -v "/.. to a subdirectory of itself" | grep -v ""
) else (
  mv -f %TMPNAME%/%3 %1
)

rm -rf %TMPNAME%*

EOF
unix2dos $IDEROOT/cmd/dunzip.cmd

exit

```

2. [fzf](https://github.com/junegunn/fzf#windows)をインストール
3. [RipGrep](https://github.com/BurntSushi/ripgrep)をインストール
4. [RipGrep-all](https://github.com/phiresky/ripgrep-all)をインストール
5. [GNU版 grepに変更](http://gnuwin32.sourceforge.net/packages/grep.htm)をインストール

```

dunzip %IDEROOT%/usr/bin https://github.com/junegunn/fzf/releases/download/0.28.0/fzf-0.28.0-windows_amd64.zip

dunzip %IDEROOT%/usr/bin https://github.com/BurntSushi/ripgrep/releases/download/13.0.0/ripgrep-13.0.0-x86_64-pc-windows-msvc.zip ripgrep-*/rg.exe

dunzip %IDEROOT%/usr/bin https://github.com/phiresky/ripgrep-all/releases/download/v0.9.6/ripgrep_all-v0.9.6-x86_64-pc-windows-msvc.zip ripgrep*/rga*.exe

dunzip %IDEROOT%/usr/bin http://downloads.sourceforge.net/gnuwin32/grep-2.5.4-bin.zip

```

### (2) [Python(pyenv-win)](https://github.com/pyenv-win/pyenv-win)

```powershell
git clone https://github.com/pyenv-win/pyenv-win.git "%PYENV_ROOT%"
cd "%PYENV_ROOT%"
mv .version pyenv-win
rm -rf [._a-oq-zR]*
mv pyenv-win/.version ./
pyenv --version
pyenv rehash

pyenv install %PYTHONVERSION%
pyenv global %PYTHONVERSION%
pyenv update
pyenv install -l | grep -v "win"

echo python %PYTHONVERSION% の他に必要なバージョンがあれば、ここで入れてください。上に出ているバージョン一覧参照

```

### (3) Python(pyenv-win) 初期設定
やってること

1. python関連 SJIS固有の不具合の強引な対策
  * [pyenv Issue51対処](https://github.com/pyenv-win/pyenv-win/issues/51)
  * [python zipファイル名日本語文字化け対策](https://oku.edu.mie-u.ac.jp/~okumura/python/encoding.html)

2. ライブラリインストール
  * pip最新化
  * 開発用ライブラリのインストール
  * CAPI開発用にpythonXX_d.libのインストール

```powershell

echo pyenv Issue51関連の不具合修正してます
grep -rl "chcp 1250" * | xargs sed -i "s/chcp 1250/chcp 932/g"
pyenv rehash

cd %PYENV%\versions

for /d %d in (*) do (
  pyenv local %d
  
  echo zipfile 文字化け不具合の修正をしてます
  sed -i.bak "s/cp437/cp932/g" %PYENV%\versions\%d\Lib\zipfile.py
  
  echo pipを最新化します
  python -m pip install --upgrade pip
  
  echo 開発用ツールをインストールしてます
  pip install autopep8 scikit-build
  pip install -r https://github.com/scikit-build/scikit-build/raw/master/requirements-dev.txt

  echo debug用ライブラリをダウンロードしてます
  curl -L -o %TEMP%\dev_d_%d.msi https://www.python.org/ftp/python/%d/amd64/dev_d.msi
  sleep 1
  msiexec /a %TEMP%\dev_d_%d.msi targetdir="%PYENV%\versions\%d" /qn
  del /s /q %TEMP%\dev_d_%d.msi
)

pyenv local %PYTHONVERSION%

```

### (3) TODO

### (4) [poetry](https://github.com/python-poetry/poetry)
```powershell
curl -L https://install.python-poetry.org | python -
poetry --version
poetry self update
poetry config --list
poetry config virtualenvs.in-project true
poetry config cache-dir "%POETRY_HOME%\pypoetry\Cache"

```

### (5) C/C++ Build Tool
#### [LLVM](https://github.com/llvm/llvm-project/releases) & [CMake](https://cmake.org/download/) & [Ninja](https://github.com/ninja-build/ninja/releases)

```powershell
echo LLVM インストール...
dunzip %IDEROOT% https://github.com/llvm/llvm-project/releases/download/llvmorg-13.0.0/LLVM-13.0.0-win64.exe
rd /s /q %IDEROOT%\\$PLUGINSDIR

echo CMake インストール...
dunzip %IDEROOT% https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.zip cmake*/*

echo Ninja インストール...
dunzip %IDEROOT%\bin https://github.com/ninja-build/ninja/releases/download/v1.10.2/ninja-win.zip

```

### (6) C/C++ Windows headers & libraries
#### [Microsoft BuildTools](https://visualstudio.microsoft.com/ja/visual-cpp-build-tools/)
インストーラ → [VSBuildTools(2022) 直リンク](https://aka.ms/vs/17/release/vs_BuildTools.exe)

* 個別のコンポーネント : 検索窓で以下の２つを最低限選択する
 * MSVC x64 ビルド ツール 最新
 * Windows 10 SDK

### (7) [VSCode](https://code.visualstudio.com/)

```powershell
echo VSCodeインストールしてます
dunzip "%VSCODE_HOME%" "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive"
curl -L -o "%TEMP%\download_vscode.zip" "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive"
7z x -aoa -o"%VSCODE_HOME%" "%TEMP%\download_vscode.zip"
del /s /q "%TEMP%\download_vscode.zip"

echo ショートカットを作成してます
set TARGET='%VSCODE_HOME%\Code.exe'
set SHORTCUT='%APPDATA%\Microsoft\Windows\Start Menu\Code.lnk'
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -Command "$ws = New-Object -ComObject WScript.Shell; $s = $ws.CreateShortcut(%SHORTCUT%); $S.TargetPath = %TARGET%; $S.Save()"
cp %SHORTCUT% %USERPROFILE%\Desktop

```

### (8) [Node.js](https://nodejs.org/ja/)
インストーラ -> [v17.2.0 直リンク](https://nodejs.org/dist/v17.2.0/node-v17.2.0-x64.msi)


```
dunzip "%NODEJS_HOME%" https://nodejs.org/dist/v16.13.1/node-v16.13.1-win-x64.zip node-v*/*

```

## 4. 個人的な初期設定
### (1) VSCode

```powershell
echo 拡張機能をインストールします
bash -c 'curl -sSL https://raw.githubusercontent.com/kirin123kirin/.vscode/main/vscode_extensions.txt | xargs -L1 code --install-extension'

echo 全般設定の設定中
curl -L -o %APPDATA%\Code\User\settings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/settings.json

echo キーバインドの設定中
curl -L -o %APPDATA%\Code\User\keybindings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/_keybindings.json

code
mshta vbscript:execute("MsgBox(""ステータスバーのダウンロードが完了するまで待ってVSCodeを再起動してください""):close")

```

### (2) Git config global & PyPI config
ユーザ名とEmailを入力してください。

```bash
echo git configとpypircの設定を行います。

bash

cat<<EOF > ~/.gitconfig
[user]
	       email = 
	       name = 
[filter "lfs"]
	       clean = git-lfs clean -- %f
	       smudge = git-lfs smudge -- %f
	       process = git-lfs filter-process
	       required = true
[gui]
	       encoding = utf-8
[core]
        autocrlf = input
EOF

notepad ~/.gitconfig

cat<<EOF > ~/.pypirc
[distutils]
index-servers=
    pypi
    pypitest

[pypi]
username: 
password: 

#[pypitest]
#repository: https://test.pypi.org/legacy/
#username : 
#password : 

EOF

notepad ~/.pypirc

exit

```

---
以下は別にやらなくても良い。
### INCLUDE, LIBPATHの環境変数を作る
Visual Studioのパスが死ぬほどめんどくさい

```bash
bash

cat <<EOF > $IDEROOT/cmd/evalvar.cmd
@echo off
set argc=0
for %%a in ( %* ) do set /a argc+=1

if %argc% lss 2 (
  echo This is a command to save the result of execution to a variable
  echo Usage: regval.cmd ^<return variable^> ^<run command^>
  set argc=
  exit /b 1
)

set argc=

for /f "tokens=*" %%a in ('%~2 ^| tr -d "\r" ^| tr "\n" " "') do set %1=%%a

EOF
unix2dos $IDEROOT/cmd/evalvar.cmd


cat <<EOF > $IDEROOT/cmd/regvalue.cmd
@echo off
set argc=0
for %%a in ( %* ) do set /a argc+=1

if %argc% neq 3 (
  echo This is the command to get the value of a registry key
  echo Usage: regval.cmd ^<return variable^> ^<registry key path^> ^<key name^>
  set argc=
  exit /b 1
)

set argc=

for /f "tokens=1,2,*" %%a in ('REG QUERY %2 /v %3') DO if not %%c=="" set %1=%%c

EOF
unix2dos $IDEROOT/cmd/regvalue.cmd


cat <<EOF > $IDEROOT/cmd/toslash.cmd
@echo off
set argc=0
for %%a in ( %* ) do set /a argc+=1

if %argc% neq 2 (
  echo This is a command to convert a Windows backslash path to a slash.
  echo Usage: regval.cmd ^<return variable^> ^<Windows Path of backslash^>
  set argc=
  exit /b 1
)

set argc=

for /f "tokens=*" %%a in ('echo %2 ^| sed "s/\\\\\\/\\//g"') do set %1=%%a

EOF
unix2dos $IDEROOT/cmd/toslash.cmd

exit


set ARCH=x64
regvalue WKIT "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Kits\Installed Roots" "KitsRoot10"
regvalue MSVC_ROOT "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\Setup" "SharedInstallationPath"
set MSVC_ROOT=%MSVC_ROOT:\Shared=%

evalvar PROCESSOR_ARCH "echo %PROCESSOR_ARCHITECTURE% ^| tr '[A-Z]' '[a-z]'"
toslash IDEROOT_S %IDEROOT%
toslash WKIT_S %WKIT%
toslash MSVC_ROOT_S %MSVC_ROOT%

evalvar WKIT_LATEST_REVISION "ls ""%WKIT_S%/Lib"" ^| tail -1"
evalvar VS_LATEST_VERSION "ls ""%MSVC_ROOT%"" ^| tail -1"
evalvar MSVC_LATEST_VERSION "ls ""%MSVC_ROOT%\VC\Tools\MSVC"" ^| tail -1"
evalvar CLANG_VERSION "ls ""%IDEROOT%/lib/clang"" ^| tail -1"

set MSBUILD_ROOT_S "%MSVC_ROOT_S%/%VS_LATEST_VERSION%/BuildTools"
set MSBUILD_S "%MSVC_ROOT_S%/VC/Tools/MSVC/%MSVC_LATEST_VERSION%"

set WKIT_LIBPATH_S=%WKIT_S%/Lib/%WKIT_LATEST_REVISION%
set WKIT_INCLUDE_S=%WKIT_S%/Lib/%WKIT_LATEST_REVISION%

set LIBPATH_S=%IDEROOT_S%/usr/local/lib
set LIBPATH_S=%LIBPATH_S%;%IDEROOT_S%/usr/local/pyenv/pyenv-win/versions/%PYTHONVERSION%/libs
set LIBPATH_S=%LIBPATH_S%;%WKIT_LIBPATH_S%/ucrt/%ARCH%
set LIBPATH_S=%LIBPATH_S%;%WKIT_LIBPATH_S%/ucrt_enclave/%ARCH%
set LIBPATH_S=%LIBPATH_S%;%WKIT_LIBPATH_S%/um/%ARCH%
set LIBPATH_S=%LIBPATH_S%;%MSBUILD_ROOT_S%/DIA SDK/lib/%PROCESSOR_ARCH%
set LIBPATH_S=%LIBPATH_S%;%MSBUILD_S%/lib/%ARCH%
set LIBPATH_S=%LIBPATH_S%;%IDEROOT_S%/lib/clang/%CLANG_VERSION%/lib
set LIBPATH_S=%LIBPATH_S%;%IDEROOT_S%/usr/local/pyenv/pyenv-win/libexec/libs
set LIBPATH_S=%LIBPATH_S%;%IDEROOT_S%/usr/lib
set LIBPATH_S=%LIBPATH_S%;%IDEROOT_S%/lib

set INCLUDE_S=%IDEROOT_S%/usr/local/include
set INCLUDE_S=%INCLUDE_S%;%IDEROOT_S%/usr/local/pyenv/pyenv-win/versions/%PYTHONVERSION%/include
set INCLUDE_S=%INCLUDE_S%;%WKIT_INCLUDE_S%/cppwinrt
set INCLUDE_S=%INCLUDE_S%;%WKIT_INCLUDE_S%/shared
set INCLUDE_S=%INCLUDE_S%;%WKIT_INCLUDE_S%/ucrt
set INCLUDE_S=%INCLUDE_S%;%WKIT_INCLUDE_S%/um
set INCLUDE_S=%INCLUDE_S%;%WKIT_INCLUDE_S%/winrt
set INCLUDE_S=%INCLUDE_S%;%MSBUILD_ROOT_S%/DIA SDK/include
set INCLUDE_S=%INCLUDE_S%;%MSBUILD_ROOT_S%/VC/Auxiliary/VS/include
set INCLUDE_S=%INCLUDE_S%;%MSBUILD_S%/include
set INCLUDE_S=%INCLUDE_S%;%IDEROOT_S%/usr/local/poetry/venv/Include
set INCLUDE_S=%INCLUDE_S%;%IDEROOT_S%/lib/clang/%CLANG_VERSION%/include
set INCLUDE_S=%INCLUDE_S%;%IDEROOT_S%/include


setx INCLUDE %INCLUDE_S%
setx LIBPATH %LIBPATH_S%

echo @call ^"^%MSBUILD_ROOT^%\Common7\Tools\VsDevCmd.bat^" %* > %IDEROOT%\bin\vsdevcmd.bat
echo @call ^"^%MSBUILD_ROOT^%\VC\Auxiliary\Build\vcvarsall.bat^" %* > %IDEROOT%\bin\vcvarsall.bat
echo @call ^"^%MSBUILD_ROOT^%\VC\Auxiliary\Build\vcvarsall.bat^" x64 %* > %IDEROOT%\bin\vcvars64.bat
echo @call ^"^%MSBUILD_ROOT^%\VC\Auxiliary\Build\vcvarsall.bat^" x86 %* > %IDEROOT%\bin\vcvars32.bat

```
以上、
