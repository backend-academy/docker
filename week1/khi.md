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


## Database Upgrades with Named Volumes

## Edit Code Running in Containers with Bind Mounts

