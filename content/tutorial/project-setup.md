+++
title = "プロジェクト作成"
weight = 2
draft = true
+++

oTree プロジェクトとアプリを作成します。

## プロジェクトの作成

`otree startproject` は oTree プロジェクトの雛形を作成するコマンドです。

```
otree startproject <project_name>
```

このコマンドを実行すると、指定した名前のフォルダが作成され、その中に oTree プロジェクトに必要なファイル一式が生成されます。

### 独裁者ゲームプロジェクトの作成

任意のディレクトリで以下のコマンドを実行します。
この例では、`dictator-game` というプロジェクトにします。

```bash
otree startproject dictator-game
```

実行すると、デモ用のゲームを入れるか聞かれます。
ここでは `n` を選択します。

```
Include sample games? (y or n): n
```

作成されたディレクトリに移動します。

```bash
cd dictator-game
```

この時点でのフォルダ構成は以下のようになります。

```
dictator-game/
├── _static/
├── __init__.py
├── .gitignore
├── Procfile
├── requirements.txt
└── settings.py
```

この中で重要なのは `settings.py` と `_static/` フォルダです。
`settings.py` にはセッション構成や言語設定などプロジェクト全体の設定を記述します。
`_static/` には CSS や画像ファイルなどを配置します。
初期状態では `_static/global/empty.css` が作成されています。

このチュートリアルでは `settings.py` の設定方法を後ほど説明しますが、`_static/` については扱いません。
これについては、[静的ファイル]({{< relref "/guide/static-files" >}}) を参照してください。

{{% notice note %}}
`settings.py` と `_static` フォルダ（空でも可）以外のファイルは削除しても問題ありません。
以降のフォルダ構成の説明では、これらのファイルは省略します。
{{% /notice %}}

{{% details title="oTree 5 のフォルダ構成" %}}

oTree 5 で startproject を実行した場合は `_templates` フォルダが追加されています。

```
dictator-game/
├── _static/
├── _templates/
├── __init__.py
├── .gitignore
├── Procfile
├── requirements.txt
└── settings.py
```

`_static/` には `global/empty.css`、`_templates/` には `global/Page.html` が含まれています。

{{% /details %}}


## アプリケーションの作成

`otree startapp` はアプリケーションの雛形を作成するコマンドです。

```
otree startapp <app_name>
```

このコマンドを実行すると、指定した名前のフォルダが作成され、その中にアプリに必要なファイルが生成されます。

### dictator アプリの作成

プロジェクトディレクトリ内で以下のコマンドを実行します。

```bash
otree startapp dictator
```

アプリが作成されると、フォルダ構成は以下のようになります。

```
dictator-game/
├── _static/
├── dictator/
│   ├── __init__.py
│   ├── MyPage.html
│   └── Results.html
└── settings.py
```

oTree において、アプリは単なるフォルダにすぎません。
アプリ名はフォルダ名を指し、同名のアプリは作成できません。

{{% stepcode url="https://github.com/takahiromiura/oTree_instruction_program/tree/tutorial/01-app-created/tutorial/dictator-game" %}}
Githubでコードを確認
{{% /stepcode %}}


### アプリ内ファイルの整理

このチュートリアルでは、モデルとページのコードを `__init__.py` に書かず、`models.py` と `pages.py` に分けて記述します。
作成した `dictator/` フォルダ内のファイルを次のように整理してください。

1. `__init__.py` の内容をすべて削除し、空にする（ファイル自体は削除しない）
2. `models.py` と `pages.py` を新規作成する
3. `MyPage.html` と `Results.html` を削除する（画面の作成時に改めて必要なファイルを作成します）

整理後のファイル構成は以下のようになります。

```
dictator-game/
├── _static/
├── dictator/
│   ├── __init__.py
│   ├── models.py
│   └── pages.py
└── settings.py
```

{{% notice style="note" title="ファイル構成について" %}}
oTree の公式ドキュメントでは {{< glossary term="no-self format" link="/reference/no-self-format" >}} が使われています。
この形式ではモデルやページのコードをすべて `__init__.py` 内に記述します。
{{% /notice %}}

{{% stepcode url="https://github.com/takahiromiura/oTree_instruction_program/tree/tutorial/02-initial-setup/tutorial/dictator-game" %}}
Githubでコードを確認
{{% /stepcode %}}

これで最初の設定が完了しました。
次のページから、実際にコードを書いていきます。
