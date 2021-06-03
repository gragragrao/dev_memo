# Docker SQL Server

## 目的

- DWHで取ってきたバックアップファイルがSQL Serverのbakファイル
- SQL Serverはmacでは開けないので、dockerで環境を作る必要がある

## 方法

### 1. docker for mac のインストール

- [公式URL](https://hub.docker.com/editions/community/docker-ce-desktop-mac) からインストールした。
- SQL Serverではディスク領域が3.25GB以上必要なため、 `Preference > Advanced` からメモリの上限を4GBに増やす。
- CPUコア数は4に設定した。

### 2. コンテナイメージの作成と実行

#### pull

- [pull出来るimage一覧](https://hub.docker.com/_/microsoft-mssql-server)

```.sh
docker pull mcr.microsoft.com/mssql/server:2019-CU9-ubuntu-16.04
```

#### run

```.sh
docker run \
    -e 'ACCEPT_EULA=Y' \
    -e 'SA_PASSWORD=m-(ugrLM7xb(' \
    -p 1401:1433 \
    --name dwh_sql \
    -v dwh_sqldata:/var/opt/mssql \
    -d mcr.microsoft.com/mssql/server:2019-CU9-ubuntu-16.04
```

#### check

- ここで `コンテナID` が確認できる。

```.sh
# プロセスを確認する
docker ps
```

#### connect

- パスワードは `""` で囲んで入力する。
- `1>` が出たら成功。

```.sh
# dockerのコンテナへ接続する
$ docker exec -i -t <container_id> "bash"

# sql serverへログイン
(mssql@ec02e2cc37de)$ /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "m-(ugrLM7xb("
```

#### file transport

- ローカルからdockerコンテナにファイルを転送する

```.sh
docker exec -it dwh_sql mkdir /var/opt/mssql/backup

docker cp <local file path> dwh_sql:/var/opt/mssql/backup
```

#### バックアップを復元する

##### 1. バックアップ内の論理ファイル名とパスを一覧表示する

```.sh
$ docker exec -it dwh_sql /opt/mssql-tools/bin/sqlcmd -S localhost \
    -U SA -P 'm-(ugrLM7xb(' \
    -Q 'RESTORE FILELISTONLY FROM DISK = "/var/opt/mssql/backup/dwh_bk_20210219.bak"' \
    | tr -s ' ' | cut -d ' ' -f 1-2

> LogicalName PhysicalName
-------------------------------------------------------------------------------------------------------------------------------- 
dwh_bk_20210219 C:\Program
dwh_bk_20210219_log C:\Program
```

##### 2. バックアップの復元を実行する

```.sh
$ docker exec -it dwh_sql /opt/mssql-tools/bin/sqlcmd \
    -S localhost -U SA -P 'm-(ugrLM7xb(' \
    -Q 'RESTORE DATABASE DWH_SAMPLE FROM DISK = "/var/opt/mssql/backup/dwh_bk_20210219.bak" WITH MOVE "dwh_bk_20210219" TO "/var/opt/mssql/data/dwh_bk_20210219", MOVE "dwh_bk_20210219_log" TO "/var/opt/mssql/data/dwh_bk_20210219_log"'

> Processed 536 pages for database 'DWH_SAMPLE', file 'dwh_bk_20210219' on file 1.
Processed 2 pages for database 'DWH_SAMPLE', file 'dwh_bk_20210219_log' on file 1.
RESTORE DATABASE successfully processed 538 pages in 0.018 seconds (233.289 MB/sec).
```

##### 3. 復元されたデータベースを確認する

```.sh
$ docker exec -it dwh_sql /opt/mssql-tools/bin/sqlcmd \
    -S localhost -U SA -P 'm-(ugrLM7xb(' \
    -Q 'SELECT Name FROM sys.Databases'

> Name
----------------
master
tempdb
model
msdb
DWH_SAMPLE   
```

##### 4. テーブルをcsvで書き出す

```export.sql
set nocount on

select
    *
from
    $(table_name)

set nocount off
```

```.sh
$ docker cp export.sql dwh_sql:/tmp
$ docker exec -it dwh_sql mkdir /tmp/exported_data
$ cd /Users/kentaroogura/Library/Mobile\ Documents/com\~apple\~CloudDocs/txp/2.\ DWH/2.\ 院内DWH作成/202103宇都宮データ確認
$ chmod 755 export.sh
$ ./export.sh
$ docker cp dwh_sql:/tmp/exported_data/. ./exported_data/

# 1つのテーブルのエクスポート文
$ docker exec -it dwh_sql /opt/mssql-tools/bin/sqlcmd \
    -S localhost \
    -d DWH_SAMPLE \
    -U SA -P 'm-(ugrLM7xb(' \
    -b -s, -W \
    -i /tmp/export.sql \
    -v table_name='<table_name>' \
    -o /tmp/exported_data/<table_name>.csv

>
```

### 3. SQL Serverコマンド

#### データベース一覧を表示する

```.sqlcmd
select name from sys.databases;
go
```

#### データベースを作成する

```.sqlcmd
create database <database-name>;
go
```

#### テーブルの一覧

```.sqlcmd
select name from sysobjects where xtype = 'U';
go
```

## 参考

- [Microsoft公式 - Linux の Docker コンテナーで SQL Server データベースを復元する](https://docs.microsoft.com/ja-jp/sql/linux/tutorial-restore-backup-in-sql-server-container?view=sql-server-ver15)
- [Dockerを使用してMacからSQL Serverに接続する（方法編）](https://qiita.com/m_rn/items/5c1c5523146d964ff9e4)
- [コマンドプロンプトでSQL Serverを使う](https://qiita.com/chihiro/items/75b12aca631f79be28b2)
- [SQLServerのデータをCSVに出力するバッチファイルを作ろう](https://qiita.com/erik_t/items/e634c9d549e93e5badd5)
