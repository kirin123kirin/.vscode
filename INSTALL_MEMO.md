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
```Batchfile
set IDEROOT C:\ide
set PYTHONVERSION 3.9.6

set VSCODE_HOME %IDEROOT%\VSCode
set USRLOCAL %IDEROOT%\usr\local
set PYENV_ROOT %USRLOCAL%\pyenv
set PYENV %PYENV_ROOT%\pyenv-win
set PYTHONPATH %PYENV%\versions\%PYTHONVERSION%
set POETRY_HOME %USRLOCAL%\poetry
set NODEJS_HOME %USRLOCAL%\nodejs

setx Path "C:\Program Files\7-Zip;C:\Program Files (x86)\sakura;%PYENV%\bin;%PYENV%\shims;%PYTHONPATH%;%PYTHONPATH%\Scripts;%PYTHONPATH%\Tools\scripts;%POETRY_HOME%\bin;%IDEROOT%\bin;%IDEROOT%\cmd;%IDEROOT%\mingw64\bin;%IDEROOT%\usr\bin;%VSCODE_HOME%\bin;%NODEJS_HOME%;%APPDATA%\npm"

setx IDEROOT %IDEROOT%
setx PYTHONVERSION %PYTHONVERSION%
setx VSCODE_HOME %VSCODE_HOME%
setx USRLOCAL %USRLOCAL%
setx PYENV_ROOT %PYENV_ROOT%
setx PYENV %PYENV%
setx PYTHONPATH %PYTHONPATH%
setx POETRY_HOME %POETRY_HOME%
setx NODEJS_HOME %NODEJS_HOME%


exit

```

## <span style="color: red; ">注意：IDEROOT</span>に既存のディレクトリを指定したら
同名のファイル又はディレクトリがある場合には、上書き良否確認などは一切せず
<span style="color: red; ">強制的に上書き</span>するので
<span style="color: red; ">新規ディレクトリの指定</span>を強く推奨する。

### [ローカル変数"GOMI"とは？](https://zenn.dev/ef/articles/fede252753800b12f42b)

## 2. とりあえず入れるもの
以下最新版をインストールします
### (1) [Windows用Wget](https://sevenzip.osdn.jp/download.html) ※Invoke-WebRequestは遅すぎるのとcurlよりファイルサイズが小さいため
### (2) [7zip](https://sevenzip.osdn.jp/download.html) ※この後の作業で必須
### (3) [サクラエディタ](https://github.com/sakura-editor/sakura/releases) ※好み。なんでもよい
### (4) [Git for Windows](https://github.com/git-for-windows/git/releases) ※Git必須なのと、mingw環境もそこそこ活用するため

```Batchfile
## Wget.exe download
$links = (Invoke-WebRequest -UseBasicParsing -Uri "https://github.com/webfolderio/wget-windows/releases").Links
$wgeturl = [string]::Concat("https://github.com", ($links.href | Select-String -Pattern ".*64bit-OpenSSL.zip" | Select-Object -first 1) )
Invoke-WebRequest -UseBasicParsing -Uri $wgeturl -OutFile .\wget.zip
Expand-Archive -Force -Path .\wget.zip -DestinationPath "${Env:IDEROOT}\usr\bin"
if ($?) { del .\wget.zip }

## Make Github release Latest URL get function
Function GitLatestVersion ($url, $pattern) {
    $lasthref = wget.exe -nv -qO- $url | Select-String -Pattern ("<a href=.*" + $pattern) | Select-Object -first 1
    [regex]::replace($lasthref, '^.*<a href="(.+)" rel=.*$', { "https://github.com" + $args.groups[1].value })
}

## 7Zip Install
wget.exe https://sourceforge.net/projects/sevenzip/files/latest/download -O 7zip.exe
Start-Process -Verb runas .\7zip.exe
if ($?) { del .\7zip.exe }

## Sakura Editor Install
$sakuraurl = GitLatestVersion "https://github.com/sakura-editor/sakura/releases" "Win32-Release-Installer.zip"
wget.exe -O .\sakura.zip $sakuraurl
7z x .\sakura.zip
Start-Process -Verb runas .\sakura_install*.exe
if ($?) { del .\sakura* }

## Git for Windows Install
$giturl = GitLatestVersion "https://github.com/git-for-windows/git/releases" "Git.*64-bit.tar."
wget.exe -O .\git-for-windows.tar.bz2 $giturl
echo %IDEROOT%にインストールしてます
7z.exe x .\git-for-windows.tar.bz2 -bsp2
if ($?) { del .\git-for-windows.tar.bz2 }
7z.exe x -o"${Env:IDEROOT}" .\git-for-windows.tar -aoa -bsp2
if ($?) { del .\git-for-windows.tar }
rm -Recurse ${Env:IDEROOT}\'\$PLUGINSDIR\'

exit

```
* このエラーは気にしない
  * 「ERROR: Cannot create symbolic link : クライアントは要求された特権を保有していません。 : fd, stderr, stdin, stdout, mtab」

