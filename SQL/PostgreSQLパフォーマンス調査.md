# PostgreSQLパフォーマンス調査

## 前提

- pgAdminで確認

## クエリプラン(実行計画)の確認方法

postgreSQLではEXPLAINコマンドが存在しており、
こちらを利用することでクエリプランの確認が可能
pgAdminではクエリタブにExplain(F7)が存在している。
Explain(F7)の隣にANALYZEとSettingsがありオプションを設定することが可能。

EXPLAINのみだとクエリは実行されない。
EXPLAIN ANALYZEでクエリが実行される。

```sql
explain select * from orders;
```

### Explain(F7)の結果

Node Typeが「 Seq Scan」がテーブルスキャン

#### Analysisタブ

- Exclusive（専用時間）
  そのノード自体だけにかかった時間
- Inclusive（包括時間）
  そのノードと子ノード全体にかかった時間
- Actual（実際の行数）
- Plan（予想行数）

#### Statisticsタブ

- % of query
  クエリ全体の実行時間に対する、その処理の時間の割合
- % of relation
  そのテーブル（リレーション）へのアクセス処理のうち、その処理タイプが占める割合
  インデックス活用度が分かる

#### Gather

NoteTypeがGather
パラレルクエリ (並列クエリ) の実行時に、
複数のワーカープロセスで実行された結果をまとめてリーダープロセスに返すノードのこと。
結果の順序が重要でない場合に利用されます。﻿

Gather Mergeは、ソートされた結果を必要とする場合に利用されるノード

## pg_stat_statements

SQLServerのプロファイラみたいなことができる
拡張モジュールとして提供されている。

ただし、条件の値が違っても構造が同じなら同一クエリとして統計が取られる。
これによりSQL文のパターン分析ができるが、特定の値での性能問題は見つけにくい。

起動コマンドに"shared_preload_libraries=pg_stat_statements"が必要

```yml
# docker-compose.yml
services:
  postgres:
    image: postgres:latest  # 既存イメージを直接使用
    container_name: postgres_test_pafo
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: testdb
      POSTGRES_USER: postgres
    ports:
      - "5432:5432"
    command:
      - "postgres"
      - "-c"
      - "shared_preload_libraries=pg_stat_statements"
      - "-c"
      - "pg_stat_statements.track=all"
      - "-c"
      - "pg_stat_statements.max=10000"
      - "-c"
      - "pg_stat_statements.save=on"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # 初期化スクリプトのみ
    restart: unless-stopped

volumes:
  postgres_data:
```

```sql
--有効化
CREATE EXTENSION pg_stat_statements;
```

以下が使用可能になる

```sql
select * from pg_stat_statements

SELECT 
    LEFT(query, 80) AS sql_query,
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_time_ms,
    ROUND(mean_exec_time::numeric, 2) AS avg_time_ms,
    shared_blks_read AS disk_reads,
    shared_blks_hit AS buffer_hits,
    shared_blks_written AS disk_writes
FROM pg_stat_statements
WHERE calls > 0
ORDER BY total_exec_time DESC
LIMIT 10;
```

```sql
# 設定値確認
SHOW pg_stat_statements.max;          -- 保存する統計の最大数
SHOW pg_stat_statements.save;         -- 再起動時の統計保持
SHOW pg_stat_statements.track;        -- 追跡レベル
SHOW pg_stat_statements.track_utility; -- ユーティリティ文の追跡
```

