+++
draft = false
title = 'oTree の概念'
weight = 2
math = true
+++

前ページで説明した実験用語と oTree の概念との対応を説明します。

## 対応表

| 実験用語 | oTree の概念 |
|----------|-------------|
| セッション | Session |
| パート | App |
| ラウンド | Subsession |
| ステージ | Page |
| 参加者 | Participant |
| グループ | Group |
| プレイヤー | Player |

## 実験の区切りとの対応

### セッション → Session

oTree での実験の一番大きな区切りも Session です。
Session では、参加者は設定を共有します。

### パート → App

oTree はセッション内の一連のタスクを **アプリケーション** (App) という単位で区切ります。

### ラウンド → Subsession

oTree ではラウンドに相当するものは **サブセッション** (Subsession) です。

### ステージ → Page

oTree では、ステージに対応するものは **ページ** (Page) で、実験で参加者に表示される画面の単位を表します。
基本的には、ステージよりもページの方が多くなります。
例えば、独裁者ゲームはより細かい手順に分割できます。

1. ゲームの説明
2. プレイヤーのロールの提示
3. 独裁者の選択 (独裁者のみ表示)
4. 結果の表示

したがって、この例では 4 つのページが必要になります。

## 参加者に関する用語との対応

参加者 (Participant), グループ (Group), プレイヤー (Player) は oTree でも同じように考えても問題ありません。

## oTree の入れ子構造

### 参加者の構造

Session, App, Subsession, Group, Player は以下のような入れ子構造になっています。

```
Session
├── App 1 (dictator_game)
│   ├── Subsession (ラウンド 1)
│   │   ├── Group 1
│   │   │   ├── Player 1 (Participant A)
│   │   │   └── Player 2 (Participant B)
│   │   └── Group 2
│   │       ├── Player 3 (Participant C)
│   │       └── Player 4 (Participant D)
│   └── Subsession (ラウンド 2)
│       └── ...
└── ...
```

### 画面フローの構造

Session, App, Page は以下のような入れ子構造になっています。

```
Session
├── App 1 (dictator_game)
│   ├── Page 1 (Instruction)
│   ├── Page 2 (Role)
│   ├── Page 3 (Offer)
│   └── Page 4 (Results)
├── App 2 (ultimatum_game)
│   ├── Page 1 (Instruction)
│   ├── Page 2 (Offer)
│   ├── Page 3 (Response)
│   └── Page 4 (Results)
└── App 3 (survey)
    ├── Page 1 (Demographics)
    └── Page 2 (Feedback)
```
