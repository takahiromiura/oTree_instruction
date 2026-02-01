+++
title = "Subsession"
draft = false
+++

Subsession クラスのリファレンスです。

## 属性一覧

| 属性 | 型 | 説明 |
|-----|-----|------|
| `round_number` | `int` | 現在のラウンド数 (アプリごとに 1 から) |

## メソッド一覧

### 組み込みメソッド

| メソッド | 戻り値 | 説明 |
|---------|-------|------|
| `in_round` | `Subsession` | 指定ラウンドのインスタンスを取得 |
| `in_rounds` | `List[Subsession]` | 指定したラウンド数の範囲のインスタンスを取得 |
| `in_previous_rounds` | `List[Subsession]` | 現在より前までのラウンドのインスタンスを取得 |
| `in_all_rounds` | `List[Subsession]` | 全ラウンドのインスタンスを取得 |
| `get_players` | `List[Player]` | 全プレイヤーを取得 |
| `get_groups` | `List[Group]` | 全グループを取得 |
| `get_group_matrix` | `List[List]` | グループ構成を取得 |
| `set_group_matrix` | `None` | グループ構成を設定 |
| `group_like_round` | `None` | 指定ラウンドと同じグループ構成にする |
| `group_randomly` | `None` | グループをランダムにシャッフル |

### オーバーライド可能なメソッド

必要に応じて、実験者が実装するメソッドです。

| メソッド | 戻り値 | 説明 |
|---------|-------|------|
| `creating_session` | `None` | 実験開始時に実行される初期化関数 |
| `group_by_arrival_time_method` | `List[Player]` | 到着順グルーピングのルール定義 |

---

## 属性

### round_number

{{< api sig="round_number: int" >}}
現在のラウンド数を返します。

- アプリケーションごとに 1 から振られる
- 例: 2 ラウンド制のアプリでは、1 ラウンド目は `1`、2 ラウンド目は `2`
{{< /api >}}

---

## メソッド

#### creating_session

{{< api sig="creating_session()" >}}
実験開始時に実行される初期化関数です。
グループや役割の割り当て方法を変えたい場合に定義します。

- 省略可能
- 実験開始時に、全アプリ・全ラウンドの `creating_session` が順番に実行される

{{% details title="実行順序の例" %}}
Trust Game (2 ラウンド) → Ultimatum Game (1 ラウンド) の場合:

1. Trust Game 1 ラウンド目の `creating_session`
2. Trust Game 2 ラウンド目の `creating_session`
3. Ultimatum Game 1 ラウンド目の `creating_session`
{{% /details %}}

{{% notice style="warning" title="動的な割り当てには使えない" %}}
`creating_session` は実験開始時に全て実行され、以降は実行されません。
そのため、選択に応じたグループや役割の割り当てはこのメソッドでは実現できません。
これを行うには、`Page` クラスの `before_next_page` や `WaitPage` の `after_all_players_arrive` を使用してください。
{{% /notice %}}

**実装例:**

```python
def creating_session(self):
    # 各ラウンドでグループをランダムに割り当て
    self.group_randomly()
```
{{< /api >}}

---

### ラウンドのデータ取得

別ラウンドの Subsession インスタンスを取得できます。

{{% notice style="note" title="同じアプリのみ取得可能" %}}
取得できるのは、同じアプリの Subsession インスタンスのみです。
{{% /notice %}}

#### in_round

{{< api sig="in_round(round_number)" returns="Subsession" >}}
指定したラウンドの Subsession インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `round_number` | int | 取得するラウンド番号 |
{{< /api >}}

#### in_rounds

{{< api sig="in_rounds(first, last)" returns="List[Subsession]" >}}
指定した範囲のラウンドの Subsession インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `first` | int | 開始ラウンド番号 |
| `last` | int | 終了ラウンド番号 |
{{< /api >}}

#### in_previous_rounds

{{< api sig="in_previous_rounds()" returns="List[Subsession]" >}}
現在より前のラウンドの Subsession インスタンスを取得します。

- 現在の Subsession インスタンスは含まれない
- `self.in_rounds(1, self.round_number - 1)` と同等
{{< /api >}}

#### in_all_rounds

{{< api sig="in_all_rounds()" returns="List[Subsession]" >}}
現在までの全ラウンドの Subsession インスタンスを取得します。

- 現在の Subsession インスタンスも含まれる
- `self.in_rounds(1, self.round_number)` と同等
{{< /api >}}

---

### プレイヤー・グループの取得

