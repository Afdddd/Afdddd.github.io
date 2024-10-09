# [CommerceCore]부하 테스트 (1) - 대용량 데이터 준비

서버의 부하 테스트를 위해 대용량 데이터가 필요하다.

python을 사용해서 대용량의 데이터가 담긴 csv파일을 만들고 csv 파일을 mysql의 `LOAD DATA INFILE` 명령어를 사용해 `Insert` 하도록 하겠다.

### csv 파일 생성

우선 임의의 사용자를 만들기 위해 python을 사용해 스크립트를 만들자

python의 faker 라이브러리를 사용할것이다.

faker는 테스트용 가짜 데이터를 생성할때 사용하는 라이브러리이다.

```java
pip install faker
```

![image.png](image.png)

설치가 완료되면 대량의 데이터를 생성할 스크립트를 짜자

```python
import csv
from faker import Faker
from datetime import datetime

# Faker 객체 생성
fake = Faker()

# CSV 파일 생성
filename = 'users_data.csv'
num_rows = 100000  # 생성할 데이터의 개수 (10만 개)

# CSV 파일에 데이터 작성
with open(filename, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    
    # CSV 파일의 헤더 작성
    writer.writerow(['user_id', 'address', 'create_time', 'email', 'name', 'password', 'phone_num'])
    
    # 무작위 데이터 생성 및 CSV에 기록
    for user_id in range(1, num_rows + 1):  # user_id는 자동 증가
        address = fake.address().replace("\n", ", ")
        create_time = fake.date_time_this_decade().strftime('%Y-%m-%d %H:%M:%S.%f')  # 6자리 밀리초 포함
        email = fake.email()
        name = fake.name()
        password = fake.password(length=10)
        phone_num = fake.phone_number()
        
        # 데이터를 한 줄씩 CSV에 작성
        writer.writerow([user_id, address, create_time, email, name, password, phone_num])

print(f'{num_rows}개의 데이터가 {filename}에 생성되었습니다.')
```

![image.png](image%201.png)

실행을 시켜주면 10만개의 데이터가 들어있는 csv 파일이 생성이 된다.

### `LOAD DATA INFILE` 실행

이제 이 csv 파일을 mysql의 `LOAD DATA INFILE`  명령어를 사용해 서버에 csv파일을 업로드를 해보자.

`LOAD DATA INFILE` 은MySQL에서 **외부 파일(CSV나 텍스트 파일 등)의 데이터를 테이블에 빠르게 삽입**하기 위해 사용되는 명령어이다.

```sql
LOAD DATA INFILE 'file_path' -- CSV 또는 텍스트 파일의 경로
INTO TABLE table_name -- 파일에서 읽어온 데이터를 삽입할 MySQL 테이블의 이름
FIELDS TERMINATED BY ',' -- 각 필드(열) 사이를 구분하는 문자
ENCLOSED BY '"' -- 필드 값이 큰따옴표(")로 감싸져 있는 경우
LINES TERMINATED BY '\n' -- 줄바꿈 구분자
IGNORE 1 LINES; -- 파일의 첫 번째 줄(헤더)을 무시하는 옵션
```

만든 csv파일이 로컬에 있어서 `LOCAL`  옵션을 붙여줬다. 그리고 10만개의 데이터중 중복데이터가 있을 경우 무시하고 삽입할 수 있도록 `IGNORE` 옵션을 붙여줬다.

```sql
LOAD DATA LOCAL INFILE 'D:/commercecore_data/users_data.csv' 
IGNORE INTO TABLE users 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\r\n' 
IGNORE 1 LINES 
(user_id, address, create_time, email, name, password, phone_num);
```

명령어를 실행 시켰더니 오류가 발생했다.

### Error : 2068

```
Error Code: 2068. LOAD DATA LOCAL INFILE file request rejected due to restrictions on access.
```

원인은 MySQL 서버에서 `LOCAL` 옵션을 사용하여 파일을 업로드하려 할 때 보안상의 이유로 접근 제한이 걸린 오류이다.

### 해결 방법1

```sql
SHOW VARIABLES LIKE 'local_infile';
```

명령어를 실행해 `local_infile` 변수가 활성화되어 있는지 확인하고, 비활성화되어 있다면 활성화 시키면 된다.

만약 `local_infile`이 `OFF`로 되어 있다면, 아래 명령을 실행시키면 활성화 시킬 수 있다.

```sql
SET GLOBAL local_infile = 1;
```

![image.png](image%202.png)

이제 다시 `LOAD DATA LOCAL INFILE` 를 실행 시켜보자

```
Error Code: 2068. LOAD DATA LOCAL INFILE file request rejected due to restrictions on access.
```

하지만 또 다시 오류가 발생했다.

계속 로컬에서 DB서버로 파일 업로드가 실패하는데 내 예상에는 DB 서버가 private 서브넷에서 운영되어서 접근이 불가능한것같다.

### 해결방법2

로컬에서 업로드가 되지 않으니 DB서버에서 업로드 하기로 했다.

csv 파일을 DB서버에 옮기고 `LOAD DATA INFILE`을 실행 시켜 보자

지금 상황이 CSV 파일이 로컬에 있고, MySQL DB 서버는 NCP의 private 서브넷에 있으며, 로컬에서 Bastion Host를 거쳐 MySQL Workbench를 통해 DB 서버에 연결하고 있는 상황이다.

1. Bastion Host에 파일 업로드

SCP(Secure Copy Protocol)를 사용해 로컬 csv 파일을 Bastion Host로 전송

```
scp -i [ssh키 암호] [복사할 csv파일 경로] [bastion 서버 ID]@[bastion 서버 ip]:[csv 붙여넣기할 경로]
```

1. **DB 서버로 파일 복사** (Bastion Host를 통해)

```
scp [csv 파일 경로] [DB 서버 ID]@[DB 서버 ip]:[csv 붙여넣기할 경로]
```

이제 `LOAD DATA INFILE` 명령어를 실행 시키면..

### Error : 1290

```
Error Code: 1290. The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

또 다시 오류가 발생했다.

에러 코드 1290은 MySQL 서버에서 `--secure-file-priv` 옵션이 활성화되어 있어, 특정 디렉토리 외부에서 파일을 로드할 수 없다는 것을 의미한다. MySQL에서 보안상의 이유로 `LOAD DATA INFILE` 명령어를 사용할 때, 파일을 특정 디렉토리에서만 접근할 수 있게 설정두고있다.

### 해결 방법

1. **MySQL에서 `secure-file-priv` 설정 확인**

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

결과는 세 가지 중 하나일 수 있다:

- **`NULL`**: 파일 로드를 완전히 비활성화한 경우
- **특정 경로**: MySQL 서버가 파일을 읽고 쓸 수 있는 허용된 디렉토리 경로
- **비어 있음**: 모든 경로에서 파일을 읽고 쓸 수 있음

해당 결과 디렉토리에서만 파일을 로드할 수 있다.

조회 결과 : `/var/lib/mysql-files/`

2. **파일을 허용된 디렉토리로 이동**

```bash
mv /tmp/users_data.csv /var/lib/mysql-files/
```

허용된 디렉토리를 확인한 후, 그 경로로 파일을 이동

이제 다시 `LOAD DATA INFILE` 을 실행 시키면 

![image.png](image%203.png)

정상적으로 10만개의 데이터가 삽입되었다. 10만개의 데이터가 삽입되는데 걸리는 시간은 2초밖에 걸리지 않았다.

이제 대용량을 데이터가 준비되었으니 ngridner로 부하 테스트를 진행 할 차례이다.