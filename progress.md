# Claude Code 進捗ログ

## 2025-05-02: 初期セットアップ完了

### やったこと
1. Claude Codeをネイティブインストーラーでインストール（`irm https://claude.ai/install.ps1 | iex`）
2. PATHが通っていなかったのでPowerShellで環境変数を手動追加して解決
3. PowerShell再起動後 `claude --version` でv2.1.126確認
4. `claude` で初回起動 → ブラウザ認証 → ログイン成功
5. テーマ: Dark mode選択
6. 起動確認: Opus 4.7 / Claude Max / 1Mコンテキスト

### 解決したトラブル
- `claude --version` で「用語 'claude' は認識されません」エラー
  - 原因: `C:\Users\<ユーザー名>\.local\bin` がPATHに入っていなかった
  - 解決: PowerShellで `[Environment]::SetEnvironmentVariable` を使ってUser PATHに追加 → ターミナル再起動

### 学んだ基本操作
- `claude` → 起動
- `Escape` または `Ctrl+C 2回` または `/exit` → 終了
- `/help` → コマンド一覧
- `/model` → モデル切り替え
- `/init` → CLAUDE.md自動生成（未実施）
- プロジェクトフォルダに `cd` してから `claude` を起動するのが基本の流れ

---

## 2026-05-03: GitHub MCP連携 + Git基礎 + GitHubアップロードまで完走

### やったこと

#### 1. GitHub MCP連携
- GitHubのPersonal Access Token（PAT）を発行
  - 権限: `repo` + `workflow` + `read:org`
  - `workflow`にチェックを入れると`repo`配下が灰色で自動有効化される仕様を確認
- PowerShellでMCPサーバー登録:
  ```powershell
  claude mcp add -s user --transport http github https://api.githubcopilot.com/mcp -H 'Authorization: Bearer YOUR_PAT'
  ```
- `claude mcp list` で接続確認 → ✅
- 自然言語で「私のリポジトリ一覧を見せて」→ MCP経由でGitHubにアクセス成功

#### 2. リポジトリ作成 + HTML生成
- `D:\dev\my-first-repo` を作業フォルダとして作成
  - C:ドライブ圧迫を避け、開発専用に `D:\dev` 配下にまとめる方針
- `git init` でリポジトリ化
- Claude Codeに「自己紹介HTMLを作って」と依頼 → index.html生成
- ブラウザで表示確認 → 修正依頼 → F5で反映確認の流れを習得

#### 3. Git基本サイクルの習得
作業フォルダ → ステージング → リポジトリ の3段階を理解：

```
ファイル編集 → git add → git commit → 履歴に確定
```

実際にコミット3回実行：
- `94ff713` 初回コミット: 自己紹介ページを追加
- `7f3db2e` お問い合わせセクションを追加
- `ee07dec` 学習進捗ログを追加: GitHub MCP連携とGit基礎

#### 4. GitHubへのアップロード
- Claude Code（MCP経由）にGitHubリポジトリを作成依頼
  - 名前: `my-first-repo`、Private、空っぽの状態で作成
- Claude Codeが自動で以下を実行:
  - `git remote add origin https://github.com/iac-cmyk/my-first-repo.git`
  - `git branch -M main`（master → main にリネーム）
  - `git push -u origin main`
- Git Credential Managerで初回認証（ブラウザ経由でGitHubサイン）
- pushに成功し、ローカル↔GitHubの紐付け完了
- リポジトリURL: https://github.com/iac-cmyk/my-first-repo

### 解決したトラブル

#### トラブル1: PowerShellでBearerヘッダエラー
- `claude mcp add` で `Invalid header format: "Bearer"` エラー
- 原因: PowerShellのダブルクォート内の `:` や空白の解釈問題
- 解決: ダブルクォートをシングルクォートに変更

#### トラブル2: progress.md のMarkdownエスケープ事故
- チャットからコピペした長文Markdownがメモ帳に貼り付けた際、`\#` `\-` などのバックスラッシュエスケープが大量混入
- Claude Code自身が「エスケープ記号が混じっている、戻しますか？」と検知・提案してくれた
- 教訓: 長文Markdownはチャットコピペでなく、ファイルダウンロード経由で配置すべき

#### トラブル3: PowerShellの文字化け（type コマンド）
- `type progress.md` で日本語が文字化け
- 原因: PowerShell 5.1の `type` がデフォルトでUTF-8を読まない
- 解決: `Get-Content -Encoding UTF8` を使う、またはメモ帳で開いて確認
- ファイル自体は正常、表示だけの問題

### 学んだ基本操作

