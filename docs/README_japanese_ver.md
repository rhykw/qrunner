# qrunnerの設定方法
## 1. `.drone.yml`
qrunnerの実行の起点は`.drone.yml`になるため、qrunnerを正しく動作させるためには`.drone.yml`を正しく書く必要がある。  
サンプルは[ここ](../.drone.yml)に置いている。  
主な設定ポイントは以下
- localhostへのテスト用にMySQLのコンテナを`MYSQL_ALLOW_EMPTY_PASSWORD=yes`で上げる
- masterブランチの時のみ`EXEC_MODE=remote`とする（リモートサーバにクエリを実行するのはmasterマージのときだけにする）
- remote実行の際は、`MYSQL_USER`と`MYSQL_PASSWORD`を`.drone.sec`から受け取るようにする

### 環境変数について
qrunnerの動作を制御する環境変数は`.drone.yml`で設定が可能である。  
以下、設定可能な環境変数を示す。  

| 変数名 | 値 | 説明 |
| --------- | ------ | ----------- |
| `EXEC_MODE` | `local` or `remote`[Default: `local`] | 実行モード。`local`の場合はlocalhost上に起動したMySQLに対してクエリのテストを行う |
| `QUERIES_DIR` | `queries` | QUERIES_DIRは実行するクエリを置くためのディレクトリ |
| `MYSQL_USER`| `MySQLのユーザ名`[Default: `root`] | - |
| `MYSQL_PASSWORD`| `MySQLのパスワード`[Default: `""`] | [`.drone.sec`](https://docs.tea-ci.org/usage/secrets/)から取得するようにした方が良い |
| `SCHEMA_DIR` | `ディレクトリ名`[Default: `schema`] | localhostにクエリの実行テストを行う際に用いるDBスキーマを格納するディレクトリ |
| `SERVERS_INFO` | `toml形式のファイル名`[Default: `servers.toml`] | クエリを実行するDBサーバの情報を格納するファイル |
| `GATEWAY_LOCAL_PORT` | `ポート番号`[Default: `3307`] | ssh_gatewayを使う際のローカルポートとして利用される |

## 2. サーバ情報
`SERVERS_INFO`で指定したファイル（指定していない場合は`servers.toml`）に、接続するDBサーバ群のホスト名やポート番号の情報をまとめておく必要がある。
サンプルは[ここ](../.servers.toml)に置いている。  
書き方は以下の通りである。
```
[サービス名]
[サービス名.DBホストを一意に表すキー]
host_name = "<ホスト名 or IPアドレス>"
port = <ポート番号>
ssh_gateway = "<SSHユーザ名>@<SSHホスト名>"
```
「DBホストを一意に表すキー」は後述するDBスキーマやクエリの配置するディレクトリ名に対応するため重要。  
`port`は指定しない場合は`3306`が用いられる。  
SSH Gateway経由で接続したい場合は、`ssh_gateway`を指定する。  

## 3. DBスキーマ
以下のディレクトリパス配下に、実行するDBのスキーマファイルを置いておかなければならない。 
```
SCHEMA_DIR(デフォルト:schema)/サービス名/DBホストを一意に表すキー
```
配置したスキーマファイルは、localhostへのテスト時（`EXEC_MODE=local`）のDBのセットアップに用いられる。  

**※ 配置するスキーマのファイル名はDB名と同一にする**

## 4. クエリの書き方
### 基本ルール

- 1PRにつき1SQLファイルとする。（同一サーバへの複数クエリ実行したい場合は1SQLファイルに書

### SQLファイルの配置
追加するSQLファイルは以下の命名規則に従い配置する。
```
QUERIES_DIR(デフォルト:queries)/サービス名/DBホストを一意に表すキー/日付_任意の文字列（テーブル名など）.sql
```
ファイル名は実行には関係なく、ユニークな名前を付ければ良いため、日時などを用いると良い。  
