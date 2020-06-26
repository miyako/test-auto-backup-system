# test-auto-backup-system
自動バックアップシステムのテスト用リポジトリ

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

```4d
$tag_name:=$update_manager.git.version
```

リポジトリに``manifest.json``ファイルがなければ，```0.0.0``が返されます。

```4d
$tag_name:=$update_manager.git.patch().minor().major()
$update_manager.git.reset()
```

バージョン番号の``patch`` ``minor`` ``major`` 番号（ドットで区切られた数値）をインクリメントします。必要に応じて``manifest.json``ファイルが作成されます。``0.0.0``にリセットすることもできます。

```4d
$update_manager.git.push()
```

``manifest.json``ファイルをGitHubにプッシュします。``git``がGitHubにSSHで接続できるように設定されていなければなりません。

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

**運用環境**: これらのファイルをサーバー側の該当箇所に置いてください。