#### Git基本コマンド
- `git --version` → バージョン確認
- `git init` → フォルダをリポジトリ化
- `git status` → 現在の状態確認
- `git add ファイル名` → ステージングに追加
- `git add .` → カレント以下の全変更を追加
- `git commit -m "メッセージ"` → 履歴に確定
- `git log` → 履歴表示（`q`で抜ける）
- `git log --oneline` → 履歴を1行表示
- `git remote add origin URL` → リモートリポジトリを登録
- `git branch -M main` → ブランチ名変更
- `git push -u origin main` → リモートにアップロード（初回は`-u`で追跡設定）

#### Claude Code MCP関連
- `claude mcp add` → MCPサーバー追加
- `claude mcp list` → 接続中のサーバー一覧
- `claude mcp remove 名前` → サーバー削除
- `/mcp` → Claude Code内でMCP状態確認

#### MCPツール許可の選択肢
- `1. Yes` → 今回だけ許可（最初はこれ）
- `2. Yes, allow all edits during this session` → セッション中聞かない
- `3. No` → 拒否

#### PowerShell便利コマンド
- `Get-Content ファイル -Encoding UTF8` → UTF-8で正しく表示
- `clip` → パイプで渡された内容をクリップボードにコピー（例: `Get-Content xx | clip`）
- `move -Force 元 先` → 既存ファイルを上書き移動

### 学んだ概念

#### リポジトリとは
- 「変更履歴がすべて記録される特別なフォルダ」
- 普通のフォルダと違って、過去の状態にいつでも戻せる
- クラウド（GitHub）にバックアップ・共有できる

#### MCP（Model Context Protocol）とは
- 外部サービスをClaude Codeに接続して、自然言語で操作できる仕組み
- GitHub以外にも、Slack、Notion、Google Drive、Figmaなど多数連携可能

#### Gitの3段階
```
作業フォルダ          ステージング          リポジトリ
（編集中）    →      （次に保存する候補）  →  （履歴に確定）
              git add                      git commit
```

#### ローカルとリモートの関係
```
ローカル                          GitHub（クラウド）
D:\dev\my-first-repo  ←リンク→  https://github.com/iac-cmyk/my-first-repo
                      git push  →  コミット履歴を送信
                      git pull  ←  更新を取得
```

#### GitHub上での見え方
- `progress.md`（Markdown）→ 自動で整形されて綺麗に表示
- `index.html` → ソースコードとして表示（ブラウザ描画されない）
- HTMLをブラウザで描画して見たい → ローカルでダブルクリック、またはGitHub Pages（Public化が必要）

#### セキュリティの基本
- PATが漏れると、付与した権限の範囲で何でもできる鍵になる
- 漏洩リスク: コード改ざん、リポジトリ削除、Actions悪用、二次被害など
- 対策: コードに直接書かない、期限短め、最小権限、漏れたら即削除&再発行

### 今日の総合的な学び

- **トラブルも貴重な学び**: クォート問題、エスケープ事故、文字化けなど、実践でしか出会えないトラブルに対処できた
- **Claude Codeの自律性**: 「エスケープが混じっている」と自分で気づいて報告するなど、ただの実行ツールではない判断力がある
- **長文転送の最適解**: チャットコピペは事故が起こりやすい。ファイルダウンロード方式が確実
- **Privateリポジトリで練習**: 公開前提でなくても、バージョン管理と履歴の恩恵は十分受けられる

### まだやっていないこと
- ブランチ操作（branch / merge）
- 過去状態への復元（reset / revert）
- `git pull` での更新取得
- `git clone` で別PCに展開
- `.gitignore` ファイル設定
- `/init` によるCLAUDE.md作成
- 他のMCP連携（Slack、Notion、Google Drive等）
- GitHub Pagesでの公開

### 次回以降のおすすめ順序
1. **ブランチ操作**（branch / merge）← 複数の作業を並行管理する基礎
2. **過去状態への復元**（reset / revert）← ミスから戻る方法
3. **`.gitignore`** ← Gitに含めたくないファイルの除外（パスワードファイルなど）
4. **`/init` でCLAUDE.md作成** ← プロジェクトルールをClaude Codeに記憶させる
5. **他のMCP連携** ← 実用的なワークフロー構築

### 現在のリポジトリ状態
- ローカル: `D:\dev\my-first-repo`
- リモート: `https://github.com/iac-cmyk/my-first-repo` (Private)
- ブランチ: `main`
- ファイル: `index.html`, `progress.md`
- コミット数: 3

---

## 2026-05-06: ブランチ操作の基本サイクルを習得

### やったこと

#### 1. ブランチ作成と切り替え
- `git branch` で現状確認 → `main` のみ
- `git checkout -b feature/add-skills` で作成と同時に切り替え
- ブランチ名は `feature/xxx` のようにスラッシュで分類するのが慣習

