+++
title = "Group"
draft = false
+++

Group クラスのリファレンスです。

## 属性一覧

| 属性 | 型 | 説明 |
|-----|-----|------|
| `id_in_subsession` | `int` | Subsession 内でのグループ ID |
| `round_number` | `int` | 現在のラウンド数 |
| `subsession` | `Subsession` | 所属する Subsession |

## メソッド一覧

### 組み込みメソッド

| メソッド | 戻り値 | 説明 |
|---------|-------|------|
| `get_players` | `List[Player]` | グループ内の全プレイヤーを取得 |
| `get_player_by_id` | `Player` | ID でプレイヤーを取得 |
| `get_player_by_role` | `Player` | 役割名でプレイヤーを取得 |
| `set_players` | `None` | グループのメンバーを設定 |
| `in_round` | `Group` | 指定ラウンドのインスタンスを取得 |
| `in_rounds` | `List[Group]` | 指定範囲のラウンドのインスタンスを取得 |
| `in_previous_rounds` | `List[Group]` | 現在より前のラウンドのインスタンスを取得 |
| `in_all_rounds` | `List[Group]` | 全ラウンドのインスタンスを取得 |

---

## 属性

### id_in_subsession

{{< api sig="id_in_subsession: int" >}}
Subsession 内でのグループ ID。

- 1 から始まる連番
{{< /api >}}

### round_number

{{< api sig="round_number: int" >}}
現在のラウンド数。

- `subsession.round_number` と同じ値
{{< /api >}}

### subsession

{{< api sig="subsession: Subsession" >}}
このグループが所属する Subsession オブジェクト。
{{< /api >}}

---

## メソッド

### プレイヤーの取得

#### get_players

{{< api sig="get_players()" returns="List[Player]" >}}
グループ内の全プレイヤーを取得します。

- `id_in_group` 順にソートされている
{{< /api >}}

#### get_player_by_id

{{< api sig="get_player_by_id(id_in_group)" returns="Player" >}}
指定した `id_in_group` のプレイヤーを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `id_in_group` | int | 取得するプレイヤーのグループ内 ID |

- 該当するプレイヤーが存在しない場合は `ValueError` を送出

```python
class Group(BaseGroup):
    def set_payoffs(self):
        # id_in_group=1 が独裁者、id_in_group=2 が受領者
        dictator = self.get_player_by_id(1)
        receiver = self.get_player_by_id(2)
        dictator.payoff = C.ENDOWMENT - dictator.sent_amount
        receiver.payoff = dictator.sent_amount
```
{{< /api >}}

#### get_player_by_role

{{< api sig="get_player_by_role(role)" returns="Player" >}}
指定した役割のプレイヤーを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `role` | str | 取得するプレイヤーの役割名 |

- `C` クラスで役割定数を定義している場合に使用
- 該当するプレイヤーが存在しない場合は `ValueError` を送出

```python
class Group(BaseGroup):
    def set_payoffs(self):
        # C.DICTATOR_ROLE = 'dictator' などと定義している場合
        dictator = self.get_player_by_role('dictator')
        receiver = self.get_player_by_role('receiver')
        dictator.payoff = C.ENDOWMENT - dictator.sent_amount
        receiver.payoff = dictator.sent_amount
```
{{< /api >}}


---

### メンバーの設定

#### set_players

{{< api sig="set_players(players_list)" >}}
グループのメンバーを設定します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `players_list` | List[Player] | グループに設定するプレイヤーのリスト |

- リストの順番が `id_in_group` になる
- `C` クラスで役割定数を定義している場合、順番に応じて役割も自動的に再設定される

```python
class Group(BaseGroup):
    def swap_players(self):
        # グループ内のメンバーの順番を入れ替える
        players = self.get_players()
        self.set_players([players[2], players[0], players[1]])
```
{{< /api >}}

{{% notice style="tip" title="Subsession のメソッドとの違い" %}}
`set_players` はグループ内の順番 (`id_in_group`) を入れ替えるのに適しています。
グループ間でプレイヤーを移動させたい場合は、Subsession のメソッド（`set_group_matrix` など）を使うことをオススメします。
詳細は [Subsession クラスリファレンス]({{% ref "/reference/models/subsession" %}}) を参照してください。
{{% /notice %}}

---

### ラウンドのデータ取得

{{% notice style="note" title="取得できるインスタンスの範囲" %}}
取得できるのは、同じアプリ・同じグループ ID (`id_in_subsession`) の Group インスタンスのみです。
ラウンド間でグループ構成を変更すると、同じグループ ID でもメンバーが異なるため、意図した結果にならない可能性があります。
{{% /notice %}}

#### in_round

{{< api sig="in_round(round_number)" returns="Group" >}}
指定したラウンドの Group インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `round_number` | int | 取得するラウンド番号 |
{{< /api >}}

#### in_rounds

{{< api sig="in_rounds(first, last)" returns="List[Group]" >}}
指定した範囲のラウンドの Group インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `first` | int | 開始ラウンド番号 |
| `last` | int | 終了ラウンド番号 |
{{< /api >}}

#### in_previous_rounds

{{< api sig="in_previous_rounds()" returns="List[Group]" >}}
現在より前のラウンドの Group インスタンスを取得します。

- 現在の Group インスタンスは含まれない
- `self.in_rounds(1, self.round_number - 1)` と同等
{{< /api >}}

#### in_all_rounds

{{< api sig="in_all_rounds()" returns="List[Group]" >}}
現在までの全ラウンドの Group インスタンスを取得します。

- 現在の Group インスタンスも含まれる
- `self.in_rounds(1, self.round_number)` と同等
{{< /api >}}

---

## 関連セクション

- [Player]({{% ref "/reference/models/player" %}})
- [Subsession]({{% ref "/reference/models/subsession" %}})
- [Constants]({{% ref "/reference/models/constants" %}})
- [Payoff]({{% ref "/reference/payoff" %}})
