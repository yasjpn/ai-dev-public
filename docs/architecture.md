# アーキテクチャ図 — 現在の仕組み

2026-11-08 時点の実装状態を反映した構成図です。
方針は「案4: ハイブリッド型（段階昇格型）」。試作は軽い場所で速く、本格開発はこのリポジトリを正本に行います。

## 全体図

```mermaid
flowchart TB
    subgraph IN["入口（実装済み）"]
        U["ユーザー / iPhone<br/>音声入力・チャット・通知の確認<br/>判断と承認（ログイン・二段階認証は本人）"]
    end

    subgraph BRAIN["頭脳（実装済み）"]
        M["Muse VM (Linux / Ubuntu 24.04)<br/>チャット / ファイル / ターミナル / ブラウザ / スキル / cron"]
        MEM["記憶の層<br/>MEMORY.md・日次ログ・人物/ゴール<br/>原典を要約で置き換えない"]
        SUB["委任（部分実装）<br/>サブエージェント並行作業<br/>外部 AI 接続は未確認"]
        M --- MEM
        M --- SUB
    end

    subgraph DEV["AI駆動開発（実装済み）"]
        SP["試作: Muse スペース / VM ワークスペース<br/>プロトタイプ、ダッシュボード、検証"]
        GH[("正本: GitHub リポジトリ<br/>yasjpn/ai-dev-workspace (private)<br/>ブランチ → PR → 承認 → マージ")]
        PB["運用: docs/playbook.md<br/>昇格基準とチェックリスト<br/>AGENTS.md の作業規約"]
        SP -->|"育てると決めたら昇格"| GH
        GH --- PB
    end

    subgraph LINK["連携サービス"]
        G["Google（接続済み）<br/>Gmail / Drive / Sheets / Docs / Slides / Calendar"]
        X["未接続<br/>仕事メール (Outlook) / Slack / Dropbox / 1Password<br/>TimeTree / Obsidian / LINE は専用コネクタなし"]
    end

    subgraph FUTURE["将来（未実装）"]
        PC["手元の PC (Windows)<br/>Tailscale / SSH / Playwright MCP<br/>読み取り橋・書き込み橋"]
        NAS[("ファイルサーバー NAS<br/>原典と成果物の最終保管")]
    end

    U -->|"依頼・追加指示"| M
    M -->|"成果物・要点と URL・承認依頼"| U
    M --> DEV
    M -->|"読み書き（承認制の操作あり）"| G
    M -.->|"未接続"| X
    M -.->|"未接続"| PC
    PC -.-> NAS
```

## 開発サイクル

```mermaid
sequenceDiagram
    participant U as iPhone（依頼・承認）
    participant M as Muse VM
    participant R as GitHub リポジトリ
    U->>M: 要件を音声/チャットで伝える
    M->>M: 仕様を docs/ に起こし作業を分解
    M-->>U: チェックポイント（仕様の確認）
    U->>M: 承認
    M->>R: ブランチを作成して実装
    M->>M: テスト・動作確認
    M->>R: プルリクエストを作成
    M-->>U: 要約・差分・確認結果を報告
    U->>M: マージ承認
    M->>R: マージ
```

## コンポーネントと状態（2026-11-08）

| 区分 | 状態 | 補足 |
|---|---|---|
| iPhone 入口（音声・チャット・承認） | 実装済み | この運用自体がその形で回っている |
| Muse VM (Linux) 本体 | 実装済み | git / Node.js 24 / Python 3.12 を確認 |
| 記憶（MEMORY・日次ログ・ゴール） | 実装済み | 原典主義で運用中 |
| Muse スペース（試作） | 実装済み | 構成図ダッシュボード等で使用実績あり |
| GitHub 連携・専用リポジトリ | 実装済み | OAuth 接続、App の対象選択、読み書きを確認 |
| Google 系 6 サービス | 実装済み | Gmail / Drive / Sheets / Docs / Slides / Calendar |
| 見張り・自動化（業務 cron） | 部分実装 | 基盤はあるが業務用設定は未設定 |
| 別の AI への委任 | 部分実装 | サブエージェントの仕組みのみ |
| CI (GitHub Actions) | 未設定 | 最初のプロジェクトの技術選定と同時に作る |
| 公開先（ホスティング） | 未設定 | 公開するものが生まれた時点で接続 |
| 手元 PC / Tailscale / NAS | 未実装 | 図の「本線」。将来段階 |
| Slack / Dropbox / 1Password / 仕事メール | 未接続 | コネクタは存在する |
| TimeTree / Obsidian / LINE | 未実装 | 専用コネクタなし。代替か新規作成が必要 |

## 承認と安全の境界

- 送信・公開・投稿・削除・支払い・マージは本人の承認を経る
- 資格情報はチャットやログに出さない。ログインと二段階認証は本人が行う
- 秘密情報はリポジトリにコミットしない
- 読み取りは広く、書き込みは狭く（ブランチ + PR 経由、`main` 直プッシュ禁止）
