# サービス別サマリ（試験直前用）

## SQS
- 削除条件: DeleteMessage / maxReceiveCount到達 / キューのパージ
- 可視性タイムアウトは再配信防止の猶予（削除ではない）

## Redshift
- Spectrum: S3を直接クエリ。低頻度/長期データ向き（JOINや性能は弱め）
- 分散スタイル: KEY=結合最速/偏り注意, ALL=小テーブル向け, EVEN=均等だが結合遅い
- COPY暗号化: SSE-KMS対応、SSE-C非対応
- 暗号化は再作成不要で有効化可能、KMSローテ可
- ステージングテーブルでUPSERT/重複排除
- S3連携: `COPY`(S3→RS)、`UNLOAD`(RS→S3)、Data Sync不可
- Data Sharing: 共有ビュー + Lake FormationでPII除外、リージョン/アカウント跨ぎ可
- DDM: ロールでマスキング可
- 監査: Redshiftネイティブ + CloudTrail
- ストアドプロシージャ実行: `CALL ...`、アプリ保存/DB保存の2種
- Query Editor v2のスケジュールはEventBridge + Redshift Data API
- Federated Query: RDS/Aurora (MySQL/PostgreSQL)
- IdPフェデレーションをネイティブ提供
- XMLは`COPY`非対応 → Glueで変換＋小ファイルのグループ化＋ブックマーク

## Lambda
- KDS連携の同時実行数は「シャード数×並列化係数」で決まる
- iteratorAgeMilliseconds = 取得から処理までの遅延
- エフェメラルストレージは最大10GB、永続はEFS

## Kinesis Data Streams
- スループット最適化: シャード増/並列化係数/拡張ファンアウト/バッチサイズ&ウィンドウ
- 書き込み1MB不足→シャード増 or KPL、読み出し2MB不足→拡張ファンアウト/並列化係数
- S3直書き不可。S3出力はFirehose必須
- パーティションキー: 最高スループット=ランダム、順序保持=グループ単位
- KDSの価値: 保持/再試行 + 複数宛先のファンアウト

## Firehose
- バッファ時間デフォルト300秒

## AppFlow
- SaaS→AWS転送、経路暗号化可

## AWS Glue
- メトリクス: ジョブ実行モニタリングでプロファイル確認
- 小さなファイル統合: `groupSize` を上げる
- プッシュダウン述語: 読み取り量削減/メモリ不足対策
- 大量パーティション: `catalogPartitionPredicate` + パーティションインデックス
- ブックマーク: 増分処理（RDS→S3等）
- Crawler: スキーマ/形式/プレフィックスが揃うと単一テーブル
- Data Catalog暗号化はCMK必須
- Schema Registry: スキーマ一貫性確保
- FindMatches: 異なるDDL間の重複/名寄せ
- Data Quality: データチェック追加で一貫性/正確性を評価

## DataBrew
- ノーコード整形、品質評価はData Quality Rules
- プロファイル=可視化、レシピ=加工

## Athena
- S3クエリはGlue Data Catalog前提
- パーティション射影: 大量パーティションで高速
- パーティション管理: MSCK REPAIR / ALTER TABLE DROP PARTITION
- キュー遅延対策: プロビジョンド容量

## DMS
- CDCは「フルロード+CDC」推奨、ソースログ有効化
- Redshiftへの転送可
- DMS→S3は増分ではなくフルバックアップ

## DynamoDB
- ホットパーティション対策: 高カーディナリティのPK
- TTL設定可能

## OpenSearch
- PB級で低コスト&高検索: UltraWarm

## Macie
- S3のPII検出/監査、マスキングはGlue側

## Step Functions
- 新タスク追加: `Type: Task` + `Resource` + `Next`（Endではない）

## EventBridge
- オンプレからPutEventsでLambda起動可能

## EMR
- セキュリティ設定でTLS/EMRFS暗号化
- コスト最適化: On-Demand + Spot併用
- HDFS vs EMRFS: 高速ランダムI/OはHDFS、EMRFSはS3連携

## CloudWatch
- データパイプライン監視はCloudWatch
- Logs Insightsでログクエリ

## QuickSight
- BIツール、SPICEで高速
- データ準備はQuickSight内でノーコード

## Lake Formation
- カラム/行単位のアクセス制御（Glue+IAMはテーブル単位）
- データレイク権限: 管理者/DB作成者/テーブル作成者

## Iceberg
- レコード削除/更新/タイムトラベル対応、AthenaでDELETE

## KMS
- デフォルトキーは運用最小、条件付きでCMK必須
- CMK必須: Glue Data Catalog暗号化、CloudWatch Logs暗号化、クロスアカウント、BYOK、鍵ポリシー編集
- CMKは「IAMポリシー AND キーポリシー」両方必要
- DSSE-KMS: 2重暗号化

## Kafka / MSK
- KDSはAWS独自の高マネージド、MSKはKafkaのマネージド

## その他
- Hive: Hadoop HDFSのDWH、AthenaはS3
- Parquet/ORC/Avro: Avroはリアルタイム向き
- CloudTrail Lake: CloudTrailをSQLで分析（AthenaはJSON注意）
- AWS SCT: 移行前のスタンドアロン、移行先決定後はDMS Schema Conversion
