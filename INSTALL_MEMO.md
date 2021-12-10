# Windows 10 64bit C++,Pythonの開発環境構築

* IDE : Visual Studio Code
   * cpp extention
   * python extention
   * markdown extention
   * Git Lense
   * Histroy
   * etc..
* C/C++ : Clang/LLVM
* C/C++ header & library : Windows10 SDK & MSVC Build
* python : pyenv & poetry
* SCM : Git for Windows
* etc : 7zip, sakura editor

2021/12/10 最新版

## 環境変数設定

```powershell
setx IDEROOT C:\ide
setx VSCODE_HOME %IDEROOT%\VSCode
setx USRLOCAL %IDEROOT%\usr\local
setx DATAROOT C:\data
setx PYENV_ROOT %USRLOCAL%\pyenv
setx PYENV %PYENV_ROOT%\pyenv-win
setx PYTHONVERSION 3.9.6
setx PYTHONPATH %PYENV%\versions\%PYTHONVERSION%
setx POETRY_HOME %USRLOCAL%\poetry

setx PYPATHES %PYENV%\bin;%PYENV%\shims;%PYTHONPATH%;%PYTHONPATH%\Scripts;%PYTHONPATH%\Tools\scripts;%POETRY_HOME%\bin
setx WKIT "C:\Program Files (x86)\Windows Kits\10"
setx MSVC_ROOT "C:\Program Files (x86)\Microsoft Visual Studio\2022"
setx MSVC "%MSVC_ROOT%\BuildTools\VC\Tools\MSVC\14.29.30705"

setx INCLUDE "%IDEROOT%\include;%PYTHONPATH%\include;%IDEROOT%\usr\include;%IDEROOT%\usr\local\include;%MSVC%\ATLMFC\include;%MSVC%\include;%WKIT%\include\10.0.18362.0\ucrt;%WKIT%\include\10.0.18362.0\shared;%WKIT%\include\10.0.18362.0\um;%WKIT%\include\10.0.18362.0\winrt;%WKIT%\include\10.0.18362.0\cppwinrt"
setx LIBPATH "%IDEROOT%\lib;%PYTHONPATH%\libs;%IDEROOT%\usr\lib;%IDEROOT%\usr\local\lib;%WKIT%\Lib"
exit

```
* PYTHONVERSIONは pyenvで指定可能なバージョン

```powershell
setx Path "%Path%;C:\Program Files\7-Zip;C:\Program Files (x86)\sakura;%IDEROOT%\bin;%IDEROOT%\cmd;%IDEROOT%\mingw64\bin;%IDEROOT%\usr\bin;%PYPATHES%;%VSCODE_HOME%\bin"
mkdir %IDEROOT%
mkdir %DATAROOT%
exit

```

