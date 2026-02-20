# ディレクトリを作成する

```batch
mkdir -p C:\code\automate
mkdir -p C:\code\chromium_git
```

# depot_toolsを取得する

```batch
cd c:\code
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

# ビルドスクリプトを取得する  
このスクリプトは、Gitからのソース取得とビルドを行うが、ここでの方法ではソース取得だけに使う。

```batch
cd c:\code\automate
curl -O https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py
```

# ソースコード取得用のバッチを作成する。  
以下の内容のバッチファイルを、`C:\code\chromium_git\update.bat`に作成する。  
なお、`GN_DEFINES`で指定するフラグは背反ルール等があるので、エラーにならないように設定する必要がある。  

```batch
set GN_DEFINES=is_official_build=true is_debug=false target_cpu=x64 enable_nacl=false proprietary_codecs=true ffmpeg_branding=Chrome
# Use vs2017 or vs2019 as appropriate.
set GN_ARGUMENTS=--ide=vs2019 --sln=cef --filters=//cef/*
python ..\automate\automate-git.py --download-dir=c:\code\chromium_git --depot-tools-dir=c:\code\depot_tools --no-distrib --no-build --branch=5304
```

# ソースコードを取得する  
作成したバッチファイルを実行する。  
概ね2時間前後でソースの取得が完了する。  
取得するソースは、以下の3つになる。
- `depot_tools`(ビルド用のツール)
- `cef`のソース
- `chromium`のソース

```batch
cd c:\code\chromium_git
update.bat
```

# 取得したソースを確認する
`.git`があるフォルダで以下を実行する。  

```
git status
```

取得しているソースは、特定の`branch`、`tag`、`revision`のモノであるので、必要としているソースであるかチェックする。

# VCのソリューションファイル作成用のバッチを作成する
以下の内容のバッチファイルを、`c:\code\chromium_git\chromium\src\cef\create.bat`に作成する。  

```batch
set CEF_USE_GN=1
set GN_DEFINES=is_official_build=true is_debug=false target_cpu=x64 enable_nacl=false proprietary_codecs=true ffmpeg_branding=Chrome
set GN_ARGUMENTS=--ide=vs2019 --sln=cef --filters=//cef/*
call cef_create_projects.bat
```

# ファイルを修正する
cefプロジェクトの説明では問題なくビルドできるはずであるが、エラーが発生するため、
`C:\code\chromium_git\chromium\.gclient`を編集する。

```json
solutions = [
    {
        'managed': False,
        'name': 'src', 
        'url': 'https://chromium.googlesource.com/chromium/src.git', 
        'custom_vars': {
            'checkout_pgo_profiles': True,  // ★この部分をTrueにする
        },
        'custom_deps': {
            'build': None, 
            'build/scripts/command_wrapper/bin': None, 
            'build/scripts/gsd_generate_index': None, 
            'build/scripts/private/data/reliability': None, 
            'build/scripts/tools/deps2git': None, 
            'build/third_party/lighttpd': None, 
            'commit-queue': None, 
            'depot_tools': None, 
            'src/chrome_frame/tools/test/reference_build/chrome': None, 
            'src/chrome/tools/test/reference_build/chrome_linux': None, 
            'src/chrome/tools/test/reference_build/chrome_mac': None, 
            'src/chrome/tools/test/reference_build/chrome_win': None,
        }, 
        'deps_file': 'DEPS',
        'safesync_url': ''
    }
]
```

# VCのソリューションファイルを作成する  
このコマンドを実行すると、cefのパッチがchromiumのコードに適用されて、最終的にVisualStudioのソリューションファイルが作成される。  

```batch
cd c:\code\chromium_git\chromium\src\cef
create.bat
```

# VCのソリューションファイルでビルドを行う  
この処理は約10時間かかった。

```batch
cd c:\code\chromium_git\chromium\src
ninja -C out\Release_GN_x64 cef
```

# 参考サイト
[https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md](https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart.md)


# 参考

## cefのソースを取得する

```batch
cd c:\code\chromium_git
git clone https://bitbucket.org/chromiumembedded/cef.git
```

```batch
Cloning into 'cef'...
remote: Enumerating objects: 81784, done.
remote: Counting objects: 100% (5674/5674), done.
remote: Compressing objects: 100% (1782/1782), done.
remote: Total 81784 (delta 4243), reused 5126 (delta 3886), pack-reused 76110
Receiving objects: 100% (81784/81784), 36.31 MiB | 16.98 MiB/s, done.
Resolving deltas: 100% (68307/68307), done.
Updating files: 100% (2124/2124), done.
```

## chromiumのソースを取得する

```batch
cd c:\code\chromium_git
git clone https://github.com/chromium/chromium.git
```

```batch
Cloning into 'chromium'...
remote: Enumerating objects: 20164926, done.
remote: Counting objects: 100% (6141/6141), done.
remote: Compressing objects: 100% (3054/3054), done.
remote: Total 20164926 (delta 3038), reused 5892 (delta 2837), pack-reused 20158785
Receiving objects: 100% (20164926/20164926), 36.20 GiB | 17.80 MiB/s, done.
Resolving deltas: 100% (15917987/15917987), done.
Checking objects: 100% (67108864/67108864), done.
Updating files: 100% (412385/412385), done.
```

