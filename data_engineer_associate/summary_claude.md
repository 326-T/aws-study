# AWS Data Engineer Associate チートシート

---

## Amazon Redshift

### 基本
- **Spectrum**: S3上のデータを直接クエリ（たまに使うデータ向け、従量課金）
- **COPY**: S3からロード。**SSE-KMS対応、SSE-C非対応、XML非対応**
- **UNLOAD**: RedshiftからS3へエクスポート

### 分散スタイル
| スタイル | 配置 | 用途 |
|---------|------|------|
| KEY | 共通キーを同じノード | 巨大テーブル結合 |
| ALL | 全ノードにコピー | 小さいマスタ |
| EVEN | 均等配布 | 結合しないテーブル |

### その他
- **ステージングテーブル**: 重複排除/UPSERT時に使う
- **動的データマスキング(DDM)**: ロール別にマスキング
- **Data Sharing**: リージョン/アカウント間でデータ共有可能
- **フェデレーテッドクエリ**: Aurora/RDS(PostgreSQL, MySQL)のライブデータをクエリ
- **ネイティブIdPフェデレーション**: 対応
- **暗号化**: 再作成不要で有効化可、KMSローテ可
- **監査**: ネイティブ監査機能 + CloudTrail
- **クエリエディタv2**: ストアドプロシージャの定期実行可
- **SUPER + PartiQL**: ネストJSONをそのまま取り込み・クエリ
- **ストアドプロシージャ実行**: `CALL procedure_name(args)`

---

## Amazon Kinesis

### Data Streams
- **S3直接書込み不可** → Firehose経由
- **順序保持**: シャード内のみ保証
- **パーティションキー**: ランダム=最大スループット、グループ別=順序保持

### スループット最適化
| 状況 | 対策 |
|------|------|
| 全体流量が限界 | シャード増加 |
| 1シャード内渋滞 | 並列化係数(max 10) |
| 読取制限(429) | 拡張ファンアウト |

- **書込み不足** → シャード増 or KPL
- **読出し不足** → 拡張ファンアウト or 並列化係数

### Data Firehose
- **バッファデフォルト**: 300秒
- **ほぼリアルタイム + 最小運用** → 第一候補
- **S3からの取込みは不可**（ストリーム受信専用）

---

## AWS Glue

### 主要機能
- **Crawler**: スキーマ推論・カタログ化
- **ETL Job**: 変換処理
- **ブックマーク**: 増分処理（S3/RDB対応）
- **groupSize**: 小さいファイルをまとめる
- **Data Quality**: データ品質チェック

### パーティション最適化
| 機能 | 実行場所 | インデックス | 大量パーティション |
|------|---------|-------------|------------------|
| Pushdown Predicates | Job側 | 不可 | 遅い |
| catalogPartitionPredicate | カタログ側 | 可能 | **速い** |

→ 大量パーティション + インデックス = **catalogPartitionPredicate**

### その他
- **プッシュダウン述語**: メモリ不足/最適化で選ぶ
- **Data Catalog暗号化**: **カスタマー管理キー(CMK)のみ**
- **Schema Registry**: スキーマ一貫性確保
- **FindMatches**: 異なるDDLから共通ユーザ取得
- **Crawlerが1テーブル出力**: 形式・圧縮・スキーマ・プレフィックスが統一

---

## AWS Glue DataBrew

- **ノーコード**データプレパレーション
- **プロファイルジョブ**: 統計・分布・外れ値
- **レシピジョブ**: 列結合・型変換・欠損穴埋め
- **データ品質ルール**: 制約定義・バリデーション
- **PII検出・マスキング**: 組込み機能あり

---

## Amazon Athena

### パーティション管理
- **MSCK REPAIR TABLE**: パーティション追加
- **ALTER TABLE DROP PARTITION**: 削除
- **パーティション射影**: カタログ参照スキップ、インメモリ計算 → **大量パーティションに最適**

### キュー対策
- **プロビジョンド容量**: 専用リソース確保でキュー回避

### 注意点
- 直接S3クエリ不可 → **Glue Data Catalog必須**（省略されることも）
- **Apache Iceberg**: レコードレベルの挿入/更新/削除、タイムトラベル対応

---

## Amazon DynamoDB

### ホットパーティション対策
- **高カーディナリティのキー**を選ぶ（SENSOR_ID > BUILDING_ID）

### その他
- **TTL**: 設定可能

