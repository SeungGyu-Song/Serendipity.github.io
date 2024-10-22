# Docker 설치
ref : 
### sudo 안 붙이고 docker 실행하기
`sudo chmod 666 /var/run/docker.sock` 
## nvidia-docker2 설치
이걸 설치해야 내 로컬에 있는 gpu를 사용할 수가 있음.
대신에 로컬에서 nvidia-driver를 미리 설치해야함.

ref : [tstory 블로그](https://d-ontory.tistory.com/8)
```sh
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \ && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \ && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update

sudo apt-get install -y nvidia-docker2

```

`docker run --rm --gpus all ubuntu:20.04 nvidia-smi`로 테스트.



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

`docker run -itd --name pytorch -v /home/snggu/:/root/snggu -p 8888:8888 --gpus all --restart=always pytorch/pytorch`

- `-v` 옵션은 로컬 호스트와 도커 컨테이너 사이에 공유 디렉토리를 연결해주는 옵션
	- 내 로컬 디렉토리를 써주면 된다.
	- 여기서는 로컬의 /home/snggu 디렉토리를 도커 컨테이너의 /root/snggu로 연결해주었음.
	- 따라서 내 컴퓨터의 /home/snggu에 파일을 넣어주면 도커 컨테이너에서 /root/snggu에서 해당 파일을 사용할 수가 있게 됨.
- `--restart=always` 옵션은 도커가 재실행 될 때 해당 컨테이너가 같이 실행되는 거.
	- 필요 없을 듯.
- 

| 수행                               | 명령어                            |     |
| -------------------------------- | ------------------------------ | --- |
| 실행중인 컨테이너 목록 확인                  | docker ps -a                   |     |
| 전체 컨테이너 목록 확인                    | docker container ls -a         |     |
| 컨테이너 시작                          | docker start 컨테이너 ID           |     |
| 컨테이너 접속                          | docker attach 컨테이너 ID          |     |
| 컨테이너 멈춤                          | docker stop 컨테이너 ID            |     |
| 컨테이너 생성 및 시작                     | docker run 컨테이너  ID            |     |
| 컨테이너 생성 및 시작 및 접속                | docker run -it 컨테이너 ID         |     |
| 컨테이너 삭제                          | docker rm 컨테이너 ID              |     |
| 실행되고 있던 컨테이너 접속 (터미널 분할해서 동시 사용) | docker exec -it 컨테이너 /bin/bash |     |
| 컨테이너 빠져나오기                       | exit                           |     |
| 이미지 확인                           | docker images                  |     |


# DockerFile 만들기
내가 도커 이미지의 컨테이너에서 작업을 하다가 새로운 의존성을 설치하게 될 경우, 이를 업데이트 하는 방식은 두 가지가 있다. : DockerFile 수정, Docker commit

Docker commit의 경우 해당 container를 끄기 전에 현재 상태를 이미지로 저장하는 방식이다. 하지만 남들과 공유를 하거나 유지 관리를 하기에 부적절한 것 같아서 DockerFile 만드는 방식으로 하려고 한다.

```Docker
FROM pytorch/pytorch:2.5.0-cuda11.8-cudnn9-devel

MAINTAINER snggu@kaist.ac.kr

RUN apt-get -y -qq update && \

pip install numpy matplotlib librosa

WORKDIR /root/snggu

COPY . /root/snggu
```

