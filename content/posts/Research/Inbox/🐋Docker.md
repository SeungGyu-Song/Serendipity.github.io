# Docker 설치
ref : 

### sudo 안 붙이고 docker 실행하기
`sudo chmod 666 /var/run/docker.sock` 
# Docker 명령어 

```txt
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

|옵션|설명|
|---|---|
|-d|detached mode 흔히 말하는 백그라운드 모드|
|-p|호스트와 컨테이너의 포트를 연결 (포워딩)|
|-v|호스트와 컨테이너의 디렉토리를 연결 (마운트)|
|-e|컨테이너 내에서 사용할 환경변수 설정|
|–name|컨테이너 이름 설정|
|–rm|프로세스 종료시 컨테이너 자동 제거|
|-it|-i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션|
|–link|컨테이너 연결 [컨테이너명:별칭]|


`docker run --rm -it ubuntu:22.04 /bin/bash` 
- 프로세스 종료하면 바로 삭제하려고  `--rm` 옵션 추가.
- bash shell을 실행하고 키보드 입력을 위해 `-it` 옵션 추가.

`docker run -d -p 1234:6379 redis`
- detached mode로 실행하기 위해 -d 옵션 추가.
- 컨테이너의 포트를 호스트의 포트로 연결하기 위해 -p 옵션 추가, 호스트의 1234 포트를 컨테이너의 6379 포트로 연결.
- 