#### get_players

{{< api sig="get_players()" returns="List[Player]" >}}
現在のラウンドの全プレイヤーを取得します。

- Subsession 内のプレイヤー ID 順にソートされている
{{< /api >}}

#### get_groups

{{< api sig="get_groups()" returns="List[Group]" >}}
現在のラウンドの全グループを取得します。

- Subsession 内のグループ ID 順にソートされている
{{< /api >}}

---

### グループ構成の操作

#### get_group_matrix

{{< api sig="get_group_matrix()" returns="List[List[Player]]" >}}
グループ構成を取得します。

- グループごとのプレイヤーのリストを返す
- グループ ID 順にソート
- 各グループ内のプレイヤーはグループ内 ID 順にソート
{{< /api >}}

#### set_group_matrix

{{< api sig="set_group_matrix(matrix)" >}}
グループ構成を設定します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `matrix` | List[List[Player \| int]] | グループ構成を表す二次元リスト |

- グループ ID 順にメンバーを指定
- Player インスタンスまたはプレイヤー ID で指定可能

```python
def creating_session(self):
    # プレイヤー ID でグループを指定
    self.set_group_matrix([[1, 2], [3, 4]])
```
{{< /api >}}

#### group_like_round

{{< api sig="group_like_round(round_number)" >}}
指定したラウンドと同じグループ構成にします。

| 引数 | 型 | 説明 |
|------|-----|------|
| `round_number` | int | コピー元のラウンド番号 |

```python
def creating_session(self):
    if self.round_number > 1:
        # 1 ラウンド目と同じグループ構成を維持
        self.group_like_round(1)
```
{{< /api >}}

#### group_randomly

{{< api sig="group_randomly(*, fixed_id_in_group=False)" >}}
グループ構成をランダムにシャッフルします。

| 引数 | 型 | 説明 |
|------|-----|------|
| `fixed_id_in_group` | bool | グループ内 ID を固定するか (デフォルト: `False`) |

- `fixed_id_in_group=False`: グループもメンバーもシャッフル
- `fixed_id_in_group=True`: グループ内 ID は固定し、グループ間でプレイヤーをシャッフル
{{< /api >}}

---


#### group_by_arrival_time_method

{{< api sig="group_by_arrival_time_method(waiting_players)" returns="List[Player] | None" >}}
到着順グルーピングで、より細かいルールを定義します。


| 引数 | 型 | 説明 |
|------|-----|------|
| `waiting_players` | List[Player] | WaitPage で待機中のプレイヤーリスト (到着順) |

- WaitPage で `group_by_arrival_time = True` を設定した場合に使用可能
- グループを形成するプレイヤーのリストを返す、または `None` (まだグループを形成しない場合)

{{% notice style="note" title="実行タイミング" %}}
このメソッドは新しいプレイヤーが WaitPage に到着するたびに実行されます。

到着順に 2 人グループを作成する場合の例:

1. プレイヤー A が到着 → `group_by_arrival_time_method([A])` が呼ばれる
2. プレイヤー B が到着 → `group_by_arrival_time_method([A, B])` が呼ばれる
3. プレイヤー C が到着 → `group_by_arrival_time_method([C])` が呼ばれる

プレイヤーが接続を切断したり、タイムアウトで除外されても、それ自体では再実行されません。
{{% /notice %}}

**実装例:**

```python
def group_by_arrival_time_method(self, waiting_players):
    # 3 人揃ったらグループを形成
    if len(waiting_players) >= 3:
        return waiting_players[:3]
```
{{< /api >}}

{{% notice style="note" title="グループ内 ID、役割の割り当て" %}}

返したリストの順番がそのままプレイヤーのグループ内 ID (`id_in_group`) になります。

- リストの 1 番目 → `id_in_group = 1`
- リストの 2 番目 → `id_in_group = 2`
- ...

`C` クラスで役割名を定義している場合は、`id_in_group` の順に役割が割り当てられます。

役割を到着順でランダムにしたい場合はプレイヤーの順番をシャッフルしてから返します。
{{% /notice %}}

---

## 関連セクション

- [Player]({{% ref "/reference/models/player" %}}) - Player クラスリファレンス
- [Group]({{% ref "/reference/models/group" %}}) - Group クラスリファレンス
- [WaitPage]({{% ref "/reference/pages/waitpage" %}}) - WaitPage クラスリファレンス
- [ガイド: グループ割り当て]({{% ref "/guide/group-assignment" %}}) - グループ構成の実践的な操作方法
