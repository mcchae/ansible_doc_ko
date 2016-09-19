## Getting Started with Docker

Ansible 은 Docker 컨테이너를 지휘하기 위하여 다음과 같은 모듈을 제공합니다.

* `docker_service` : 단일 다커 데몬 또는 Swarm에서 컨테이너를 지휘하기 위하여 이미 존재하는 다커 compose 파일 이용. compose 버전 1,2를 지원
* `docker_container` : 컨테이너의 생성, 변경, 멈춤, 시작, 삭제 등의 컨테이너 라이프 사이클 관리
* `docker_image` : 다커 이미지를 생성, pull, push, tag, remove 등을 통한 이미지 관리
* `docker_image_facts` : 다커 호스트 이미지 캐쉬에서 하나 이상의 이미지를 조사하여 플레이북의 fact 정보로 이용할 수 있도록 함
* `docker_login` : 다커 허브 또는 다른 다커 저장소의 인증을 위한 다커 엔진 설정 파일 갱신
* `docker (동적 인벤토리)` : 하나 이상의 다커 호스트에 사용가능한 모든 컨테이너를 이용하여 인벤토리 생성

Ansible 2.1 이후부터 위와 같은 다커 모듈이 추가되었고 컨테이너 지휘가 가능합니다. 위의 모듈 외에도 다음과 같은 것을 작업하고 있습니다.

이미지를 생성하기 위하여 *Dockerfile*을 아직도 사용하고 계십니까? [ansible-container](https://github.com/ansible/ansible-container)를 확인해 보십시오. Ansible 플레이북으로 이미지를 생성하는 것이 가능합니다.

[OpenShift](https://www.openshift.org/)에 있는 docker-compose 를 실행하기 위하여 [ansible-container](https://github.com/ansible/ansible-container)에 있는 `shipit` 명령어를 이용해 보십시오. 많은 수고를 하지 않아도 단순한 랩탑 응용프로그램을 클라우드에 확장 가능한 응용프로그램으로 전환 가능합니다.

기타 다커 관련 아이디어나 계획 등에 대해서는 [해당 저장소](https://github.com/ansible/proposals/tree/master/docker)에 있는 문서를 참고하십시오.


### 요구사항

docker를 이용하기 위해서는 [docker-py](https://docker-py.readthedocs.org/en/stable/) 1.7.0 이후 버전이 설치되어 있어야 합니다.

```sh
$ pip install 'docker-py>=1.7.0'
```

`docker_service` 모듈은 [docker-compose](https://github.com/docker/compose) 모듈도 필요로 합니다.

```sh
$ pip install 'docker-compose>=1.7.0'
```

### Docker API 연동

로컬 또는 원격 API 연결하여 패러미터를 전달하고 하는 것을 환경 변수로 할 수 있습니다. 우선 순위는 명령행에 주는 것이 우선이고 그 다음이 환경변수 입니다. 만약 명령행 인자와 환경변수 모두 발견되지 않으면 디폴트 값을 사용합니다. 

#### 패러미터

* `docker_host` : Docker API 에 접속하기 위하여 URL 또는 Unix socker 경로가 사용됩니다. 디폴트는 ***​unix://var/run/docker.sock*** 입니다. 원격 서비스에 접속하기 위해서는 ***tcp://192.0.2.23:2376*** 와 같이 TCP 연결을 합니다. 만약 TLS를 이용한 암호화된 통신을 하려면 *tcp* 대신 *https* 를 이용합니다.
* `api_version` : Docker 호스트에서 작동하는 API의 버전인데 디폴트는 docker-py가 지원하는 API의 마지막 버전 입니다.
* `timeout` : API 를 호출하고 응답을 기다리는 타임아웃(초) 입니다. 디폴트는 60초 입니다.
* `tls` : TLS 암호화 통신으로 API 연결을 하는 플래그입니다. 디폴트는 False 입니다.
* `tls_verify` : Docker 호스트 서버를 인증하지 않할지 나타내는 플래그 입니다. 디폴트는 False 입니다.
* `cacert_path` : CA 인증서 파일의 경로입니다.
* `cert_path` : TLS 인증서 파일의 경로입니다.
* `key_path` : TLS 키파일의 경로입니다.
* `tls_hostname` : Docker 호스트 서버의 유효성을 검증하는 경우 서버 이름을 지정합니다. 디폴트는 *localhost* 입니다.
* `ssl_version` : SSL 버전입니다. docker-py에 의해 결정됩니다.

#### 환경 변수

Docker API를 제어하는데 사용되는 환경변수 입니다.

* `DOCKER_HOST` : Docker API 에 접속하기 위하여 URL 또는 Unix socker 경로가 사용됩니다.
* `DOCKER_API_VERSION` : Docker 호스트에서 작동하는 API의 버전인데 디폴트는 docker-py가 지원하는 API의 마지막 버전 입니다.
* `DOCKER_TIMEOUT` : API 를 호출하고 응답을 기다리는 타임아웃(초) 입니다. 
* `DOCKER_CERT_PATH` : CA 인증서 파일의 경로입니다.
* `DOCKER_SSL_VERSION` : SSL 버전입니다.
* `DOCKER_TLS` :  TLS 암호화 통신으로 API 연결을 하는 플래그입니다.
* `DOCKER_TLS_VERIFY` : Docker 호스트 서버를 인증하지 않할지 나타내는 플래그 입니다.

#### 동적 인벤토리 스크립트

인벤토리 스크립트는 하나 이상의 Docker API 서비스를 이용하여 수집된 동적 인벤토리를 생성합니다. 이것은 정적 파일이 아닌 API를 수행하는 시간에 생성되므로 동적 개념입니다. 각각의 다커 호스트가 제공하는 컨테이너를 보고 정보를 얻어옵니다. 접속할 API 스크립트는 환경 변수 또는 설정 파일에 정의됩니다.

##### Groups

이 스크립트는 다음과 같은 호스트 그룹을 생성합니다.

- 컨테이너 ID
- 컨테이너 이름
- 컨테이너 short ID
- 이미지 이름 (*image_<이미지이름>*)
- docker_host
- running
- stopped

##### 예제

``` sh
# Connect to the Docker API on localhost port 4243 and format the JSON output
DOCKER_HOST=tcp://localhost:4243 ./docker.py --pretty

# Any container's ssh port exposed on 0.0.0.0 will be mapped to
# another IP address (where Ansible will attempt to connect via SSH)
DOCKER_DEFAULT_IP=192.0.2.5 ./docker.py --pretty

# Run as input to a playbook:
ansible-playbook -i ~/projects/ansible/contrib/inventory/docker.py docker_inventory_test.yml
```

```yaml
# Simple playbook to invoke with the above example:

    - name: Test docker_inventory
      hosts: all
      connection: local
      gather_facts: no
      tasks:
        - debug: msg="Container - {{ inventory_hostname }}"
```