---

## Amazon OpenSearch Service

### ストレージ階層
| 階層 | 保存先 | コスト | 性能 | 用途 |
|------|--------|-------|------|------|
| Hot | EBS | 高 | 最高 | 頻繁な書込み |
| **UltraWarm** | S3+キャッシュ | **低** | 高 | **大規模検索** |
| Cold | S3 | 最小 | 低 | アーカイブ |

→ 「PB級 + コスト削減」= **UltraWarm**

### 用途
- **全文検索最適化** → OpenSearch
- **メタデータ検索** → OpenSearch Index + S3参照

---

## AWS Lake Formation

### 権限制御
| 項目 | Glue + IAM | Lake Formation |
|------|-----------|----------------|
| 最小単位 | テーブル | **列・行・セル** |
| 設定方法 | JSON | GRANT/REVOKE |

→ 「列・行・セルレベル制御」= **Lake Formation**

### データレイク権限階層
- データレイク管理者 > データベース作成者 > テーブル作成者

---

## Amazon SQS

### メッセージ削除条件
- DeleteMessage API
- maxReceiveCount到達
- キューパージ

※可視性タイムアウト切れでは削除されない

---

## AWS DMS (Database Migration Service)

- **フルロード + CDC**: 初期移行+継続同期を1タスクで
- **CDC**: トランザクションログ監視でリアルタイム同期
- **S3転送時**: 増分ではなくフルバックアップ
- **Redshift転送**: 対応

---

## AWS Step Functions

### 必須定義（処理追加時）
- **Type**: Task
- **Resource**: 実行API（例: `arn:aws:states:::aws-sdk:glue:startCrawler`）
- **Next**: 次の状態名（End: falseではない）

### 用途
- 外部API待ち時間をワークフロー化
- DynamoDB更新はSDK統合で直接実行

---

## Amazon EventBridge

### Pipes
- **SQS → フィルタ → Step Functions**: コード不要で直結
- 入力変換も可能

### オンプレ連携
- PutEvents APIでLambdaトリガー可能

---

## Amazon QuickSight

- **BIツール**（可視化・分析・共有）
- **SPICE**: 高速インメモリエンジン
- **データ準備**: 計算フィールド追加、型変換
- **ML Insights**: 異常検知・予測

---

## Amazon EMR

### ストレージ
- **HDFS**: オンメモリ、高速ランダムI/O、クラスター削除で消える
- **EMRFS**: S3への読み書き、読取りは実行あたり1回

### コスト最適化
- オンデマンド + スポットの組み合わせ推奨

### 暗号化
- セキュリティ設定機能で転送時/保存時暗号化

---

## その他サービス

### Amazon AppFlow
- SaaS → AWS転送、経路暗号化対応

### Amazon Macie
- S3のPII検出（マスクはしない）

### SageMaker Data Wrangler
- 視覚的データプレパレーション・特徴量エンジニアリング

### CloudTrail Lake
- CloudTrailをSQLでクエリ（出力はJSON）

### AWS SCT (Schema Conversion Tool)
- DB移行前のスタンドアロンアプリ
- 移行先決定済み → DMS Schema Conversion

---

## 暗号化キー判断

### デフォルトキーでOK
- 特に指定なし + 最小運用 + 単一アカウント

### CMK必須
- **クロスアカウント共有**
- **キーポリシー編集**
- **BYOK**
- **Glue Data Catalog暗号化**
- **CloudWatch Logs暗号化**

---

## データ形式

### Parquet/ORC
- 列指向、分析向け、スキャン削減

### Avro
- リアルタイム性が必要な時

---

## コスト比較

| 項目 | CloudWatch Logs | S3 + Athena |
|------|----------------|-------------|
| 取込み | 高(0.50/GB) | 低 |
| ストレージ | 高 | 低 |
| クエリ | 同等 | Parquetで大幅削減 |

→ 「大容量ログ + コスト効率」= **S3 + Athena**

---

## Lambda制限

- **エフェメラルストレージ**: 最大10GB
- マウントボリューム必要時 → **EFS**

---

## 二層暗号化

- **DSSE-KMS**: 2回暗号化、セキュリティ要件が厳しい場合

---

## KMSポリシー

| サービス | 同一アカウントアクセス |
|----------|---------------------|
| S3 | IAM or バケットポリシー |
| KMS | IAM **and** キーポリシー |
