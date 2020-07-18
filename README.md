# test-auto-update-system
自動アップデートシステムのテスト用リポジトリ

自動アップデートは，バージョン管理システム**git**および**GitHub**を使用しています。

### Gitの設定

* ``git``はMacにプリインストールされています。
* [GitHub をはじめましょう](https://help.github.com/ja/github/getting-started-with-github)
* [GitHub に SSH で接続する](https://help.github.com/ja/github/authenticating-to-github/connecting-to-github-with-ssh)

#### リポジトリ

モジュールを初期化するためにローカルgitリポジトリのパスを指定してください。

```4d
$appName:="My First App"
$update_manager:=update_manager ($appName)

$path:="/Users/miyako/Documents/miyako@github.com/test-auto-backup-system/"

$params:=New object
$update_manager.git.setup($path)
```

#### Manifest

バージョン情報は，``manifest.json``ファイルで管理します。

**TODO**: privateリポジトリの場合，``HTTP Get``で``manifest.json``ファイルが取得できません。

**TODO**: ビルド用のランタイムの場所は4D.appと同じフォルダーであることを想定しています。4D.appがトランスロケーションされている場合，ランタイムのパスがみつからずにビルドに失敗します。その場合，アプリのフルパスを

**TODO**: ``IsOEM``キーがセットされていません。

**TODO**: macOS 10.14.6+11.3.1では，

```
xcrun altool --store-password-in-keychain
```

に問題があります。キーチェーンの「場所」が空欄になります。回避するためには，マニュアル操作で項目を追加してください。

#### GitHubの設定

リリースの菅理にはGitHubのREST API（[Web application flow](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/#web-application-flow)）を使用します。

* GitHub Appを[登録](https://github.com/settings/applications/new)します。

* ``redirect_uri`` は "http://127.0.0.1:8080/path" に設定します。

* GitHub Appの``client_id`` ``client_secret`` ``redirect_uri`` を控えます。

[Web application flow](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/#web-application-flow)はモジュールで処理することができます。

**On Web Connection**に下記のコードを記述します。

```4d
$appName:="My First App"
$update_manager:=update_manager ($appName)

$update_manager.onWebConnection()
```

4D Web Serverを起動し，下記のコードを実行します。

```4d
If (WEB Is server running)
	
	$update_manager:=update_manager (main_application_name)
	
	$params:=New object
	
	$params.client_id:="..."
	$params.client_secret:="..."
	$params.redirect_uri:="http://127.0.0.1:8080/path"
	$params.login:="{GitHubのログイン名}"
	$params.scope:="public_repo"
	$params.state:=Generate UUID
	
	$update_manager.git.authorize($params)
	
End if 
```

OAuthの認証トークンが下記のフォルダーに作成されます。

* サーバー側/Application Support/{アプリ名}/developer/git_token
* サーバー側/Application Support/{アプリ名}/developer/token_private.pem
* サーバー側/Application Support/{アプリ名}/developer/token_public.pem

以後，モジュールのメソッドでリリースの管理にGitHubの[REST API v3](https://developer.github.com/v3/repos/releases/)が使用できるようになります。

```4d
$remote:="miyako/test-auto-backup-system"
$releases:=$update_manager.git.releases($remote)
```

* [``releases()``](https://developer.github.com/v3/repos/releases/#list-releases)
* [``createRelease()``](https://developer.github.com/v3/repos/releases/#create-a-release)
* [``updateRelease()``](https://developer.github.com/v3/repos/releases/#update-a-release)
* [``removeRelease()``](https://developer.github.com/v3/repos/releases/#delete-a-release)
* [``getRelease()``](https://developer.github.com/v3/repos/releases/#get-a-release-by-tag-name)
* [``createAsset()``](https://developer.github.com/v3/repos/releases/#upload-a-release-asset)
* [``updateAsset()``](https://developer.github.com/v3/repos/releases/#update-a-release-asset)
* [``removeAsset()``](https://developer.github.com/v3/repos/releases/#delete-a-release-asset)

これらのメソッドは開発環境でビルド後処理に使用することができます。

1. **バージョン番号**を決定
1. マニフェストに**バージョン番号**を書き込み
1. ビルド
1. ビルドアプリの``Info.plist``に**バージョン番号**を書き込み
1. 署名・ディスクイメージ作成・公証・ステープル
1. GitHubにリリースを**バージョン番号**で作成
1. アプリのディスクイメージでリリースのアセットを作成（アップロード）
1. マニフェストにアセットのダウンロードURLを書き込み
1. マニフェストをリポジトリにプッシュ

* 例

```4d
$appName:="My First App"
$update_manager:=update_manager ($appName)

$remote:="miyako/test-auto-backup-system"

$releases:=$update_manager.git.releases($remote)

$tags:=$releases.extract("tag_name")

If ($tags.length#0)
	$tag:=$tags[0]
	$status:=$update_manager.git.getRelease($remote;$tag)
End if 

$status:=$update_manager.git.getRelease($remote)  //latest
```

### 移行アシスタント

#### Preferences

v17からv18にアップグレードすると，**Preferences**フォルダーの名称が変更され，バックアップ設定ファイルの拡張子が変更されます。

* Preferences > Settings
* BackupBackup.xml > backup.4DSettings

4Dが自動的に書き換える上記のファイル以外で**Preferences**フォルダーに置かれているカスタム設定ファイルをアプリの外に移動することができます。

```4d
$appName:="My First App"
$update_manager:=update_manager ($appName)

$update_manager.preferences.migrate()
```

拡張子``4DSettings``以外のファイルとフォルダーを``/Application Support/{アプリ名}/preferences``に移動します。すでにフォルダーが存在する
場合，上書きを避けるため，フォルダー名に連番がつけられます。

#### Data

データファイル・インデックスファイル・ログファイルをアプリの外に移動します。最大で **2** 回の再起動が発生します。移行が済んでいなければ，再起動は発生しませんので，アプリのスタートアップにメソッドを実行することができます。

```4d
C_BOOLEAN($shouldRestart)
$shouldRestart:=$update_manager.prep()

If ($shouldRestart)
	$update_manager.restart()
End if 
```

1. ジャーナルを停止 → 一時データファイルに切り替え → **再起動**
1. データファイルを``/Application Support/{アプリ名}/data``に移動
1. インデックスファイルを``/Application Support/{アプリ名}/journal``に移動 → **再起動**
1. ジャーナルを開始 → バックアップ

### ダイアログ

マニフェストのURLを指定すると，必要に応じ，バックグラウンドでダウンロードが実行されます。過去に途中までダウンロードしたファイルがある場合，ファイルを削除してからダウンロードを開始します。ダイアログを閉じた後もダウンロードは継続します。

```4d
$manifestUrl:="https://raw.githubusercontent.com/miyako/test-auto-backup-system/master/manifest.json"
$update_manager.check($manifestUrl)
```

<img width="453" alt="Screen Shot 2020-06-26 at 17 44 27" src="https://user-images.githubusercontent.com/1725068/85838557-b7092100-b7d4-11ea-8fb5-38a5a1299d8d.png">

ダウンロードが完了すると，ディスクイメージが展開され，アプリが取り出されます。アップデートの準備ができると，「いますぐ更新」ボタンが表示されます。

<img width="453" alt="Screen Shot 2020-07-01 at 14 45 38" src="https://user-images.githubusercontent.com/1725068/86207484-9b719200-bba9-11ea-9f47-eaa42bb09e2f.png">

* ビルド〜署名〜申請〜公証〜押印〜公開

<img width="742" alt="スクリーンショット 2020-07-06 20 14 37" src="https://user-images.githubusercontent.com/1725068/86587837-6019fd80-bfc5-11ea-99a1-831b5423569e.png">
