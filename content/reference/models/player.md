+++
title = "Player"
draft = false
+++

Player クラスのリファレンス。

## 属性一覧

| 属性 | 型 | 説明 |
|-----|-----|------|
| `id_in_group` | `int` | グループ内での ID |
| `id_in_subsession` | `int` | サブセッション内での ID |
| `round_number` | `int` | 現在のラウンド数 |
| `payoff` | `Currency` | このラウンドの報酬 |
| `role` | `str` | 役割名 (`C` で定義した場合) |
| `participant` | `Participant` | 所属する Participant |
| `group` | `Group` | 所属する Group |
| `subsession` | `Subsession` | 所属する Subsession |

## メソッド一覧

### 組み込みメソッド

| メソッド | 戻り値 | 説明 |
|---------|-------|------|
| `in_round` | `Player` | 指定ラウンドのインスタンスを取得 |
| `in_rounds` | `List[Player]` | 指定範囲のラウンドのインスタンスを取得 |
| `in_previous_rounds` | `List[Player]` | 現在より前のラウンドのインスタンスを取得 |
| `in_all_rounds` | `List[Player]` | 全ラウンドのインスタンスを取得 |
| `get_others_in_group` | `List[Player]` | 同じグループの他のプレイヤーを取得 |
| `get_others_in_subsession` | `List[Player]` | 同じサブセッションの他のプレイヤーを取得 |

---

## 属性

### id_in_group

{{< api sig="id_in_group" returns="int" >}}
グループ内での ID。

- 1 から始まる連番
- グループ内での順序を表す
- `C` クラスで役割定数を定義している場合、この値に基づいて `role` が決まる
{{< /api >}}

### id_in_subsession

{{< api sig="id_in_subsession" returns="int" >}}
Subsession 内での ID。

- `participant.id_in_session` と同じ値
{{< /api >}}

### round_number

{{< api sig="round_number" returns="int" >}}
現在のラウンド数。

- アプリケーションごとに 1 から振られる
- `subsession.round_number` と同じ値
{{< /api >}}

### payoff

{{< api sig="payoff" returns="Currency" >}}
このラウンドの報酬。

- 値を設定すると、自動的に `participant.payoff` にも加算される
- 初期値は 0

```python
def set_payoffs(group):
    for player in group.get_players():
        player.payoff = cu(100)
```
{{< /api >}}

### role

{{< api sig="role" returns="str" >}}
プレイヤーの役割名。

- 読み取り専用（`_role` 属性への直接代入で変更可能）
- `C` クラスで役割を定義すると、`id_in_group` に基づいて自動的に設定される

```python
class C(BaseConstants):
    ...
    PROPOSER_ROLE = 'proposer'  # id_in_group=1 に割り当て
    RESPONDER_ROLE = 'responder'  # id_in_group=2 に割り当て
```
{{< /api >}}

### participant

{{< api sig="participant" returns="Participant" >}}
このプレイヤーに紐づいている Participant オブジェクト。
{{< /api >}}

### group

{{< api sig="group" returns="Group" >}}
このプレイヤーが所属する Group オブジェクト。
{{< /api >}}

### subsession

{{< api sig="subsession" returns="Subsession" >}}
このプレイヤーが所属する Subsession オブジェクト。
{{< /api >}}

---

## メソッド

### ラウンドのデータ取得

{{% notice style="note" title="取得できるインスタンスの範囲" %}}
取得できるのは、同じアプリ・同じ参加者の Player インスタンスのみです。
{{% /notice %}}

#### in_round

{{< api sig="in_round(round_number)" returns="Player" >}}
指定したラウンドの Player インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `round_number` | int | 取得するラウンド番号 |
{{< /api >}}

#### in_rounds

{{< api sig="in_rounds(first, last)" returns="List[Player]" >}}
指定した範囲のラウンドの Player インスタンスを取得します。

| 引数 | 型 | 説明 |
|------|-----|------|
| `first` | int | 開始ラウンド番号 |
| `last` | int | 終了ラウンド番号 |
{{< /api >}}

#### in_previous_rounds

{{< api sig="in_previous_rounds()" returns="List[Player]" >}}
現在より前のラウンドの Player インスタンスを取得します。

- 現在の Player インスタンスは含まれない
- `self.in_rounds(1, self.round_number - 1)` と同等
{{< /api >}}

#### in_all_rounds

{{< api sig="in_all_rounds()" returns="List[Player]" >}}
現在までの全ラウンドの Player インスタンスを取得します。

- 現在の Player インスタンスも含まれる
- `self.in_rounds(1, self.round_number)` と同等
{{< /api >}}

---

### 他のプレイヤーの取得

#### get_others_in_group

{{< api sig="get_others_in_group()" returns="List[Player]" >}}
同じグループ内の他のプレイヤーを取得します。

- 自分自身は含まれない

```python
# Results ページで他のプレイヤーの貢献量を表示
class Results(Page):
    @staticmethod
    def vars_for_template(player):
        others = player.get_others_in_group()
        return dict(
            others_contributions=[o.contribution for o in others]
        )
```
{{< /api >}}

#### get_others_in_subsession

{{< api sig="get_others_in_subsession()" returns="List[Player]" >}}
同じ Subsession 内の他のプレイヤーを取得します。

- 自分自身は含まれない
- 他のグループのプレイヤーも含まれる
{{< /api >}}

---

## 関連セクション

- [Group]({{% ref "/reference/models/group" %}})
- [Subsession]({{% ref "/reference/models/subsession" %}})
- [Constants]({{% ref "/reference/models/constants" %}})
- [Currency]({{% ref "/reference/currency" %}})
