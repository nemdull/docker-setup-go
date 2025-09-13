# docker-setup-go

本リポジトリは、Go (Golang) を用いたシンプルなバックエンドと、MySQL を用いたデータベース連携のデモを示すリポジトリです。Docker Compose で MySQL を起動する構成を含み、トランザクションを用いた基本的なDB操作の流れを実装しています。

## 技術スタック

- 言語: Go (Go 1.20 相当)
- フレームワーク/ライブラリ:
  - gin (v1.8.1)
  - gorm (v1.25.0) / sqlite ドライバ (v1.5.0)
  - PostgreSQL ドライバ: pgx/v5
  - MySQL ドライバ: go-sql-driver/mysql
- データベース: MySQL 5.7 (Docker Compose で起動)
- コンテナ/デプロイ:
  - Docker
  - Docker Compose
- 補助ファイル: createTable.sql, insertData.sql
- リポジトリ配置のヒント:
  - main.go に MySQL へ接続してトランザクション処理を行うデモコード
  - docker-compose.yml に MySQL コンテナ定義と初期設定が記載

## 主な機能

- デモ用データベース接続とトランザクション処理の実装
  - article_id = 1 のレコードの「nice」値を取得し、1 増算して更新
  - トランザクション (BEGIN / COMMIT / ROLLBACK) の基本的な使い方を実装
- 初期データの挿入・テーブル作成の参考コードが用意されている
  - insertData.sql, createTable.sql を参照可能

## 設計・実装の工夫

- トランザクションの基本パターン:
  - tx := db.Begin() で開始し、QueryRow で値を取得、再度 tx.Exec で更新、最終的に tx.Commit() で確定。エラー時は tx.Rollback()。
- エラーハンドリングの初歩的実装:
  - DB 接続・トランザクション開始・クエリ・更新時にエラーを検知し、適切に処理を中断してロールバックを試みる設計の骨子を提示。
- コメントの活用:
  - main.go 内のコメントには、将来的なデータモデルの利用や挿入処理のサンプルが用意されている形跡あり。

注意点:
- go.mod で示されている技術スタックと main.go の実装間に不整合がある可能性があります（例: main.go で mysql-driver の使用が見られる一方、go.mod には MySQL ドライバが明示されていません）。実運用時は依存関係の整合性を再確認してください。

## セットアップ & 動作確認方法

ローカル環境での実行:
- 事前に Go の環境設定が必要
  - go mod download / go mod tidy
- main.go の実行
  - go run main.go
  - 127.0.0.1:3306 に接続する MySQL が必要（後述の Docker で起動可能）

Docker を用いたセットアップ:
- MySQL を起動する Docker Compose の手順
  - docker compose up -d
  - docker-compose up -d
- 環境変数は docker-compose.yml 内の ROOTUSER, ROOTPASS, DATABASE, USERNAME, USERPASS に対応
- ローカルでの実行と同様に main.go を実行して動作を確認

動作確認の例:
- 既存データべースとテーブルが用意されていれば、main.go の処理を通じて
  - 指定 article_id の nicenum を取得
  - nicenum を 1 増やして更新
  - トランザクションが正しく完了することを確認

ファイルの関係性:
- createTable.sql / insertData.sql はデモ用の初期データ作成に用意
- docker-compose.yml は MySQL 5.7 の起動設定を提供

推奨の検証手順:
- MySQL 側で articles テーブルと記事データが存在することを確認
- main.go を実行して、nicenum+1 の更新が反映されることを確認
- 失敗時の挙動（エラーハンドリング、ロールバック）が適切かを確認

追加セットアップ手順（Docker Compose 実行例）
- Docker が動作していることを前提とします。
- 実行:
  - docker-compose up -d
  - docker-compose ps
  - docker-compose logs -f mysql
- クイック接続テスト（任意の MySQL クライアントを使用）
  - mysql -h 127.0.0.1 -P 3306 -u $USERNAME -p$USERPASS -D $DATABASE
- 停止:
  - docker-compose down

補足:
- 上記の Docker セットアップ手順は、本リポジトリの docker-compose.yml の構成を前提にしています。
- 実行時には環境変数 ROOTUSER, ROOTPASS, DATABASE, USERNAME, USERPASS の値を適切に設定してください。

## 改善ポイント / TODO

- テスト:
  - ユニットテストの未実装箇所があるため、データベースアクセスをモック化したテストの追加を検討
- エラーハンドリング:
  - db.Open のエラー時の早期リターンを強化（現在はエラー出力のみで処理継続の可能性があるため、Fatal などで停止を検討）
  - rows.Err() のチェック場所の見直しと、RowScan の失敗ケースの詳細エラーログ追加
- 設計:
  - article_id のハードコードをパラメータ化する
  - DB 接続情報を設定ファイルまたは環境変数で管理する
  - 複数のDB接続先に対応する抽象化を検討
- ドキュメント:
  - 実行手順・設定例を README に追記
- CI/CD:
  - lint/tidy のジョブ追加・有効化、GitHub Actions 等の CI 設定の整備

強調したいポイント:
- トランザクションを用いたデータベース操作の基礎を実装している点
- Go でのシンプルな DB 操作の実装パターンと、Docker での環境再現性の確保
