+++
title = "テンプレート作成"
weight = 5
draft = false
+++

このステップでは、各ページの見た目を定義する HTML テンプレートを作成します。

## このステップで作成するもの

`pages.py` で定義した各ページクラスに対応するテンプレートファイルを作成します。

| ファイル名 | 対応するページクラス | 内容 |
|-----------|---------------------|------|
| `Introduction.html` | `Introduction` | 役割の表示 |
| `Offer.html` | `Offer` | 配分額の入力フォーム |
| `Results.html` | `Results` | 結果と報酬の表示 |

テンプレートファイルはアプリケーションフォルダ (`dictator/`) 内に、ページクラスと同じ名前で作成します。

```text
dictator/
├── __init__.py
├── models.py
├── pages.py
├── Introduction.html
├── Offer.html
└── Results.html
```

{{% notice style="note" title="WaitPage のテンプレート" %}}
`WaitPage` を継承したページ（`ResultsWaitPage`）は、デフォルトの待機画面が表示されるためテンプレートは不要です。
{{% /notice %}}

## 基本概念の説明

### HTML とは

HTML (Hyper Text Markup Language) はウェブページのコンテンツをタグで記述するマークアップ言語です。
`<タグ名>内容</タグ名>` という形式で要素を記述します。

```html
これは <strong>太字</strong>です。
```

この例では `<strong>` が開始タグ、`</strong>` が終了タグ、「太字」が内容です。
よく使うタグには以下のようなものがあります。

| タグ | 意味 | 例 | 結果 |
|------|------|-----|------|
| `<p>` | 段落 | `<p>1行目</p>2行目` | <p>1行目</p>2行目 |
| `<strong>` | 太字 | `ここは<strong>重要</strong>` | ここは<strong>重要</strong> |
| `<a>` | リンク | `ここに<a href="...">リンク</a>` | ここに<a href="https://example.com">リンク</a> |

### テンプレートとは

例えば、独裁者ゲームでの結果を表示するページを作りたいとします。
独裁者は 10 通りの選択ができ、また結果の画面は自分が独裁者か受け手かで異なります。
役割と選択結果に応じてページの内容を変える場合、2 x 10 = 20 通り必要です。
これに対応するために 20 個の HTML ファイルを作成するのは骨が折れます。

その代わりに、HTML ファイルの原型となるファイルを作成しておき、結果に応じて動的にファイルを作成することができます。
この原型となる HTML ファイルのことをテンプレートと呼びます。

## oTree のテンプレート

oTree では、テンプレートファイルに対応するページクラスが動的に HTML ファイルを作成します。

### テンプレートタグ

oTree のテンプレート記法を使うには、二重波括弧 `{{ }}` を使います。
この二重波括弧を **テンプレートタグ** と呼びます。

```html
{{ 何らかの処理 }}
```

テンプレートタグには以下のような用途があります。

| 用途 | 例 |
|------|-----|
| ブロック定義 | `{{ block title }}` |
| 変数の表示 | `{{ player.role }}` |
| フォーム生成 | `{{ formfields }}` |

### ブロック構文

ページのタイトルやコンテンツなどは、ブロック構文を使います。
`{{ block <name> }}` と `{{ endblock }}` で囲まれた部分が <name> ブロックの内容になります。
ページのタイトルは `title` ブロックを使います。
メインのコンテンツは `content` ブロック内に記載します。

### 変数の表示

テンプレート内で `{{ <name> }}` と書くと、その位置に値が表示されます。

```html
あなたの役割: {{ player.role }}
```

今回の例だと、独裁者役は次のように表示されます。

```html
あなたの役割: dictator
```

テンプレートからは `player`（現在のプレイヤー）や `group`（所属グループ）にアクセスできます。
Python と同様に、`player.role` や `group.id` のように属性にアクセスできます。