#### 2. ブランチ上で作業
- index.htmlに「私のスキル」セクションを追加（HTML/CSS、Git、Claude Code）
- `git add . && git commit -m "スキルセクションを追加"` でコミット（dbfba8d）

#### 3. main への merge
- `git checkout main` で main に戻る
- `git merge feature/add-skills` で合流
- 結果: **Fast-forward マージ**（main が分岐後に進んでいなかったため、新規合流コミットなし）

#### 4. 後片付けとリモート反映
- `git branch -d feature/add-skills` で merge 済みブランチを安全削除
- `git push` でリモート（GitHub）に反映

### 解決したトラブル

#### トラブル1: checkout がブロックされる
- `.claude/settings.local.json`（Claude Codeが書き込む権限設定ファイル）に未コミットの変更があり、`git checkout main` が失敗
- エラー: `Your local changes to the following files would be overwritten by checkout`
- 解決: 一旦追加コミットして切り替え

#### トラブル2: merge 時に未追跡ファイルと衝突
- main に戻った時、`.claude/settings.local.json` が「Git管理外のファイル」として作業フォルダに残った
- feature 側にはコミット済みの同名ファイルがあるため、merge で上書きすることになり Git が停止
- エラー: `untracked working tree files would be overwritten by merge`
- 解決: `Remove-Item .claude\settings.local.json -Force; git merge feature/add-skills` で削除してから merge
- → これは `.gitignore` で最初から除外しておけば起きない問題（次回テーマに直結）

### 学んだ基本操作

#### Gitブランチ系コマンド
- `git branch` → ブランチ一覧（`*` が現在地）
- `git checkout -b 名前` → 作成して切り替え（2ステップを1回で）
- `git checkout 名前` → 既存ブランチに切り替え
- `git merge 名前` → 現在のブランチに指定ブランチを合流
- `git branch -d 名前` → merge済みブランチを安全削除（小文字d）
- `git branch -D 名前` → 強制削除（大文字D、未mergeでも消す。危険）

#### ブランチ命名の慣習
- `feature/add-skills` のように `カテゴリ/内容` 形式
- スラッシュは Git 内部でフォルダのように扱われ、整理しやすい
- 他の慣習例: `fix/...`（バグ修正）、`docs/...`（ドキュメント）、`refactor/...`

### 学んだ概念

#### ブランチとは
- 「作業線の分岐」
- main を本線、feature を実験線として使い分ける
- 失敗しても main には影響ゼロ

#### Fast-forward マージ
分岐後に main が進んでいない場合、Gitは合流コミットを作らず main のラベルを feature の最新まで「早送り」するだけ：

```
マージ前:
main:           A → B → C
                          ↘
feature:                   D → E

マージ後 (Fast-forward):
main, feature:  A → B → C → D → E
                              ↑
                         両方ここを指す
```

#### 未追跡ファイルとブランチ切り替えの衝突
- 未追跡（untracked）= Gitが管理していないが、フォルダには存在するファイル
- ブランチを切り替えると同名ファイルがあると衝突する
- 教訓: `.gitignore` で最初から除外すべきファイルを宣言しておくのが正解

### 今日の総合的な学び

- **ブランチワークフローの全体像**: 作成 → 作業 → 切替 → merge → 削除 → push の1サイクルを完走
- **Fast-forward を実体験**: 「合流コミットが作られないパターン」を最初に経験できたのは理解しやすかった
- **`.gitignore` の必要性を体感**: トラブル2で「Claude Codeの設定ファイルが毎回邪魔をする」現象に遭遇。次回 `.gitignore` を設定する動機が明確になった
- **権限プロンプトの理解**: `.claude/settings.local.json` 自体が「Claude Codeの権限許可リスト」だと判明 → 操作のたびに増減する性質を持つので、Git管理外にすべき典型例

### 現在のリポジトリ状態
- ローカル: `D:\dev\my-first-repo`
- リモート: `https://github.com/iac-cmyk/my-first-repo` (Private)
- ブランチ: `main` のみ
- ファイル: `index.html`（スキルセクション追加済）, `progress.md`, `.claude/settings.local.json`（要除外）
- コミット数: 6（main上）

### 次回以降のおすすめ順序（更新版）
1. **`.gitignore` 設定** ← 今日のトラブル根本解決。最優先
2. **過去状態への復元**（reset / revert）← ミスから戻る方法
3. **`/init` でCLAUDE.md作成** ← プロジェクトルールをClaude Codeに記憶させる
4. **マージコンフリクト体験** ← Fast-forwardでない本格的な merge
5. **他のMCP連携** ← Slack、Notion等