## 3. 個人的に外せない開発環境
### (1) 個別コマンドの作成
1. セットアップ用
この後の手順でダウンロードしまくる作業用のシェル

```shell
bash

cat <<'EOF' > $IDEROOT/cmd/dunzip.sh
#!bash
if [[ $# < 2 ]]; then
    echo "This Script is download and extract" >&2
    MYNAME=`basename $0 | sed -E "s/\.sh$//g"`
    echo "Usage: ${MYNAME} <extract target directory> <download URL> [extract wildcard rule]" >&2
    exit 1
fi

set -eu
RETCODE=0
function catch {
  RETCODE=1
}
trap catch ERR


TMPNAME="${LOCALAPPDATA//\\/\/}"/Temp/workdir_download_will_unzip

rm -rf "${TMPNAME}"*
curl -L -o "${TMPNAME}.zip" "$2"

if [[ ! -d "$1" ]]; then
    mkdir "$1"
fi

if [[ $# == 2 ]]; then
  7z x -o"$1" "${TMPNAME}.zip" -aoa
else
  7z x -o"${TMPNAME}" "${TMPNAME}.zip" -aoa
  cp -fR "${TMPNAME}"/$3 "$1"
fi

rm -rf "${TMPNAME}"*

exit $RETCODE
EOF

cat <<'EOF' > $IDEROOT/cmd/getlatest.sh
#!bash
if [[ $# < 1 ]]; then
    echo "This Script is find latest release version for windows installer file" >&2
    MYNAME=`basename $0 | sed -E "s/\.sh$//g"`
    echo "Usage: ${MYNAME} <URL>" >&2
    echo "Example: ${MYNAME} https://github.com/git-for-windows/git/releases" >&2
    exit 1
fi

URL="$1"

if [[ $URL =~ ^https://github.com.*releases.* ]]; then
  curl -sSL $URL | grep "<a href.*release.*download.*rel=\"nofollow\"" | sed -E 's;.*href="([^"]*).*;https://github.com\1;g' > $TEMP/work_latestlist.txt
  grep `head -1 $TEMP/work_latestlist.txt | sed -E "s;.*download/([^/]+)/.*;\1;g"` $TEMP/work_latestlist.txt && rm -f $TEMP/work_latestlist.txt
elif [[ $URL =~ ^https://www.python.org/ftp.* ]]; then
  curl -sSL https://www.python.org/downloads | grep '<a class="button" href="http' | sed -r 's;.*href="([^"]+)".*;\1;g' | grep ".exe$"
elif [[ $URL =~ ^https://nodejs.org/.* ]]; then
  curl -sSL https://nodejs.org/dist/latest/ | sed -E 's;.*href="([^"]+)".*;https://nodejs.org/dist/latest/\1;g' | grep "win-x64.zip"
else
  echo "error: Unknown URLType $URL" >&2
  exit 1
fi
EOF


cat <<'EOF' > $IDEROOT/cmd/dunzip.cmd
@echo off
@rem This Script is Simply Transfer to (Same Filename).sh
bash %~dp0\%~n0.sh %*
EOF


unix2dos $IDEROOT/cmd/dunzip.sh
unix2dos $IDEROOT/cmd/getlatest.sh
unix2dos $IDEROOT/cmd/dunzip.cmd

cp $IDEROOT/cmd/dunzip.cmd $IDEROOT/cmd/getlatest.cmd

set GOMI=%LOCALAPPDATA%\Microsoft\WindowsApps
for /f "tokens=*" %a in ('echo %Path% | sed -E "s@(.*)(;%GOMI%)(.*)@\1\3\2@g"') do setx Path "%a"
exit

```

