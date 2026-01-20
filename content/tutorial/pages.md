+++
title = "ページの定義"
weight = 4
draft = false
+++

## このステップで実装すること

このステップでは、`pages.py` を編集します。

1 つの画面はページクラス（ロジック）とテンプレート（見た目）で構成されます。
ページクラスでは、入力{{% glossary term="フォーム" link="/reference/forms/" desc="参加者が値を入力する欄" %}}の設定や表示条件を定義します。

このステップでは、以下を実装します。

- 各ページの機能（フォーム入力、条件付き表示、待機後の処理）
- ページの表示順序 (`page_sequence`)

[実装前の準備]({{% ref "/tutorial/preparation" %}}) で整理した内容のうち、次を実装します。

| ページ | 対応する手順 | 内容 | 対象 |
|--------|-------------|------|------|
| Introduction | 手順 4 | 参加者に自分の役割を表示 | 全員 |
| Offer | 手順 5 | 独裁者役が配分額を入力 | 独裁者のみ |
| ResultsWaitPage | 手順 6 | 全員の準備完了を待ち、報酬を計算 | 全員 |
| Results | 手順 7 | 各参加者に結果と報酬を表示 | 全員 |

## 必須要素

`pages.py` には、表示する画面ごとのページクラスと、表示順序の `page_sequence` を設定します。

ページクラスは oTree が提供する `Page` または `WaitPage` を継承します。
`Page` は通常の画面、`WaitPage` は待機画面に使います。
今回は ResultsWaitPage のみ `WaitPage` を継承し、他は `Page` を継承します。

実験中、各プレイヤーは `page_sequence` に定義された順番でページを閲覧していきます。

`page_sequence` はページクラスのリストで、表示したい順番に並べます。まずは骨格となるコードを書きます。

```python { title="dictator/pages.py" }
from otree.api import Page, WaitPage


class Introduction(Page):
    pass


class Offer(Page):
    pass


class ResultsWaitPage(WaitPage):
    pass


class Results(Page):
    pass


page_sequence = [Introduction, Offer, ResultsWaitPage, Results]
```

ここから各ページの機能を実装していきます。


## Introduction ページ

このページでは、参加者に役割を表示します。

`Page` クラスには、その画面を閲覧しているプレイヤーが紐づいています。
ページクラス内では `self.player` で、テンプレートでは `player` で、現在のプレイヤーの `Player` オブジェクトにアクセスできます。

Introduction ページでは役割を表示しますが、テンプレートから `player.role` で役割情報を取得できるため、ページクラスに追加の実装は不要です。

```python { title="dictator/pages.py" }
class Introduction(Page):
    pass
```

## Offer ページ

このページでは、独裁者役が配分額を入力します。
2 つの機能を実装する必要があります。

1. 独裁者役のみにページを表示する
2. 配分額を入力するフォームを表示する

### 表示条件の設定

ページの表示条件を制御するには、`is_displayed` メソッドを使います。
このメソッドが `True` を返すと、そのプレイヤーにはページが表示されます。
それ以外の場合、ページはスキップされて、プレイヤーは次のページに進みます。

独裁者役のみに表示するため、`player.role` が `C.DICTATOR_ROLE` (= `"dictator"`) と一致するかを判定します。
`models.py` から定数クラスの `C` を読み込んでおきます。
`pages.py` の冒頭に追加します。

```python { title="dictator/pages.py" }
from otree.api import Page, WaitPage
from .models import C
```

また、`is_displayed` メソッドは次のようになります。

```python { title="dictator/pages.py" }
class Offer(Page):

    def is_displayed(self):
        return self.player.role == C.DICTATOR_ROLE
```

### フォームの設定

参加者に入力させたい{{% glossary term="フィールド" link="/reference/fields/" desc="モデルに定義するデータ項目" %}}を指定します。

`form_model` には入力データの格納先モデルを文字列（`"player"` または `"group"`）で指定します。
`form_fields` には入力させたいフィールド名をリストで指定します。

配分額 (`allocated_amount`) は `Group` モデルに定義したため、以下のように設定します。

```python { title="dictator/pages.py" }
class Offer(Page):
    form_model = "group"
    form_fields = ["allocated_amount"]

    def is_displayed(self):
        return self.player.role == C.DICTATOR_ROLE
```

独裁者役が入力した配分額は、自動で所属グループの `allocated_amount` フィールドにデータが格納されます。

## ResultsWaitPage ページ

グループ全員が WaitPage に到達すると、`after_all_players_arrive` メソッドが呼び出されます。
メソッド内では `self.group` で待機が完了したグループの `Group` オブジェクトに、`self.group.get_players()` でそのメンバーのリストにアクセスできます。

ここでは、グループ内の各メンバーの報酬を設定するために、取得したプレイヤーの `set_payoff` メソッドを実行しています。

```python { title="dictator/pages.py" }
class ResultsWaitPage(WaitPage):

    def after_all_players_arrive(self):
        for player in self.group.get_players():
            player.set_payoff()
```

## Results ページ

このページでは、独裁者役の選択額と計算した報酬を表示します。
テンプレートからそれぞれ `group.allocated_amount` と `player.payoff` で選択額と報酬を取得できるため、ページクラスに追加の実装は不要です。

```python { title="dictator/pages.py" }
class Results(Page):
    pass
```

## まとめ

このステップで実装した内容を整理します。

### ページ構成

| ページ | 継承元 | 機能 |
|--------|--------|------|
| `Introduction` | `Page` | 役割情報の表示 |
| `Offer` | `Page` | 配分額の入力（独裁者のみ） |
| `ResultsWaitPage` | `WaitPage` | 報酬計算の実行 |
| `Results` | `Page` | 結果表示 |

### 使用した Page クラスの機能

| 機能 | 使用箇所 | 説明 |
|------|----------|------|
| `is_displayed` | `Offer` | ページの条件付き表示 |
| `form_model` | `Offer` | フォームデータの格納先 |
| `form_fields` | `Offer` | 入力フィールドの指定 |

### 使用した WaitPage クラスの機能

| 機能 | 使用箇所 | 説明 |
|------|----------|------|
| `after_all_players_arrive` | `ResultsWaitPage` | 全員到着後の処理 |

### 完成したコード

```python { title="dictator/pages.py" }
from otree.api import Page, WaitPage
from .models import C


class Introduction(Page):
    pass


class Offer(Page):
    form_model = "group"
    form_fields = ["allocated_amount"]

    def is_displayed(self):
        return self.player.role == C.DICTATOR_ROLE


class ResultsWaitPage(WaitPage):

    def after_all_players_arrive(self):
        for player in self.group.get_players():
            player.set_payoff()


class Results(Page):
    pass


page_sequence = [Introduction, Offer, ResultsWaitPage, Results]
```

{{% stepcode url="https://github.com/takahiromiura/oTree_instruction_program/tree/tutorial/04-pages/tutorial/dictator-game" %}}
GitHub でコードを確認
{{% /stepcode %}}
