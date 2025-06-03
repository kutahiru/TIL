# SQL

## postgreSQL

### テーブル定義の確認

```
SELECT column_name, data_type, character_maximum_length, is_nullable 
FROM information_schema.columns 
WHERE table_name = 'テーブル名'
ORDER BY ordinal_position;
```

