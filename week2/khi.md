# Container lifetime, volume

컨테이너는 휘발성이 강합니다.
근데 컨테이너 사라진다고 그 안에 들은 중요한 데이터까지 다 사라지면 곤란하잖아요.
이를 대비하기 위해 컨테이너 생존과 상관없이 저장되는 volume이라는게 있어요.
일종의 데이터베이스? 라고 생각하면 됩니다.

## Container Lifetime & Persistent Data
![](https://images.velog.io/images/kpl5672/post/1254c06b-1c57-41f5-b48c-723910860b4f/Screen%20Shot%202020-12-08%20at%209.21.33%20PM.png)
## Persistent Data: Data volumes
-VOLUME command in Dockerfile

-mysql offical 도커 이미지를 보면 
`VOLUME /var/lib/mysql`
요런 코드가 있다.
명시된 VOLUME은 컨테이너가 내려가도 사라지지 않는다.
수동으로 지워야 지워진다.
VOLUME은 컨테이너보다 수명이 길다!

mysql 이미지를 inspect해보자. 아래와 같은 volume정보를 얻을 수 있다.
docker pull mysql -> docker image inspect mysql
![](https://images.velog.io/images/kpl5672/post/87e34b95-fa48-4540-979a-ddd1f9b73992/Screen%20Shot%202020-12-08%20at%209.27.10%20PM.png)

이제 이미지를 실행해보자.
`docker container run -d --name -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql`
실행한 이미지를 inspect 해보면 mount에 대한 정보를 볼 수 있다.
`docker container inspect mysql`

![](https://images.velog.io/images/kpl5672/post/a24797e9-96bc-4c06-aad4-aa5a1b0a67e4/Screen%20Shot%202020-12-08%20at%209.29.24%20PM.png)

`docker volume ls`로 볼륨 리스트 알 수 있다.
`docker volume inspect {volume_name}`으로 볼륨을 검사해보면 아래결과가 나온다.
![](https://images.velog.io/images/kpl5672/post/32dd7d4d-6827-4e33-b245-fb101d961458/Screen%20Shot%202020-12-08%20at%209.31.05%20PM.png)

#### 중요포인트
`docker container run -d --name mysql MYSQL_ALLOW_EMPTY_PASSWORD=True mysql` -> `docker container stop mysql`-> `docker container rm mysql`
이렇게 컨테이너 만들고 지워도 volume은 살아있다. `docker volume ls`로 확인 가능.

근데 이 volume들은 user-friendly하지가 않다.
user-friendly하게 만들려면 'named volumes'를 사용하면 된다.
it's a friendly way to assgin vols to containers!

`docker container run -d --name mysql MYSQL_ALLOW_EMPTY_PASSWORD=True -V /var/lib/mysql mysql`
위 명령어처럼 -v를 줄 수도 있는데, 이건 Dockerfile의 VOLUME명령어 부분과 역할이 똑같기 때문에 굳이 명령어 부분에서 안해도 된다.
더 유용한건 아래처럼 '볼륨이름:'을 추가해주는 것이다.
`docker container run -d --name mysql MYSQL_ALLOW_EMPTY_PASSWORD=True -V mysql-db:/var/lib/mysql mysql`
이렇게 하고 docker volume ls때려보면 방금 지정한 볼륨 이름을 볼 수 있다.
![](https://images.velog.io/images/kpl5672/post/93378d58-47e2-45ed-922a-3e44ec222e9b/Screen%20Shot%202020-12-08%20at%209.38.37%20PM.png)

`docker volume inspect mysql-db` 로 정보 체크도 가능하다.

-위의 컨테이너를 내리자(볼륨은 살아있다).
-mysql3로 이름 지어진 또다른 컨테이너를 만다는데, 방금 위에서 만든 volume을 할당한다.
`docker container run -d --name mysql3 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql`
이렇게하고 `docker dontainer inspect mysql3`해보면,
좀 더 읽기 쉬운 이름을 볼 수 있고, 경로도 바뀐걸 알 수 있다.
![](https://images.velog.io/images/kpl5672/post/f007d068-6083-4536-98f5-672dc19b0e9b/Screen%20Shot%202020-12-08%20at%209.42.14%20PM.png)

### docker volume create
:required to do this before "docker run" to use custom drivers and labels
`docker volume create `
## Persistent Data: Bind Mounting
-host 파일이나 폴더를 컨테이너의 파일 혹은 폴더로 mapping시키는 것.
-Basically just two locations pointing to the same file(s)
-Dockerfile안에선 쓸 수 없고, container run명령어 뒤에 써야한다.
`... run -v /Users/bret/stuff:/path/contianer`

1)`docker container rn -d --name nginx1 -p 8080:80 nginx`
2)`docker container run -d --name nginx2 -p 80:80 -v $(pwd):/usr/share/nginx/html nginx`
1)은 디폴트 nginx가 실행되고, 2)는 로컬 현재 디렉토리의 파일을 담고있는 컨테이너가 실행된다. 
첫 사진은 디폴트 엔진엑스 화면, 두번쨰는 index.html을 "HI"로 바꾸고 2)를 실행한 결과다.
![](https://images.velog.io/images/kpl5672/post/a7dcc4d7-7414-462c-b5b7-3e66aa667f35/Screen%20Shot%202020-12-13%20at%209.00.56%20PM.png)
![](https://images.velog.io/images/kpl5672/post/bd86b953-6543-4d5e-81e5-97885d74909b/Screen%20Shot%202020-12-13%20at%209.01.00%20PM.png)

-만약, 로컬의 index.html을
```
"HI"
"changed version"
```
로 바꾸면, 돌고있는 컨테이너도 바로 변한다.
![](https://images.velog.io/images/kpl5672/post/f76d8347-85fc-4e06-a0fd-f95bcf94966f/Screen%20Shot%202020-12-13%20at%209.05.32%20PM.png)

-컨테이너 안에서 index.html을 지워버리면 host에서도 지워진다.

-위 방법은 개발환경에서 매우 유용하다.run command -v로 실행하면 로컬 개발이 실시간으로 컨테이너에 반영되기 때문이다. container안의 shell에 들어갈 필요가 없어진다.
wow!

#### Quiz
Which type of persistent data allows you to attach an existing directory on your host to a directory inside of a container?

->Bind mount



## Database Upgrades with Named Volumes
과제 내용:
db를 도커로 돌리고 있는데, 버전 업그레이드를 해야 할 상황이 오면 어떻게 대처해야 할까? (일반적으로 컨테이너를 내리면 데이터도 같이 사라지는데, 새로 업데이트한 버전이 기존 데이터를 그대로 유지하게 해야한다.)

지침:
1)구버전 postgres를 docker run한다(v옵션으로 볼륨 지정하기)
2)container stop하기
3) 새로운 버전 docker run하기(v옵션으로 기존 볼륨 지정)
4)로그를 확인해서 데이터가 그대로 이전됐는지 확인한다.
팁:
-`docker volume ls`로 내가 지정한 볼륨 볼 수 있다.

정답
`docker container run --name old_psql -d -v psql:/var/lib/postgresql/data postgres:9.6.1`
`docker container logs -f psql`
`docker container run --name new_psql -d -v psql:/var/lib/postgresql/data postgres:9.6.2`
`docker container logs `



## Edit Code Running in Containers with Bind Mounts

