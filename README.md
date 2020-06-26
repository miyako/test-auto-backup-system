# test-auto-backup-system
自動バックアップシステムのテスト用リポジトリ

#### Gitの設定

自動アップデートは，バージョン管理システム**git**および**GitHub**を使用しています。

#### Mac

* ``git``はMacにプリインストールされています。
* [GitHub をはじめましょう](https://help.github.com/ja/github/getting-started-with-github)
* [GitHub に SSH で接続する](https://help.github.com/ja/github/authenticating-to-github/connecting-to-github-with-ssh)

#### リポジトリ

モジュールを初期化するためにリポジトリのローカルパスを指定してください。このリポジトリは，``git``でバージョン管理されていることが前提です。

```4d
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
```

バージョン番号の``patch`` ``minor`` ``major`` 番号（ドットで区切られた数値）をインクリメントします。必要に応じて``manifest.json``ファイルが作成されます。
