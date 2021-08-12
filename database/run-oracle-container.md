# 오라클 컨테이너 실행

```bash
$ docker container run --detach --interactive --tty --name myoracle --publish-all store/oracle/database-enterprise:12.2.0.1
246e931cc34df2a5dbc9cc07e5a5974b53f760f30379306e47c64692f3153b3b

$ docker container ls --filter name=myoracle
CONTAINER ID   IMAGE                                       COMMAND                  CREATED         STATUS                   PORTS                                                                                      NAMES
246e931cc34d   store/oracle/database-enterprise:12.2.0.1   "/bin/sh -c '/bin/ba…"   2 minutes ago   Up 2 minutes (healthy)   0.0.0.0:55005->1521/tcp, :::55005->1521/tcp, 0.0.0.0:55004->5500/tcp, :::55004->5500/tcp   myoracle
```

# 퍼블리시된 포트 확인

```bash
$ docker container port myoracle
1521/tcp -> 0.0.0.0:55005
1521/tcp -> :::55005
5500/tcp -> 0.0.0.0:55004
5500/tcp -> :::55004
```

# sqlplus 로 데이터베이스 접속

## 연결 정보를 커맨드에서 명시하는 방법

```bash
$ sqlplus sys/Oradoc_db1@"(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=55005))(CONNECT_DATA=(SID=ORCLCDB)))" as sysdba
```

## 연결 정보를 tnsnames.ora 파일에 명시하는 방법

`tnsnames.ora` 파일의 경로는 임의적이지만 이 파일이 위치한 디렉터리의 경로를 `TNS_ADMIN` 환경변수에 지정해야 한다:

```bash
$ echo $TNS_ADMIN
.

$ cat ./tnsnames.ora
ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=55005))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=55005))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))

$ sqlplus sys/Oradoc_db1@ORCLCDB as sysdba
```