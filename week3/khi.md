# Container Registries: Image Storage and Distribution

<강의 목차>
- Docker Hub: Digging Deeper
- Understanding Docker Registry
- Run a Private Docker Registry
- Using Docker Registry with Swarm
- Third Party Image Registries


이 섹션은 모두 container registries에 관한 내용입니다.
-image registry는 container plan에 속해야 한다.
-auto-build
-Docker Store(sotre.docker.com)이 Hub와 어떻게 다른지
-Docker Cloud(could.docker.com)이 Hub와 어떻게 다른지
-Cloud상에서 Swarms의 새로운 특성을 활용해 Mac/win과 Swarm을 연결하기
-Private image store로 활용을 위해 Docker Registry를 설치하고 사용하기
-3rd party registry 옵션들


## Docker Hub: Digging Deeper
-docker hub: Docker Registry plus simple image Building
-Hub에서 private은 1개까지 꽁짜고, 2개부턴 추가과금 됩니다.

#### Webhook
은 github, jenkins, travis-ci등과 연동시키는 기능을 합니다.
```
A webhook is an HTTP call-back triggered by a specific event. You can create a single webhook to start and connect multiple webhooks to further build out your workflow.

When an image is pushed to this repo, your workflows will kick off based on your specified webhooks.
```
#### organization
-hub에서도 organization 기능이 있다. 깃허브와 형태, 기능이 유사하다.

#### Automated Builds
> Autobuild triggers a new build with every git push to your source code repository

<img width="1194" alt="Screen Shot 2020-12-21 at 10 23 15 AM" src="https://user-images.githubusercontent.com/60768642/102730204-36883f00-4377-11eb-9522-b6d794fe47fd.png">


## Understanding Docker Registry

## Run a Private Docker Registry
-로컬 registry만드는 실습
-registry는 실제로 5000포트를 사용하는 HTTPS 서버다.
<이 강의에서 실습할 내용>
1) 어떤 이미지를 허브에서 내려받는다.
2) 그 이미지를 re-tag 해주고 새로운 registry로 push한다.
3) 해당 이미지를 local cache에서 제거하고, 새롭게 만들었떤 registry에서 pull받는다.
4) re-create registry using a bind mount and see how it stores data
(여러 registry가 있지만 강의에서는 빠른 전개를 위해 local 사용한다).

*Registry and proper TLS*
-도커는 HTTPS가 아닌 registry랑은 통신하지 않는다.
-except localhost(127.0.0.0/8)
-For remote self-signed TLS, enable "insecure-registry" in engine

실습
1)먼저 명령어로 registry를 만든다. 
`docker conatiner run -d -p 5000:5000 --name registry registry`
2) 실습용으로 적당한 hello-world이미지를 받는다.
`docker pull hello-world`

3)이미지를 푸쉬하기 위해 새롭게 태그해준다. 여기서 중요한 건 push하려는 target registry의 호스트를 적어주는 거다.
`docker tag hello-world 127.0.0.1:5000/hello-world`

`docker image ls`로 확인하면 아래처럼 두 개를 볼 수 있다.
<img width="806" alt="Screen Shot 2020-12-21 at 10 44 53 AM" src="https://user-images.githubusercontent.com/60768642/102730921-9ed82000-4379-11eb-89c9-794d6d59bbe2.png">


4) `docker push 127.0.0.1:5000/hello-world` 이 명령어를 치면 적혀진 ip:port를 먼저 확인해서 거기로 간다.

5) `docker image remove hello-world`,
`docker image remove 127.0.0.1:5000/hello-world`

이까지 하면 hello-world이미지는 이제 없다.

6) `docker pull 127.0.0.1:5000/hello-world`

7) `docker container kill registry`, `docker container rm registry`

8) docker container run -d -p 5000:5000 --name registry -v $(pwd)/registry-data:/var/lib/registry registry


<img width="762" alt="Screen Shot 2020-12-21 at 11 01 28 AM" src="https://user-images.githubusercontent.com/60768642/102731727-ec558c80-437b-11eb-91ee-0e9b3e612f2d.png">


## Quiz

1. Docker hub can configure a ( ) to trigger automatic building for other services like Jenkins, TravisCI and more.
2. What does docker prevent by default when communicating with remote registries?
3. In order to run a container from an image from a different registry than Docker Hub, you would need to specify that registry in the settings of the Docker daemon. (True or False?)

1. webhook
2. It prevents talking to any registries without HTTPS, except localhost.
3. False, We just specify and image with our custom registry, like we did with the image in the lecture 127.0.0.1:5000/hello-world.


