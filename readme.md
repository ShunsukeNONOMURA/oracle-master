# oracle master
dockerでのoracleのサーバー起動。

## 使い方
起動まで
```
git clone https://github.com/ShunsukeNONOMURA/oracle-master.git

# setup (ここでは簡易的に権限設定している)
mkdir oradata
mkdir startup
chmod 777 oradata
chmod 777 startup

# 起動
docker compose up

# DATABASE IS READY TO USE! が表示される
```

以下実行して初期化
```
docker compose exec db ./setPassword.sh password123
docker compose exec db sqlplus sys/password123@freepdb1 as sysdba

# SQL内で接続情報作成
CREATE USER myuser
IDENTIFIED BY password123
DEFAULT TABLESPACE users
TEMPORARY TABLESPACE temp
QUOTA UNLIMITED ON users;

GRANT CONNECT, RESOURCE TO myuser;

exit;
```

接続確認
```
docker compose exec db sqlplus myuser/password123@freepdb1
```

## vector_memory_sizeの変更
vector検索用の初期設定。CDBレベルで設定して再起動が必要。
```
docker compose exec db sqlplus sys/password123@freepdb1 as sysdba

# SQL内で設定更新
show parameter vector_memory_size;

# CDBレベルで更新
alter session set container=CDB$ROOT;
alter system set vector_memory_size = 512M scope=spfile;
shutdown immediate;

# docker再起動してパラメータ更新
docker compose exec db sqlplus sys/password123@freepdb1 as sysdba
show parameter vector_memory_size;
```

## 参考
- [Oracle Database Free](https://container-registry.oracle.com/ords/f?p=113:4:103964643960397:::4:P4_REPOSITORY,AI_REPOSITORY,AI_REPOSITORY_NAME,P4_REPOSITORY_NAME,P4_EULA_ID,P4_BUSINESS_AREA_ID:1863,1863,Oracle%20Database%20Free,Oracle%20Database%20Free,1,0&cs=3elILP2QyPpWaLXPK3HnCwbeJr7Z9ATR98UrqEJigk6rGV-6knchnmVKYEnwAFxAqr8AZtlomHXrcdD6VKtxKUw)
- [Oracle Database 23ai のベクトル機能を試す手順 (1)](https://qiita.com/ryotayamanaka/items/156932a48e65d3ddc5ac)