お気に入りのコマンドDL
2. [fzf](https://github.com/junegunn/fzf#windows)をインストール
3. [RipGrep](https://github.com/BurntSushi/ripgrep)をインストール
4. [RipGrep-all](https://github.com/phiresky/ripgrep-all)をインストール

```Batchfile

for /f "tokens=*" %u in ('getlatest "https://github.com/junegunn/fzf/releases" ^| grep windows_amd64') do dunzip %IDEROOT%/usr/bin "%u"

for /f "tokens=*" %u in ('getlatest https://github.com/BurntSushi/ripgrep/releases ^| grep x86_64.*windows-msvc.zip') do dunzip %IDEROOT%/usr/bin "%u" ripgrep-*/rg.exe

for /f "tokens=*" %u in ('getlatest https://github.com/phiresky/ripgrep-all/releases ^| grep "x86_64.*windows-msvc.zip"') do dunzip %IDEROOT%/usr/bin "%u" ripgrep*/rga*.exe

```

### (2) [Python(pyenv-win)](https://github.com/pyenv-win/pyenv-win)

```Batchfile
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

```Batchfile

echo pyenv Issue51関連の不具合修正してます
grep -rl "chcp 1250" * | xargs sed -i.bak "s/chcp 1250/chcp 932/g"
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

python -V

```

### (4) [poetry](https://github.com/python-poetry/poetry)
```Batchfile
curl -L https://install.python-poetry.org | python -
poetry --version
poetry self update
poetry config --list
poetry config virtualenvs.in-project true
poetry config cache-dir "%POETRY_HOME%\pypoetry\Cache"
poetry config --list

```

### (5) C/C++ Build Tool
#### [LLVM](https://github.com/llvm/llvm-project/releases) & [CMake](https://cmake.org/download/) & [Ninja](https://github.com/ninja-build/ninja/releases)

```Batchfile
echo LLVM インストール...
for /f "tokens=*" %u in ('getlatest https://github.com/llvm/llvm-project/releases ^| grep win64.exe$') do dunzip %IDEROOT% "%u"

echo CMake インストール...
for /f "tokens=*" %u in ('getlatest https://github.com/Kitware/CMake/releases ^| grep windows-x86_64.zip') do dunzip %IDEROOT% "%u" cmake*/*

echo Ninja インストール...
for /f "tokens=*" %u in ('getlatest https://github.com/ninja-build/ninja/releases ^| grep win') do dunzip %IDEROOT%/bin "%u"

```

### (6) C/C++ Windows headers & libraries
#### [Microsoft BuildTools](https://visualstudio.microsoft.com/ja/visual-cpp-build-tools/)

### インストールウィザード手順
* 個別のコンポーネントタブを選択 : 検索窓で以下の２つを最低限選択する
 * MSVC x64 ビルド ツール 最新
 * Windows 10 SDK

```
curl -L https://aka.ms/vs/17/release/vs_BuildTools.exe -o %TEMP%\vs_BuildTools.exe
cmd /k %TEMP%\vs_BuildTools.exe && rm %TEMP%\vs_BuildTools.exe

```

### (7) [VSCode](https://code.visualstudio.com/)

```Batchfile
echo VSCodeインストールしてます
dunzip "%VSCODE_HOME%" "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-archive"

echo VSCodeのショートカットを作成してます
set TARGET='%VSCODE_HOME%\Code.exe'
set SHORTCUT='%APPDATA%\Microsoft\Windows\Start Menu\Code.lnk'
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -Command "$ws = New-Object -ComObject WScript.Shell; $s = $ws.CreateShortcut(%SHORTCUT%); $S.TargetPath = %TARGET%; $S.Save()"
cp %SHORTCUT% %USERPROFILE%\Desktop

```

### (8) 必要なら[Node.js](https://nodejs.org/ja/)
```Batchfile
for /f "tokens=*" %u in ('getlatest https://nodejs.org/dist') do dunzip %NODEJS_HOME% "%u" node-v*/*

```

## 4. 個人的な初期設定
### (1) VSCode

```Batchfile
echo 拡張機能をインストールします
bash -c 'curl -sSL https://raw.githubusercontent.com/kirin123kirin/.vscode/main/vscode_extensions.txt | xargs -L1 code --install-extension'

echo 全般設定の設定中
curl -L -o %APPDATA%\Code\User\settings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/settings.json

echo キーバインドの設定中
curl -L -o %APPDATA%\Code\User\keybindings.json https://raw.githubusercontent.com/kirin123kirin/.vscode/main/_keybindings.json

mshta vbscript:execute("MsgBox(""ステータスバーのダウンロードが完了するまで待ってVSCodeを再起動してください""):close")

code

```

### (2) Git config global & PyPI config
ユーザ名とEmailを入力してください。

```shell
echo git configとpypircの設定を行います。

bash

cat<<'EOF' > ~/.gitconfig
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

cat<<'EOF' > ~/.pypirc
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
### INCLUDE, LIBPATHの環境変数を作りたいだけ
Windows SDK? Visual Studioのパスが死ぬほどめんどくさいので
無理やり環境変数INCLUDE、LIBPATHをぶち込む

```shell

bash

IDEROOT_S=$(echo $IDEROOT | sed "s;\\\;/;g")
WKIT=$(reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Kits\Installed Roots" | grep "KitsRoot" | sed "s;\\\;/;g" | sed -E "s;.*KitsRoot.+\s\s([^\s].+)/$;\1;g")

MSVC_ROOT=$(reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\Setup" | grep "SharedInstallationPath" | sed "s;\\\;/;g" | sed -E "s;.*SharedInstallationPath.+\s\s([^\s].+)/Shared/?$;\1;g")
MSBUILD=`ls -d "$MSVC_ROOT"/[0-9]*/BuildTools | tail -1`
MSVC=`ls -d "$MSBUILD"/VC/Tools/MSVC/[0-9]* | tail -1`
CLANGVERSION=`ls -d "$IDEROOT_S"/lib/clang | tail -1`

INCLUDE=`ls -d "$(ls -d "$WKIT"/Include/[0-9]* | tail -1)"/* | tr '\n' ';'`
INCLUDE="${INCLUDE};${MSVC}/include;$INCLUDE;$MSVC/include;$IDEROOT_S/usr/local/poetry/venv/Include;$IDEROOT_S/lib/clang/${CLANGVERSION}/include;$IDEROOT_S/include"

LIBPATH=`ls -d "$(ls -d "$WKIT"/Lib/[0-9]* | tail -1)"/*/$ARCH | tr '\n' ';'`
LIBPATH="${LIBPATH};${MSVC}/lib/${ARCH};${LIBPATH};$IDEROOT_S/lib/clang/${CLANGVERSION}/lib"
LIBPATH="${LIBPATH};$IDEROOT_S/usr/local/pyenv/pyenv-win/libexec/libs;$IDEROOT_S/usr/lib;$IDEROOT_S/lib"

setx INCLUDE "${INCLUDE}"
setx LIBPATH "${LIBPATH}"

cat <<EOF > $IDEROOT_S/vsdevcmd.cmd
@echo off
@call "$MSBUILD/Common7/Tools/VsDevCmd.bat" %*
EOF
cp "$IDEROOT_S"/vsdevcmd.cmd "$IDEROOT_S"/vcvarsall.cmd

exit

```
以上、