## とりあえず入れるもの
### [7zip](https://sevenzip.osdn.jp/download.html)
インストーラー -> [v21.06 直リンク](https://www.7-zip.org/a/7z2106-x64.exe)
### [サクラエディタ](https://github.com/sakura-editor/sakura/releases)
インストーラー -> [v2.4.1 直リンク](https://github.com/sakura-editor/sakura/releases/download/v2.4.1/sakura-tag-v2.4.1-build2849-ee8234f-Win32-Release-Installer.zip)

適当にインストール

## [Git for Windows](https://github.com/git-for-windows/git/releases)
現時点での最新版のこれを使う
https://github.com/git-for-windows/git/releases/download/v2.34.1.windows.1/Git-2.34.1-64-bit.tar.bz2

```powershell
powershell wget https://github.com/git-for-windows/git/releases/download/v2.34.1.windows.1/Git-2.34.1-64-bit.tar.bz2 -OutFile %TEMP%\git-for-windows.tar.bz2
echo %IDEROOT%にインストールしてます
7z x %TEMP%\git-for-windows.tar.bz2 -so | 7z x -si -ttar -o%IDEROOT%
del /s /q %TEMP%\git-for-windows.tar.bz2

```

* このエラーは出てもよい
  * 「ERROR: Cannot create symbolic link : クライアントは要求された特権を保有していません。 : fd, stderr, stdin, stdout, mtab」

## 開発環境
## Python(pyenv運用)
#### [python](https://www.python.org/ftp/python/)
 pyenv インストールまでの一時的な環境なのでembedを使う。pyenvインストール後はすぐ消してしまう
 https://www.python.org/ftp/python/3.9.9/python-3.9.9-embed-amd64.zip

```powershell
curl -L -o %TEMP%\py.zip https://www.python.org/ftp/python/3.9.9/python-3.9.9-embed-amd64.zip
7z x -o%TEMP%\pytmp %TEMP%\py.zip
set Path=%TEMP%\pytmp;%Path%

```

#### [pyenv](https://github.com/pyenv/pyenv.git)

```powershell
curl -L https://bootstrap.pypa.io/get-pip.py | python - install pyenv-win --target %PYENV_ROOT%
cd %PYENV_ROOT%
rm -rf bin *distutil* install* pip* pkg_resources setuptools* wheel*
rd /s /q %TEMP%\pytmp
del %TEMP%\py.zip

pyenv install %PYTHONVERSION%

echo "python %PYTHONVERSION% の他に必要なバージョンがあればここで入れてください。

```

#### pyenvの初期設定

```powershell

pyenv global %PYTHONVERSION%
pyenv --version

grep -rl "chcp 1250" * | xargs sed -i "s/chcp 1250/chcp 65001/g"
echo sedする理由は https://github.com/pyenv-win/pyenv-win/issues/51

pyenv rehash

pyenv update
pyenv install -l

cd %PYENV%\versions
for /d %d in (*) do (
  python -m pip install --upgrade pip
  pip install setuptools wheel autopep8 flake8 pytest

  echo debug用ライブラリをダウンロードしてます
  curl -L -o %TEMP%\dev_d_%d.msi https://www.python.org/ftp/python/%d/amd64/dev_d.msi
  sleep 1
  msiexec /a %TEMP%\dev_d_%d.msi targetdir="%PYENV%\versions\%d" /qn
  del /s /q %TEMP%\dev_d_%d.msi
)

mv %LOCALAPPDATA%\Microsoft\WindowsApps\python.exe %LOCALAPPDATA%\Microsoft\WindowsApps\_python.exe
mv %LOCALAPPDATA%\Microsoft\WindowsApps\python3.exe %LOCALAPPDATA%\Microsoft\WindowsApps\_python3.exe

```
* dev_d.msiをダウンロードしている理由 -> python39_d.libが欲しいため


#### [poetry](https://github.com/python-poetry/poetry)
```powershell
curl -L https://install.python-poetry.org | python -
poetry --version
poetry self update
poetry config --list
poetry config virtualenvs.in-project true
poetry config cache-dir "%POETRY_HOME%\pypoetry\Cache"

```

### C/C++ Build Tool
#### [LLVM](https://github.com/llvm/llvm-project/releases) & [CMake](https://cmake.org/download/) & [Ninja](https://github.com/ninja-build/ninja/releases)

```powershell
echo LLVM インストール...
curl -L -o %TEMP%\llvm.zip https://github.com/llvm/llvm-project/releases/download/llvmorg-13.0.0/LLVM-13.0.0-win64.exe
7z x -o%IDEROOT% %TEMP%\llvm.zip
del %TEMP%\llvm.zip
rd /s /q %IDEROOT%\\$PLUGINSDIR

echo CMake インストール...
curl -L -o %TEMP%\cmake.zip https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.zip
7z x -o%TEMP%\cmake %TEMP%\cmake.zip
bash -c "cp -fR $TEMP/cmake/cmake*/* $IDEROOT/"
del %TEMP%\cmake.zip
rd /s /q %TEMP%\cmake

echo Ninja インストール...
curl -L -o%TEMP%\ninja.zip https://github.com/ninja-build/ninja/releases/download/v1.10.2/ninja-win.zip
7z x -o%IDEROOT%\bin %TEMP%\ninja.zip
del /s /q %TEMP%\ninja.zip

```

### C/C++ Windows headers & libraries
#### [Microsoft BuildTools](https://visualstudio.microsoft.com/ja/visual-cpp-build-tools/)
インストーラ → [VSBuildTools(2022) 直リンク](https://aka.ms/vs/17/release/vs_BuildTools.exe)

* 個別のコンポーネント : 検索窓で以下の２つを最低限選択する
 * MSVC x64 ビルド ツール 最新
 * Windows 10 SDK

##### ここはやらなくてもよいが、VSのヘッダーとライブラリをコピーしてる
```bash

cd %USRLOCAL%
bash
WKIT_S=$(echo "$WKIT" | sed "s/\\\/\//g" | sed -r "s/(.):/\/\1/g")
MSVC_ROOT_S=$(echo "$MSVC_ROOT" | sed "s/\\\/\//g" | sed -r "s/(.):/\/\1/g")
USRLOCAL_S=$(echo "$USRLOCAL" | sed "s/\\\/\//g" | sed -r "s/(.):/\/\1/g")

mkdir include lib

cp -R "$WKIT_S"/include/*/* include/
cp -R "$WKIT_S"/Lib/*/*/x64/* lib/
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Auxiliary .
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Redist .
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Tools/MSVC/*/lib/x64/* lib/
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Tools/MSVC/*/include/* include/
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Tools/MSVC/*/crt .
cp -R "$MSVC_ROOT_S"/BuildTools/VC/Tools/MSVC/*/bin/Hostx64/x64 bin
exit


echo @call ^"^%MSVC_ROOT^%\BuildTools\Common7\Tools\VsDevCmd.bat^" %* > %IDEROOT%\bin\vsdevcmd.bat
echo @call ^"^%MSVC_ROOT^%\BuildTools\VC\Auxiliary\Build\vcvarsall.bat^" %* > %IDEROOT%\bin\vcvarsall.bat
echo @call ^"^%MSVC_ROOT^%\BuildTools\VC\Auxiliary\Build\vcvarsall.bat^" x64 %* > %IDEROOT%\bin\vcvars64.bat
echo @call ^"^%MSVC_ROOT^%\BuildTools\VC\Auxiliary\Build\vcvarsall.bat^" x86 %* > %IDEROOT%\bin\vcvars32.bat

```

#### [VSCode](https://code.visualstudio.com/)
[latest zip data](https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive)

```powershell
echo VSCodeインストールしてます
curl -L -o "%TEMP%\download_vscode.zip" "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive"
7z x -aoa -o"%VSCODE_HOME%" "%TEMP%\download_vscode.zip"
del /s /q "%TEMP%\download_vscode.zip"

echo ショートカットを作成してます
set TARGET='%VSCODE_HOME%\Code.exe'
set SHORTCUT='%APPDATA%\Microsoft\Windows\Start Menu\Code.lnk'
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -Command "$ws = New-Object -ComObject WScript.Shell; $s = $ws.CreateShortcut(%SHORTCUT%); $S.TargetPath = %TARGET%; $S.Save()"
cp %SHORTCUT% %USERPROFILE%\Desktop

```

##### VSCode 個人的な初期設定一括

```powershell
echo 拡張機能をインストールします
curl -L -o %TEMP%\vscode_extensions.txt https://raw.githubusercontent.com/kirin123kirin/.vscode/main/vscode_extensions.txt
for /f %n in (%TEMP%\vscode_extensions.txt) do (code --install-extension %n)
del /s /q %TEMP%\vscode_extensions.txt

echo 全般設定の設定中
curl -L -o %APPDATA%\Code\User\settings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/settings.json

echo キーバインドの設定中
curl -L -o %APPDATA%\Code\User\keybindings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/_keybindings.json

code
mshta vbscript:execute("MsgBox(""ステータスバーのダウンロードが完了するまで待ってVSCodeを再起動してください""):close")

```

### Git config global & PyPI config
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
