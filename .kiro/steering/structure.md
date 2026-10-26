---
inclusion: always
---

# フォルダの使い方

| フォルダ | 置くもの | 変更してよいか |
|---|---|---|
| `docs/` | 手順書 | 変更しない |
| `materials/` | ハンズオン用のサンプル資料 | 変更しない |
| `exercises/` | 演習のお題 | 変更しない |
| `examples/` | 完成見本 | 変更しない |
| `participants/＜名前＞/` | 利用者の成果物（HTML など） | 本人のフォルダだけ変更してよい |
| `.kiro/specs/＜Spec名＞/` | Spec の文書（requirements.md / design.md / tasks.md） | 本人の Spec だけ変更してよい |
| `.kiro/steering/` | Steering | 共通の4ファイルは変更しない。個人用は `memo-＜名前＞.md` |
| `.kiro/skills/＜スキル名＞/` | スキル | `minutes-summary` は変更しない。利用者が作るスキルは名前に `-＜名前＞` を付ける |

- 利用者の成果物は、必ず **`participants/＜名前＞/`** に置いてください。`＜名前＞` は今のブランチ名 `practice/＜名前＞` から判断し、分からなければ利用者に確認してください
- 利用者本人以外の名前のフォルダや Spec は変更しないでください（仲間と一緒に使う場合もあるため）