| 変数 | 説明 |
|------|------|
| `player` | 現在のプレイヤーの `Player` インスタンス |
| `group` | 所属グループの `Group` インスタンス |
| `subsession` | 現在のラウンドの `Subsession` インスタンス |
| `C` | 定数クラス |

### フォーム

参加者に値を入力させるには `{{ formfields }}` を使います。
`pages.py` の `form_fields` で指定した{{% glossary term="フィールド" link="/reference/fields/" desc="モデルに定義するデータ項目" %}}の入力欄が自動的に生成されます。

### 次に進むボタン

次のページに進むボタンは `{{ next_button }}` で表示します。

### 基本構成

テンプレートファイルの基本的な内容は次のようになります。

```html
{{ block title }}
    ページのタイトル
{{ endblock }}

{{ block content }}
    ページの内容

    {{ next_button }}
{{ endblock }}

```


## Introduction.html

参加者に役割を表示するページです。
ページのタイトルは「役割情報」とします。
`player.role` で役割情報を取得します。
`p` タグでボタンと少し余白を空けるようにしています。

{{% notice style="tip" title="表示の調整" %}}
    文字の大きさやボタンの配置などの微調整は、実際の画面を見て行います。
{{% /notice %}}

```html { title="dictator/Introduction.html" }
{{ block title }}
    役割情報
{{ endblock }}

{{ block content }}
    <p>あなたの役割: {{ player.role }}</p>
    {{ next_button }}
{{ endblock }}
```

しかし、これだと独裁者役は `dictator`、受け手役は `receiver` という文字列が表示されます。
独裁者役には「選択者」、受け手役には「受取人」という役割のラベルを表示したい場合は、`if` 構文を用います。

`{{ if <condition>}}` と `{{ endif }}` で囲った部分が条件式を満たした時に表示されます。
条件が満たされない場合に表示したい場合、途中に `{{ else }}` を挟みます。
これを利用すると、次のように書けます。

```html { title="dictator/Introduction.html" }
{{ block title }}
    役割情報
{{ endblock }}

{{ block content }}
    {{ if player.role == C.DICTATOR_ROLE }}
        <p>あなたの役割: 選択者</p>
    {{ else }}
        <p>あなたの役割: 受取人</p>
    {{ endif }}
        {{ next_button }}
{{ endblock }}
```

## Offer.html

独裁者役が配分額を入力するページです。
初期保有量の値を `C.INITIAL_ENDOWMENTS` で取得します。
`{{ formfields }}` と書くと、`pages.py` で指定した `allocated_amount` の入力欄が生成されます。

```html { title="dictator/Offer.html" }
{{ block title }}
    配分の選択
{{ endblock }}

{{ block content }}
    <p>{{ C.INITIAL_ENDOWMENTS }} ドルのうち、いくら受取人に渡しますか？</p>
    <p>0 から {{ C.INITIAL_ENDOWMENTS }} までの整数値を入力してください。</p>

    {{ formfields }}
    {{ next_button }}

{{ endblock }}
```

## Results.html

結果と報酬を表示するページです。
`{{ group.allocated_amount }}` で独裁者が入力した配分額、`{{ player.payoff }}` で計算された報酬が表示されます。

```html { title="dictator/Results.html" }
{{ block title }}
    結果
{{ endblock }}

{{ block content }}
    <p>独裁者の配分額: {{ group.allocated_amount }}</p>
    <p>あなたの報酬: {{ player.payoff }}</p>

    {{ next_button }}
{{ endblock }}
```


## まとめ

このステップで作成した内容を整理します。

### 作成したファイル

| ファイル | 内容 |
|---------|------|
| `Introduction.html` | 役割の表示 |
| `Offer.html` | 配分額の入力フォーム |
| `Results.html` | 結果と報酬の表示 |

{{% stepcode url="https://github.com/takahiromiura/oTree_instruction_program/tree/tutorial/05-templates/tutorial/dictator-game" %}}
GitHub でコードを確認
{{% /stepcode %}}
