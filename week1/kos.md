# Swarm Intro and Creating a 3-Node Swarm Cluster

## Docker Swarm 개념

컨테이너를 사용하여 어떤 환경에서든 쉽게 배포할 수 있게 되었지만, 이로 인해 또다른 문제가 생겨났다.

- 컨테이너의 lifecycle 을 자동화하는 방법
- 컨테이너 규모를 손쉽게 조절하는 방법
- 컨테이너에 문제가 발생했을 때, 재실행하는 방법
- 컨테이너가 동작을 멈추지 않고 업데이트하는 방법
- 각각의 컨테이너에 필요한 환경변수들을 올바르게 저장하는 방법

위와 같이 다양한 문제들을 해결하기 위해 여러 docker host 들을 마치 하나인 것처럼 다룰 수 있게 해주는 오케스트레이션 도구가 개발된다.

`Swarm Mode` 와 같이 다중 컨테이너 패키지 어플리케이션을 배포하는 동안 사용되는 컨테이너, 리소스의 자동화, 정렬, 조정 및 관리를 하는 것을 **`container orchestration`** 이라고 한다.

Docker Swarm 은 2014년 시작된 Orchestration 도구를 말하는 동시에 2016년 버전 1.12 부터 시작된 Swarmkit의 기능을 말하기도 한다. 이 두 가지는 동일한 기능을 하지만 별개의 프로젝트로 존재한다.

1.12 버전 이전은 Docker와 독립되어 있었으나, 그 이후 Swarmkit은 Docker 에 built-in 되었다.

해당 강의에서는 bulit-in 된 `Docker Swarm Mode` 에 대해서 알아본다.

<br>

![](https://images.velog.io/images/monsterkos/post/c36e1dc1-32fd-43d1-8dee-bec5c3a33448/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%206.52.41.png)

`Swarm Mode` 는 Manager 와 Worker 로 구분할 수 있으며, 각 Manager는 swarm을 통해서 각 worker와 통신할 수 있는 권한을 갖는다. 접근이 허락된 Manager 만이 Worker와 통신하며 정보를 주고 받을 수 있다.

![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)

분산 시스템에서 환경을 공유하는 알고리즘을 `consensus algorithm` 이라고 하며 docker swarm 은 Raft 라는 알고리즘을 통해 swarm 안에서 환경을 공유한다. 앞서 봤던 것 처럼 Manager 는 클러스터 상태 유지, 스케줄링, Swarm mode HTTP API endpoints 제공 등의 역할을 한다.
Worker 는 Manager에 의해서 할당된 Task를 수행하는 역할을 한다.

Docker Engine 을 Swarm Mode 에 참여시키면(Docker swarm mode를 시작하게 되면), 서비스를 생성한다. 서비스는 사용자가 Dockr swarm 에게 단위 업무를 할당하는 논리적 단위로, Swarm 에 의해서 여러 Task로 분할되어 처리된다. 그리고 분할된 Task 는 각각의 컨테이너와 1:1 로 매칭된다.

![](https://docs.docker.com/engine/swarm/images/services-diagram.png)

![](https://docs.docker.com/engine/swarm/images/service-lifecycle.png)

서비스를 생성하면 사용자는 기존 docker api가 아닌 새로운 Swarm API 를 통해서 Manager와 통신하게 되고, Manager가 각각의 task를 Worker에게 할당하는 방식으로 동작한다.

## Docker Swarm 설정

```
docker info
```

![](https://images.velog.io/images/monsterkos/post/fff7ed0f-65ac-4381-91b9-760fb92846ef/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.25.12.png)

`Swarm: inactive` 가 확인된다. Swarm 을 구성하기 위해 init 해준다.

![](https://images.velog.io/images/monsterkos/post/5eaf0c23-89bf-4e33-b4dc-698e74d5aff7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.27.36.png)

![](https://images.velog.io/images/monsterkos/post/5ec7e944-762e-4727-bbf0-8e37f5020cd2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.29.26.png)

init 하는 순간 여러가지 환경이 구성된다.

```
docker node ls
```

생성된 Manager node 를 확인할 수 있다.

![](https://images.velog.io/images/monsterkos/post/74d264c7-db5f-4c0b-a576-b705dda1c518/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.31.17.png)

service 를 생성하면 아래와 같이 자동으로 name이 부여되고 서비스가 생성된다.

![](https://images.velog.io/images/monsterkos/post/59ac14ab-311d-4983-85d1-9e5ef9c1b4e3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.34.37.png)

현재 생성된 서비스를 업데이트하여 task 복사본 3개를 생성하였다.

![](https://images.velog.io/images/monsterkos/post/25a5488c-d9df-4e79-9ffc-ccc08dbcab04/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.38.18.png)

![](https://images.velog.io/images/monsterkos/post/2dc8017c-8434-43e0-80fd-d2b3b86c89c7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.39.32.png)

위와 같이 3개의 task가 생성되었고, 모두 running 중인 것을 볼 수 있다.

<br>

![](https://images.velog.io/images/monsterkos/post/f09cd19d-1293-4492-a79a-d51fd529be87/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.47.02.png)

container 하나를 삭제하면 자동으로 다른 container를 실행한다. 서비스가 항상 running 상태를 유지하도록 하는 것이다. 따라서 모든 컨테이너를 삭제시키고 싶다면 서비스 자체를 삭제시켜야 한다.

<hr>
<br>

## Creating 3-Node Swarm: Host Options

3개의 linux 환경에서 각각의 node를 swarm을 통해 관리하는 방법을 실습해본다.
이를 위해 docker machine 과 virtual box를 설치한다.
docker machine 은 하나의 호스트 머신이 아닌 여러대의 호스트 머신에서 Docker의 실행 환경을 명령으로 자동 생성하기 위한 툴 이다.

![](https://images.velog.io/images/monsterkos/post/2bf2bb7a-62b9-4fa6-9652-e1031ee6ac46/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.25.52.png)

세 개의 linux 환경을 만들고 docker swarm 을 생성한다.

![](https://images.velog.io/images/monsterkos/post/4947f347-9e54-4232-af27-8dc26685c578/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.27.03.png)

node1 에서 발급된 token 을 통해 node2 에서 swarm 에 join 한다. 최초 worker 로 swarm 에 합류한다. worker는 swarm command를 사용할 수 없다. 따라서 manager로 업데이트 해줘야 한다.

![](https://images.velog.io/images/monsterkos/post/c82294f5-2075-4474-bc6c-0fa07ada6120/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.30.17.png)

![](https://images.velog.io/images/monsterkos/post/1716a4f7-38a2-4fba-a173-c984dc29889f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.31.49.png)

swarm 에 합류하기 위한 join token 은 아래와 같이 command를 통해서 언제든지 확인할 수 있다.

![](https://images.velog.io/images/monsterkos/post/53049d94-6960-4d1b-b96b-50fe160d5bdb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-12-14%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.32.31.png)

위와 같은 방식으로 node3 도 swarm에 합류하여 Manager로 업데이트 할 수 있다.
