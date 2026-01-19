+++
title = "モデルの定義"
weight = 3
draft = true
math = true
+++

## このステップで実装すること

このステップでは、`models.py` に実験で扱うデータを定義します。具体的には以下の 3 つです。

- パラメータの設定
- データ構造の定義（フィールド）
- データ操作のロジック（メソッド）

[実装前の準備]({{% ref "/tutorial/preparation" %}}) で整理した内容のうち、次を実装します。

- 設定するパラメータ（グループサイズ、役割、初期保有量、配分額の範囲）
- 追加するフィールド（配分額）
- 実装するロジック（報酬計算）

また、グループ分けと役割割り当てはデフォルトの動作を使用します。

## 必須クラス

各アプリケーションでは、以下の 4 つのクラスを定義する必要があります。
また、これらは oTree が提供する基底クラスを継承します。

| クラス | 継承元 | 説明 |
|--------|--------|------|
| `C` | `BaseConstants` | 定数 |
| `Subsession` | `BaseSubsession` | サブセッション |
| `Group` | `BaseGroup` | グループ |
| `Player` | `BasePlayer` | プレイヤー |

このうち `C` 以外の `Subsession`、`Group`、`Player` は {{% glossary term="ORM" link="/reference/orm/" desc="オブジェクトとデータベースを対応付ける仕組み" %}} モデルです。
これらのクラスにデータベースに保存する値 ({{% glossary term="フィールド" link="/reference/fields/" desc="ORM モデルに定義するデータ項目" %}}) を定義します。

また、`C` クラスには最低限以下の 3 つのクラス属性を定義する必要があります。

| 属性 | 説明 | この例での値 |
|------|------|-------------|
| `NAME_IN_URL` | URL に表示されるアプリ名 | `choice_task` |
| `PLAYERS_PER_GROUP` | グループサイズ | `2` |
| `NUM_ROUNDS` | ラウンド数 | `1` |

独裁者ゲームは 2 人 1 組で 1 ラウンドのみ行うため、上記のように設定します。

`Subsession`、`Group`、`Player` クラスは `pass` のみで定義しても構いません。
骨格となるコードは次のようになります。

```python { title="dictator/models.py" }
from otree.api import BaseConstants, BaseSubsession, BaseGroup, BasePlayer, models

class C(BaseConstants):
    NAME_IN_URL = "choice_task"
    PLAYERS_PER_GROUP = 2
    NUM_ROUNDS = 1

class Subsession(BaseSubsession):
    pass

class Group(BaseGroup):
    pass

class Player(BasePlayer):
    pass
```

`models` はフィールドを定義するためのモジュールです。
ここからさらに必要となるデータを定義していきます。

## C の定義

定数クラスとなる `C` には、ラウンドやグループ・プレイヤーによって変化しないアプリ全体の設定情報を入れます。
このクラスは実験中に変更することができません。

役割は通常このクラスで定義します。
役割名はクラス属性の名前を `ROLE_` で始めるか、最後を `_ROLE` で終わらせる必要があります。
役割を定義すると、グループごとに定義した役割が割り当てられます。

{{% notice style="warning" title="役割の数" %}}
    役割を定義する場合、グループのサイズと役割数が同じになる必要があります。
{{% /notice %}}

この例では、独裁者役と受け手役を定義します。

```python { title="dictator/models.py" }

class C(BaseConstants):
    NAME_IN_URL = "choice_task"
    PLAYERS_PER_GROUP = 2
    NUM_ROUNDS = 1

    DICTATOR_ROLE = "dictator"
    RECEIVER_ROLE = "receiver"

```

また、初期保有量もここに入れておくのが良いでしょう。
この例では、独裁者の初期保有量を 10 (ドル) とします。

```python { title="dictator/models.py" }
class C(BaseConstants):
    NAME_IN_URL = "choice_task"
    PLAYERS_PER_GROUP = 2
    NUM_ROUNDS = 1

    DICTATOR_ROLE = "dictator"
    RECEIVER_ROLE = "receiver"

    INITIAL_ENDOWMENTS = 10

```

## Subsession の定義

ラウンドごとに、グループや役割を変えたい場合、そのロジックをここに記載します。
デフォルトでは参加者 ID 順にグループが形成され、グループ ID と役割が割り当てられます。
この例ではデフォルトのままにします。

## Group の定義

独裁者役が選択した受け手への配分額を記録する必要があります。
そのために、フィールドの名前とデータの型を定義します。
配分額は整数なので、`IntegerField` を使用します。
フィールドは `models.IntegerField()` のように定義します。

```python { title="dictator/models.py" }
class Group(BaseGroup):

    allocated_amount = models.IntegerField()

```

さらに、配分額は 0 から初期保有量 (10) までなので、最小値と最大値を `min`, `max` オプションを使って設定します。

```python { title="dictator/models.py" }
class Group(BaseGroup):

    allocated_amount = models.IntegerField(
        min = 0,
        max = C.INITIAL_ENDOWMENTS, # 10
    )

```

## Player の定義

`C` で役割を定義すると、`role` 属性に割り当てられた役割名 (この例では `dictator` もしくは `receiver`) が格納されます。

{{% notice style="note" title="payoff フィールド" %}}
`Player` クラスにはデフォルトで `payoff` という、このラウンドでの報酬額を格納するためのフィールドが定義されています。
{{% /notice %}}

この例では追加のフィールドは定義しません。

### 報酬計算

このラウンドでの報酬の計算ロジックを定義します。
報酬額は役割によって異なることに注意してください。

`set_payoff` という、報酬を計算して `payoff` に格納するメソッドを実装します。

まず、報酬額は次のようになります。

- 独裁者役: 初期保有量 - 受け手への配分額
- 受け手役: 受け手への配分額

初期保有量は `C.INITIAL_ENDOWMENTS` を参照します。
受け手への配分額は、所属しているグループの `allocated_amount` を参照します。
`Player` のインスタンスには `group` プロパティがあり、これを使って所属しているグループの `Group` インスタンスにアクセスできます。

コードは以下のようになります。

```python { title="dictator/models.py" }

class Player(BasePlayer):

    def set_payoff(self):
        allocated_amount = self.group.allocated_amount
        if self.role == C.DICTATOR_ROLE:
            self.payoff = C.INITIAL_ENDOWMENTS - allocated_amount

        else:
            self.payoff = allocated_amount

```

## まとめ

このステップで実装した内容を整理します。

### 設定したパラメータ

| パラメータ | 値 | 定義先 |
|-----------|-----|--------|
| グループサイズ | 2 | `C.PLAYERS_PER_GROUP` |
| 役割の名前 | 独裁者、受け手 | `C.DICTATOR_ROLE`, `C.RECEIVER_ROLE` |
| 初期保有量 | 10 | `C.INITIAL_ENDOWMENTS` |
| 配分額の範囲 | 0〜10 | `Group.allocated_amount` の `min`, `max` |

### 追加したフィールド

| 内容 | フィールド名 | 型 | 格納先 |
|-----|-------------|---|-------|
| 配分額 | `allocated_amount` | `IntegerField`（整数） | `Group` |

### 実装したメソッド

| 内容 | 実装先 | 備考 |
|------|--------|------|
| グループ分け | - | デフォルト動作を使用 |
| 役割割り当て | - | デフォルト動作を使用 |
| 報酬計算 | `Player.set_payoff()` | |

{{% stepcode url="https://github.com/takahiromiura/oTree_instruction_program/tree/tutorial/03-models/tutorial/dictator-game" %}}
GitHub でコードを確認
{{% /stepcode %}}
