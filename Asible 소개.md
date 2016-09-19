# Asible 소개

## 설치

```sh
$ pip install ansible
```	
설치 결과는 ansible (2.0.1.0) 이고 python 2.7 및 3.5 모두 잘 설치되었습니다.

[설치하기 참조](https://docs.ansible.com/ansible/intro_installation.html)

## 시작하기

### 원격 연결

* 원격 연결은 SSH key로 연결하는 것이 일반적
* 만약 암호를 물어보는 것이 필요하면 `--ask-pass` 옵션 사용
* 만약 sudo를 이용한 암호확인 이라면 `--ask-become-pass` 이용

> **노트** : 1.3 버전 이후부터는 파이썬의 OpenSSH 를 이용하여 하였고 1.2 부터는 `paramiko`라는 파이썬 SSH 모듈을 이용

### 첫번째 명령

`/etc/ansible/hosts` 파일에 ansible 관리대상 호스트를 넣어줍니다.

```ini
192.168.2.2
192.168.2.14
192.168.2.141
```	
이 파일을 `Inventory` 라고 하며 [Inventory 상세 설명](https://docs.ansible.com/ansible/intro_inventory.html) 에 잘 나와 있습니다.

> **노트** : 이미 SSH 키로 위의 관리 대상(현재는 그냥 호스트)을 연결하였다고 가정함. `--private-key` 옵션을 이용하여 연결할 수도 있음.

이제,

```sh
$ ansible all -m ping -u future
```
명령을 내려 볼 수 있습니다.

이것은 모든 Inventory (위에서 지정한 3개의 IP) 에 대해서 ping 이라는 모듈을 실행하는데 (`-m ping`) future 라는 사용자로 접속하는 것입니다. (`-u future`) 여기서 -u 옵션은 ping 이라는 모듈의 옵션입니다. 

현재는 그 결과가

```sh
192.168.2.2 | UNREACHABLE! => {
    "changed": false, 
    "msg": "SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue", 
    "unreachable": true
}
192.168.2.14 | UNREACHABLE! => {
    "changed": false, 
    "msg": "SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue", 
    "unreachable": true
}
192.168.2.141 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

위와 같이 연결되는 것도 있고 안되는 것도 있습니다.

시스템의 ping 명령이 아닌 ssh 연결 위에 접속되는 것으로 보이는 군요.
세세히 지정하는 것은 추후에 살펴보도록 하겠습니다. 여기서는 SSH 키 교환으로 접속이 되는 것만 응답을 하는 것을 알 수 있습니다.

```sh
$ ansible all -a "/bin/echo hello" -u future
```
라고 하여 특정 명령을 수행하는 것도 가능한데 그 결과 역시

```sh
192.168.2.2 | UNREACHABLE! => {
    "changed": false, 
    "msg": "SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue", 
    "unreachable": true
}
192.168.2.14 | UNREACHABLE! => {
    "changed": false, 
    "msg": "SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue", 
    "unreachable": true
}
192.168.2.141 | SUCCESS | rc=0 >>
hello
```
라고 하여 192.168.2.141 만 응답을 하는 것을 알 수 있습니다.

## 관리 대상 목록 (Inventory)

Ansible 에서는 관리 대상 목록을 Inventory 라고 합니다. 이 대상 목록은 디폴트로 `/etc/ansible/hosts`에 정의되어 있습니다. 하지만 `-i <path>` 옵션을 이용하여 명령행에서 지정할 수도 있습니다.
이것은 고정된 것 뿐만 아니라 [동적 관리 대상 목록 (Dynamic Inventory)](https://docs.ansible.com/ansible/intro_dynamic_inventory.html)을 이용할 수 있습니다.

### 호스트 또는 그룹

`/etc/ansible/hosts` 파일 형식은 INI 파일 형식 처럼 사용할 수 있는데,

``` ini
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

와 같이 지정할 수 있습니다.

대괄호로 묶인 `[webservers]` 나 `[dbservers]`는 그룹을 의미합니다. 디폴트로 모두 SSH 통신을 통하여 접속된다고 가정합니다. 만약 SSH 연결 포트가 디폴트인 22번이 아닌 경우 다음과 같이,

```ini
badwolf.example.com:5309
```
포트를 지정할 수 있습니다.

만약 alias 와 같이 지정하고 싶다면,

```ini
jumper ansible_port=5555 ansible_host=192.168.1.50
```
와 같이 지정하면 됩니다. 이것은 jumper를 접속하게 되면 192.168.1.50:5555 로 SSH 접속을 하게 됩니다.

다음과 같이 확장도 가능합니다.

```ini
[webservers]
www[01:50].example.com
[databases]
db-[a:f].example.com
```

`www[01:50].example.com`은 `www01.example.com` 부터`www50.example.com` 까지의 50개의 관리 대상을 의미하며,
`db-[a:f].example.com`은 `db-a.example.com` 부터 `db-f.example.com` 까지의 6개의 관리 대상을 의미합니다.

> **노트** : 만약 2.0 이전 버전이라면 `ansible_user` 대신 `ansible_ssh_user`, `ansible_host` 대신 `ansible_ssh_host` 또는 `ansible_port` 대신 `ansible_ssh_port` 라고 표현됩니다.

다음과 같이 각 대상 별 연결 방법을 달리 할 수 있습니다.

```ini
[targets]
localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_user=mdehaan
```

### 호스트 변수

```ini
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

`http_port`, `maxRequestsPerChild` 와 같이 호스트 변수를 지정하여 나중에 설명할 플레이북 에서 사용할 수 있습니다.

### 그룹 변수

```ini
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

`atlanta` 그룹에서 공통으로 사용할 변수 **ntp_server**와 **proxy**를 지정하는 방법입니다.

### 그룹의 그룹, 그룹 변수
그룹 이름 다음에 *`:children`* 을 붙여 그룹의 하위 그룹을 표현할 수 있습니다. 또는 위에서 처럼 *`:vars`* 를 붙여 그룹 변수를 포현합니다. 약간 더 복잡하게 다음과 같이 표현할 수 있습니다.

```ini
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
``` 

### 특정 호스트 또는 특정 그룹의 데이터 분리
이전과 같이 특정 관리 대상 목록 파일에 변수 등의 데이터를 지정할 수도 있지만 이를 별도의 파일로 분리할 수 있습니다. 따로 떼어 놓을 변수 파일은 YAML 파일 형식입니다. ([YAML 문법 참조](https://docs.ansible.com/ansible/YAMLSyntax.html) 역자주: `키: 값` 을 지정시 꼭 콜론 다음에 공백을 하나 두도록 합니다)

```sh
/etc/ansible/hosts
```
위와 같이 관리 목록 대상 파일이 있고 `/etc/ansible/group_vars/그룹명` 의 파일을 지정하면 해당 그룹 변수를 지정합니다. 마찬가지로 `/etc/ansible/host_vars/호스트명` 의 파일을 지정하면 해당 호스트 변수를 지정합니다.

```sh
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

예를 들어 위의 `/etc/ansible/group_vars/raleigh` 파일에

```yaml
---
ntp_server: acme.example.org
database_server: storage.example.org
```
와 같이 지정하여 **ntp_server**와 **database_server** 그룹 변수를 지정하였습니다.

만약 위와 같은 경로에 해당 파일이 없는 경우 다음과 같은 선택적 대안도 존재할 수 있습니다.
예를 들어 `releigh` 그룹에 **db_settings** 및 **cluster_settings** 라는 변수가 이용된다면 다음과 같은 파일에 YAML 형식으로 해당 내용을 넣을 수 있습니다.

```sh
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```
> **노트** : 이것은 특히 특정 변수 값이 매우 크거나 할 때 유용합니다. 버전 1.4 이후에서 지원합니다.

### 접속 관련 관리 대상 목록 패러미터

ansible 의 관리 대상 목록에 있는 호스트별 접속 정보를 위한 미리 설정된 패러미터는 다음과 같은 것들이 있습니다.

#### 호스트 연결:

##### ansible_connection
호스트에 연결될 접속 형태를 의미합니다. 이것은 Ansible 연결 플러그인의 이름입니다. SSH 프로토콜 형식은 `smart`, `ssh`, `paramiko` 중에 하나가 될 수 있고 디폴트는 `smart`입니다. SSH 연결이 아닌경우에는 아래에 별도로 기술됩니다.

> **노트** : 만약 2.0 이전 버전이라면 `ansible_user` 대신 `ansible_ssh_user`, `ansible_host` 대신 `ansible_ssh_host` 또는 `ansible_port` 대신 `ansible_ssh_port` 라고 표현됩니다.

#### SSH 연결:

##### ​ansible_host
만약 주어진 이름과 달리 접속할 호스트 명을 별도로 지정한다면 이 패러미터에 기술합니다.

##### ansible_port
만약 SSH 디폴트 포트인 22번이 아니라면 이 패러미터에 지정해 줍니다.

##### ansible_user
접속할 SSH 사용자 이름을 지정합니다.

##### ansible\_ssh_pass
만약 SSH 접속시 암호를 묻는다면 이 암호를 지정해 줍니다.
> **노트** : 이것은 보안에 문제가 있을 수 있기 때문에 `--ask-pass`를 이용하거나 SSH 키를 이용하기를 권고합니다.

##### ansible\_ssh\_private\_key_file
ssh 연결에 사용할 개인키 파일을 지정합니다. 보통 `/home/user/.ssh/id_rsa` 와 같이 지정합니다.

##### ansible\_ssh\_common_args
이 패러미터는 sftp, scp 또는 ssh 명령에 추가되는 패러미터를 지정할 수 있습니다. 특정 호스트나 그룹에 ***ProxyCommand*** 를 설정하는데 유용합니다.

##### ansible\_sftp\_extra_args
이 패러미터는 sftp 명령에 추가되는 패러미터를 지정할 수 있습니다.

##### ansible\_scp\_extra_args
이 패러미터는 scp 명령에 추가되는 패러미터를 지정할 수 있습니다.

##### ansible\_ssh\_extra_args
이 패러미터는 ssh 명령에 추가되는 패러미터를 지정할 수 있습니다.

##### ansible\_ssh_pipelining
SSH 파이프라인을 이용할 것이가를 나타냅니다. 이것은 `ansible.cfg`의 글로벌 설정에 있는 ***pipelining*** 설정에 우선합니다.

#### 권한상승: [권한 상승 문서 참조]()

##### ansible_become
***ansible_sudo*** 또는 ***ansible_su*** 명령과 동일합니다.

##### ansible\_become_method
권한 상승 방법을 지정합니다.

##### ansible\_become_user
***ansible_sudo_user*** 또는 ***ansible_su_user*** 와 동일합니다.
> **노트** : 이것은 보안에 문제가 있을 수 있기 때문에 `--ask-become-pass`를 이용하거나 SSH 키를 이용하기를 권고합니다.

#### 원격 호스트 환경 관련 패러미터:

##### ansible\_shell_type
타겟 시스템의 쉘을 지정합니다. ***ansible_shell_executable*** 을 본쉘 호환이 되지 않은 것을 특별히 지정하지 않은한 이 패러미터는 사용하면 안됩니다. 디폴트로 sh 스타일의 문법을 이용한 명령 형식을 디폴트로 사용합니다. 만약 이 패러미터를 `csh` 또는 `fish` 로 지정하면 해당 쉘 명령 형식을 따릅니다.

##### ansible\_python_interpreter
타겟 호스트의 파이썬 인터프리터 경로를 지정합니다. 

##### ansible\_*_interpreter
`ansible_python_interpreter`와 유사하게 ruby, perl 등의 인터프리터 경로를 지정합니다. 이 경우 파이썬이 아닌 해당 인터프리터로 구성된 모듈을 구동시킵니다.

#### 버전 2.1에 추가된 패러미터

##### ansible\_shell_executable
이 패러미터는 `ansible.cfg` 글로벌 설정파일에 있는 `executable` 의 **/bin/sh** 대신 타겟 머신에서 제어되는 명령의 쉘을 지정합니다. **/bin/sh** 에서 지원되지 않는 경우에만 사용하기 바랍니다.

#### 호스트 파일 사용 예

```ini
some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
```

### SSH 연결이 아닌 경우

Ansible은 SSH 연결 위에서 플레이북을 실행시키도록 되어 있지만 이 방식 이외의 연결도 가응합니다. ***ansible_connection=\<connector>*** 패러미터에 의해 변경할 수 있습니다. 다음과 같은 비 SSH 연결이 가능합니다.

#### local
ansible을 수행하는 머신 자체에서 실행되는 경우입니다.

#### docker
이 연결은 로컬에 있는 docker 클라이언트를 이용하여 다커 컨테이너에 직접 playbook을 실행시킵니다. 이 커넥터에는 아래와 같은 패러미터가 지원됩니다.

##### ansible_host
연결할 다커 컨테이너의 이름을 지정합니다.

##### ansible_user
컨테이너에서 운영될 사용자를 지정합니다. 물론 해당 사용자는 컨테이너에서 미리 존재해야 합니다.

##### ansible_become
만약 `​become_user` 가 `true`로 설정된다면 컨테이너에서 동작합니다.

##### ansible\_docker\_extra_args
다커가 인식할 수 있는 추가적인 아규먼트를 문자열로 표현한 것입니다. 이 패러미터는 주로 원격 다커 데몬을 다루는 설정에 이용될 수 있습니다.

다음은 컨테이너를 배포하는 것에 대한 예제입니다.

```yaml
- name: create jenkins container
  docker:
    name: my_jenkins
    image: jenkins

- name: add container to inventory
  add_host:
    name: my_jenkins
    ansible_connection: docker
    ansible_docker_extra_args: "--tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/client-cert.pem --tlskey=/path/to/client-key.pem -H=tcp://myserver.net:4243"
    ansible_user: jenkins
  changed_when: false

- name: create directory for ssh keys
  delegate_to: my_jenkins
  file:
    path: "/var/jenkins_home/.ssh/jupiter"
    state: directory
```

# 동적 Invntory

설정관리 시스템의 사용자는 종종 설정 관리 목록 (Inventory)을 다른 소스트웨어 시스템에 보관하기를 원할 수 있습니다. 클라우드 제공자, LDAP, [Cobbler](http://cobbler.github.io) 또는 CMDB 소프트웨어 등과 같은 곳에 저장할 경우도 있습니다.

Ansible은 이런 모든 외부 저장소를 쉽게 연동할 수 있습니다. 이미 `contrib/inventory` 디렉터리에 이미 되어 있는 것도 있으나 아래에서는 EC2/Eucalyptus, Rackspace Cloud, 및 OpenStack 등과 연동되는 것을 확인해 보도록 하겠습니다.

[Ansible Tower](https://docs.ansible.com/ansible/tower.html) 또한 Inventory를 관리하는 DB를 가지고 있으며 REST API를 제공하고 웹으로 관리할 수 있도록 되어 있습니다. Inventory 편집기를 별도 가지고 있습니다. 설정관리에 대한 추적도 가능합니다. 플레이북을 실행하면서 어디에서 오류가 발생했는지 추적도 가능합니다.



## 예: Cobbler 외부 관리 대상 목록 스크립트

`/etc/ansible/cobbler.ini` 파일에 다음과 같이 지정할 수 있습니다.

```ini
[cobbler]

# Set Cobbler's hostname or IP address
host = http://127.0.0.1/cobbler_api

# API calls to Cobbler can be slow. For this reason, we cache the results of an API
# call. Set this to the path you want cache files to be written to. Two files
# will be written to this directory:
#   - ansible-cobbler.cache
#   - ansible-cobbler.index

cache_path = /tmp

# The number of seconds a cache file is considered valid. After this many
# seconds, a new API call will be made, and the cache file will be updated.

cache_max_age = 900
```

***/etc/ansible/cobbler.py*** 스크립트를 바로 실행할 수 있습니다. 

다음과 같은 식으로 실행될 수 있습니다.

```sh
cobbler profile add --name=webserver --distro=CentOS6-x86_64
cobbler profile edit --name=webserver --mgmt-classes="webserver" --ksmeta="a=2 b=3"
cobbler system edit --name=foo --dns-name="foo.example.com" --mgmt-classes="atlanta" --ksmeta="c=4"
cobbler system edit --name=bar --dns-name="bar.example.com" --mgmt-classes="atlanta" --ksmeta="c=5"
```

## 예: AWS EC2 외부 관리 대상 목록 스크립트

[EC2 external inventory scritp 참조](https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py)

> TODO : 자세한 내용은 [원본 문서 페이지](https://docs.ansible.com/ansible/intro_dynamic_inventory.html)를 참조하시기 바랍니다.


# 패턴

Ansible에서 패턴이라는 것은 관리하기 위한 호스트를 결정하는 것입니다. 이것은 통신할 호스트들을 의미하지만 플레이북 관점에서는 실제 무언가 명령을 실행하고 로직을 처리하는 플레이하는 대상의 호스트(장비)를 의미합니다.

기본적인 실행 패턴입니다.

```sh
ansible <pattern_goes_here> -m <module_name> -a <arguments>
```

예,

```sh
$ ansible webservers -m service -a "name=httpd state=restarted"
```

패턴은 호스트 뿐만 아니라 그룹이 될 수도 있습니다.

만약 패턴에 다음과 같이,

```ini
all
*
```
라고 주었다면 이것은 관리 대상 목록 (Inventory)에 있는 모든 호스트를 의미합니다.

또는 호스트나 IP 주소를 직접 지정할 수도 있습니다.

```ini
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
```

아래와 같이 하나의 그룹 또는 여러개의 그룹을 지정할 수도 있습니다.

```ini
webservers
webservers:dbservers
```

만약 다음과 같이 지정한다면

```ini
webservers:! phoenix
```
이것은 `webservers` 그룹에는 있으나 `phoenix` 그룹에는 없는 호스트만을 의미합니다.

```ini
webservers:&staging
```
반면 위에 내용은 `webservers`와 `staging` 그룹에 모두 포함된 호스트를 의미합니다.

여러 조합도 가능한데,

```
webservers:dbservers:&staging:!phoenix
```
위의 패턴은 ​`webservers` 또는 `dbservers` 그룹에 포함되는 모든 호스트 중에서 `staging` 그룹에 있으면서 `phoenix` 그룹에는 존재하지 않는 호스트를 의미합니다.

```
*.example.com
*.com
```
와일드카드 `*`를 사용하여 호스트명을 지정할 수 있으면,

```
one*.com:dbservers
```
와일드카드를 사용한 호스트명 뿐만 아니라 그룹명을 같이 사용해도 됩니다.


그룹에 있는 호스트 중 일부만 선택할 수도 있습니다.

예를 들어,

```ini
[webservers]
cobweb
webbing
weber
```
`webservers` 그룹에 위와  같이 세개의 호스트가 있다면

```ini
webservers[0]       # == cobweb
webservers[-1]      # == weber
webservers[0:1]     # == webservers[0],webservers[1]
                    # == cobweb,webbing
webservers[1:]      # == webbing,weber
```
위와 같이 색인을 통하여 선택할 수도 있습니다.

```ini
~(web|db).*\.example\.com
```
심지어는 위와 같이 패턴에 정규식을 지정하는 것도 가능합니다.


# 애드-혹 명령어 소개

애드-혹 명령은 필요시 한번만 수행하고 이후 동일한 수행을 위하여 저장할 필요가 없는 명령입니다.
플레이북 실행에 앞서 간단히 ansible을 할 수 있는 기능을 살펴보는데 도움이 될 수 있습니다.
예를 들어 랩의 전체 휴가에 앞서 모든 호스트를 일괄적으로 끄는데 있어 플레이북을 작성하는 것 보다는 애드-혹 명령을 내리는 것이 더 간단합니다.

## 병렬화 및 쉘 명령어

```sh
$ ansible atlanta -a "/sbin/reboot" -f 10
```
위의 명령어는 간단히 atlanta 그룹에 있는 호스트에 `​/sbin/reboot` 명령을 내리는데 `-f 10` 옵션으로 10개 까지 동시성을 제공한다는 것입니다. 여기서 동시성이라는 것은 동시에 10개씩의 호스트에 접속하여 명령을 내린다는 의미입니다.

위의 명령은 root 권한만 가능한데 연결은 특정 SSH 사용자로 연결하는 경우가 많으므로

```sh
$ ansible atlanta -a "/usr/bin/foo" -u username --become --ask-become-pass
```
위와 같이 할 수 있습니다.

```sh
$ ansible raleigh -m shell -a 'echo $TERM'
```
애드-혹 명령을 수행하는데 위에서 처럼 특정 모듈 `-m shell`을 지정할 수 있습니다. (실은 `-m`이 생략되면 디폴트 모듈인 `command` 모듈이 실행되며 해당 모듈의 수행 명령으로 `-a ...` 이 동작하게 됩니다.)

## 파일 전송
 
 관리 대상 호스트의 파일에 대하여 파일 전송을 하려면 
 
 ```sh
 $ ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"
 ```
 `copy` 모듈을 이용하면 됩니다.

 
``` sh
$ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600"
$ ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
```
`file` 모듈을 이용하여 파일 퍼미션이나 속성을 변경할 수 있습니다. 
> **노트** : `file` 모듈에서 사용되는 옵션은 `copy` 모듈에서도 동일하게 적용됩니다.

```sh
$ ansible webservers -m file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
```
`file` 모듈은 `mkdir -p` 명령과 같이 해당 폴더를 생성할 수 있습니다.

```sh
$ ansible webservers -m file -a "dest=/path/to/c state=absent"
```
반대로 해당 디렉터리를 삭제할 수 있습니다.

## 패키지 관리

레드햇 계열의 리눅스에 패키지 관리하는 yum 및 데비안,우분투 계열의 패키지 관리하는 apt (aptitude)를 사용하는 모듈을 이용할 수 있습니다.

```sh
$ ansible webservers -m yum -a "name=acme state=present"
```
`yum` 모듈로 **acme** 패키지를 설치하는데 패키지 update는 하지 않습니다.

```sh
$ ansible webservers -m yum -a "name=acme-1.5 state=present"
```
위와 같이 특정 버전을 지정할 수 있습니다.

```sh
$ ansible webservers -m yum -a "name=acme state=latest"
```
가장 최신버전의 패키지를 설치합니다.

```sh
$ ansible webservers -m yum -a "name=acme state=absent"
```
해당 패키지 `acme`가 설치되어 있지 않은지 확인합니다.

## 사용자 및 그룹

```sh
$ ansible all -m user -a "name=foo password=<crypted password here>"
$ ansible all -m user -a "name=foo state=absent" 
```
리눅스의 사용자와 그룹과 관련된 명령을 내릴 수 있습니다.

## 소스 버전 관리 시스템에서 배포

```sh
$ ansible webservers -m git -a "repo=git://foo.example.org/repo.git dest=/srv/myapp version=HEAD"
```
***git*** 버전 관리를 이용하여 관리 대상 호스트에 해당 소스를 가져올 수 있습니다.

Ansible 모듈은 버전관리 시스템에서 해당 코드가 업데이트되었을 때 알 수 있는 변경 핸들러를 통해 알 수 있으므로 git에서 해당 프로그램이나 코드를 가져오고 아파치 서버를 재기동 하는 등의 일을 할 수 있습니다.

## 서비스 관리

```sh
$ ansible webservers -m service -a "name=httpd state=started"
```
`httpd`서비스가 시작되었는가 확인하고 안되어 있으면 시작합니다.

```sh
$ ansible webservers -m service -a "name=httpd state=restarted"
```
유사하게 재기동 시킵니다.

```sh
$ ansible webservers -m service -a "name=httpd state= stopped"
```
해당 서비스를 멈춥니다.

## 시간 제한을 갖는 백그라운드 작업
오래 걸리는 작업은 일단 백그라운드로 동작하게 한 다음 일정 시간이 지나 그 상태를 조사할 수 있습니다. 
예를 들어 `long_running_operation` 라는 작업을 비 동기적으로 백그라운드에서 실행시키고 3600초의 시간 (`-B 3600`)을 갖고 폴링하지 않는 (`-P 0`) 경우,

```sh
$ ansible all -B 3600 -P 0 -a "/usr/bin/long_running_operation --do-stuff"
```
위와 같이 설정할 수 있습니다.

만약 백그라운드로 실행만 시켰을 경우에는 해당 작업 ID를 가지고 다음과 같이,

```sh
$ ansible web1.example.com -m async_status -a "jid=488359678239.2844"
```

`async_status` 모듈을 이용하여 이후 상태를 조사할 수 있습니다.

그것이 아니면 폴링 방식이 디폴트이며,

```sh
$ ansible all -B 1800 -P 60 -a "/usr/bin/long_running_operation --do-stuff"
```

위 명령은 백그라운드로 최대 30분(1800초)를 돌고 나서 매 1분(60초) 마다 폴링을 하면서 그 결과를 확인합니다.


## Fact 모음
fact는 플레이북 입장에서 해당 시스템에서 사용가능한 변수등을 나타내는 것입니다. 이것은 작업을 실행하는데 있어 조건 제어문을 수행하는 사용될 뿐만 아니라 해당 시스템의 애드-혹 정보를 구할 수도 있습니다.

```sh
$ ansible all -m setup
```


# 설정 파일

ansible 설정 파일은 다음과 같은 순서로 찾습니다.

* ANSIBLE_CONFIG (an environment variable)
* ansible.cfg (in the current directory)
* .ansible.cfg (in the home directory)
* /etc/ansible/ansible.cfg


## 가장 최근의 설정 구하기

`/etc/ansible` 디렉터리 안에 **.rpmnew** 와 같이 마지막 설정파일이 있을 수 있습니다.


## 환경 변수로 설정

`ANSIBLE_CONFIG` 환경변수로 설정할 수도 있습니다.


## 환경 설정 내용

### 디폴트 설정
`[default]` 부분에 있는 설정 내용입니다.

#### ​action_plugins
액션은 모듈 실행, 템플릿 실행 등과 같은 일을 하기 위한 일련의 코드입니다.
이런 액션 플러그인은 필요시 해당 코드가 있는 플러그인 폴더를 지정합니다.

```ini
action_plugins = ~/.ansible/plugins/action_plugins/:/usr/share/ansible_plugins/action_plugins
```

#### allow\_world\_readable_tmpfiles
> 버전 2.1에 추가됨

이 항목은 임시 파일이 관리 대상 머신에 생성되도록 하여 작업을 실패로 나타내는 대신 경고로 뜨도록 하는데 권한을 가지고 있지 않은 사용자로 작업할 때 유용합니다.

```ini
allow_world_readable_tmpfiles=True
```

#### ansible_managed

이 문자열은 다른 곳에서 사용될 때 치환될 수 있습니다.

```ini
{{ ansible_managed }}
```
와 같이 어디선가 사용되었고,

ansible 설정에,
```ini
ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host}
```
라고 되어 있다면 이 문자열이 치환됩니다.

#### ask_pass
이것은 플레이북이 실행될 때 암호를 물을지 안 물을지 나타내는 플래그입니다. 디폴트는 묻지 않습니다.

```ini
ask_pass=True
```
> 만약 보안을 위하여 모두 SSH 키로 접속한다면 이 항목은 False 로 놓아야 합니다.

#### ask\_sudo_pass
Ansible이 플레이북을 통하여 관리 대상 장비에서 `sudo` 명령을 내렸고 이 경우 암호를 물어볼지 안 물어볼지를 나타내는 플래그입니다. 디폴트는 묻지 않습니다.

```ini
ask_sudo_pass=True
```

#### ask\_vault_pass

Ansible에서 볼트(금고)는 암호 등의 알려지면 안되는 민감한 정보를 담고 있는 공간입니다. 이 볼트 암호를 물어볼지 안물어볼지 나타내는 플래그입니다. 디폴트는 안물어봅니다.

```sh
ask_vault_pass=True
```

#### bin\_ansible_callbacks
> 버전 1.8 이후

ansible 이 기동될 때 콜백 플러그인 들을 불러올지 아닐지를 나타내는 플래그입니다. ansible 명령에서 로그를 남기거나 결과를 알려주는 등의 역할을 수행한다면 이 플래그를 True로 놓습니다. (하지만 로드하는데 시간은 더 걸립니다) 하지만 ` /usr/bin/ansible-playbook`은 이 플래그에 상관없이 항상 콜백 플러그인을 로드합니다.


#### callback_plugins
특정 이벤트 발생 시 호출되거나 어떤 알림에 의한 수행 등을 하는 코드를 ansible에서는 콜백이라 합니다. 이런 콜백 플러그인이 있는 디렉터리 위치를 나타내는 항목입니다.

```ini
callback_plugins = ~/.ansible/plugins/callback:/usr/share/ansible/plugins/callback
```

#### stdout_callback
> 2.0에서 추가

이 설정은 ansible-playbook 에서 stdout 에 대한 기본 콜백을 변경합니다.

```ini
stdout_callback = skippy
```

#### callback_whitelist
> 2.0에서 추가

현재 바로 사용가능한 모든 콜백 플러그인이 이미 존재하지만 특별히 지정하지 않는한 사용할 수 없습니다. 이 설정은 사용가능한 콜백 플러그인을 지정하도록 합니다.

```ini
callback_whitelist = timer,mail
```


#### command_warnings
> 1.8에서 추가

아래와 같이

```yaml
- name: usage of git that could be replaced with the git module
  shell: git update foo warn=yes
```

플레이북 설정에서 shell 명령으로 `git update foo` 명령을 내렸을 때 해당 명령 결과의 경고가 나타날 수 있는데 이것을 디폴트로 보이게 할 것인지 아닌지를 나타내는 플래그 입니다.

하지만 위와 같이 개별 명령 뒤에 warn=yes 등으로 전체 설정을 덮어쓸 수 있습니다.


#### connection_plugins

연결 플러그인은 ansible에서 명령이나 파일을 전달하기위한 채널을 확장하는 사용되는 것입니다.

```ini
connection_plugins = ~/.ansible/plugins/connection_plugins/:/usr/share/ansible_plugins/connection_plugins
```

#### deprecation_warnings
> 1.3에서 추가

`ansible-playbook` 명령의 결과에 예전 명령에 대한 경고 (deprecating warning)를 보이게 할지 아닐지를 나타내는 플래그입니다.

```ini
deprecation_warnings = True
```

#### display\_args\_to_stdout
> 2.1.0에서 추가

기본적으로 `ansible-playbook`은 각각의 작업에 대한 헤더 정보를 stdout에 보여줍니다. `name:` 이나 `shell:`과 같은 설정 헤더입니다. (YAML에서의 키 값이죠) 해당 명령을 수행할 때 그 다음 명령을 함께 stdout으로 출력하게 하거나 아니거나를 나타내는 플래그입니다.

```ini
display_args_to_stdout=False
```

#### display\_skipped_hosts

```ini
display_skipped_hosts=True
```


#### error\_on\_undefined_vars

```ini
error_on_undefined_vars=False
```
만약 이 플래그가 false 이면 `{{ template_expression }}` 와 같이 확장될 변수가 존재하지 않아도 그대로 출력됩니다. 만약 True 이고 확장될 변수가 존재하지 않으면 오류가 발생합니다.


#### executable
sudo 명령을 한 다음 수행될 쉘을 나타냅니다.

```ini
executable = /bin/bash
```


#### filter_plugins
필터는 템플릿에 이용되는 특별한 함수입니다. 이런 필터 플러그인 폴더를 지정합니다.

```ini
filter_plugins = ~/.ansible/plugins/filter_plugins/:/usr/share/ansible_plugins/filter_plugins
```

#### force_color
TTY 가 없이도 수행될 때 컬러 모드를 켤지를 나타냅니다.

```ini
force_color = 1
```

#### force_handlers
> 1.9.1에서 추가

대상 호스트에서 수행 시 오류가 발생했더라도 알림 핸들러가 수행될지를 나타내는 플래그 입니다.

```ini
force_handlers = True
```


#### forks

이것은 관리 대상 호스트가 여러개가 있고 동일한 명령을 내리는데 동시에 얼마만큼의 호스트에 적용하는 가를 나타냅니다. 디폴트는 5 입니다. 일반적으로 50 내지 500 이상 까지 설정하는 경우도 있습니다.
디폴트 5는 너무 작게 잡아놓은 것입니다.

```ini
forks=5
```

#### gathering
버전 1.6 부터 원격 시스템에서 발견된 변수를 모으는 팩트에 관한 정책을 가리키는 항목입니다.
***implicit***가 디폴트이며 이것은 `gather_facts: False`가 지정되어 있지 않는한 캐쉬를 무시하고 무조건 해당 정보를 모읍니다. ***smart*** 라는 값을 주면 해당 호스트의 팩트가 존재하지 않은 경우 가져오려고 하고 캐쉬에 있다면 그 정보를 이용합니다.

```ini
gathering = smart
```

> 2.1 부터는 아래와 같이 변경됨

```ini
gather_subset = all
```

- **all** : 모든 팩트를 모음 (디폴트)
- **network** : 네트워크 팩트를 모음
- **hardware** : 하드웨어 팩트를 모음
- **virtual** : 대상 머신에 호스트되고 있는 가상머신 팩트를 모음
- **ohai** : ohai 에서 팩트 모음
- **facter** : facter 에서 팩트 모음


```ini
# Don't gather hardware facts, facts from chef's ohai or puppet's facter
gather_subset = !hardware,!ohai,!facter
```

위와 같이 콤마로 목록을 줄 수도 있고 `!`을 이용하여 제외하고 의미를 가질 수도 있습니다.

만약 아주 기본적인 것을 제외한 모든 팩트를 가져오지 않으려면,

```ini
gather_subset = !all
```
라고 하면 됩니다.


#### hash_behaviour

```ini
hash_behaviour=replace
```


#### hostfile
> 1.9 부터 사용되지 않음. 대신 ***inventory*** 항목 이용


#### host\_key_checking

```ini
host_key_checking=True
```

#### inventory

관리 대상 목록을 지정하는 파일 및 스크립트 또는 디렉터리 등의 위치를 지정합니다.

```ini
inventory = /etc/ansible/hosts
```

#### jinja2_extensions

```ini
jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n
```

#### library
Ansible이 디폴트로 모듈을 찾는 위치를 지정합니다.

```ini
library = /usr/share/ansible
```

콜론을 주어 여러 경로를 같이 줄 수 있습니다. 또한 `./library` 폴더를 같이 찾습니다.


#### local_tmp
>2.1 이후

Ansible이 관리 대상 장비에서 명령을 수행하거나 모듈을 수행할 때, 수행할 모듈의 내용 등이 내려가서 해당 모듈을 수행합니다. 해당 명령이 종료될 때 내려갔던 모듈도 삭제됩니다. 이 항목은 이런 임시로 수행되는 폴더를 지정합니다.

```ini
local_tmp = $HOME/.ansible/tmp
```

Ansible은 이 해당 폴더 안에서 임의의 하위 디렉터리를 만들어 작업합니다.


#### log_path

```ini
log_path=/var/log/ansible.log
```

#### lookup_plugins

```ini
lookup_plugins = ~/.ansible/plugins/lookup_plugins/:/usr/share/ansible_plugins/lookup_plugins
```

#### module\_set_locale

이 플래그가 활성화되면 원격 호스트에서 LANG, LC_MESSAGES, 및 LC_ALL 등이 설정됩니다.

> 2.1에서는 디폴트로 True 이고 2.2 에서는 디폴트로 False 입니다.


####  module_lang

모듈을 수행할 디폴트 LANG을 지정합니다.

```ini
module_lang = en_US.UTF-8
```

#### module_name

Ansible 에 `-m ...` 을 지정하지 않아도 수행할 디폴트 모듈을 나타냅니다.

```ini
module_name = command
```

#### nocolor

```ini
nocolor=0
```

#### nocows
`cowsay` 명령을 수행할 것인가를 나타내는 항목입니다.

```ini
nocows=0
```

#### pattern

```ini
hosts=*
```

#### poll_interval

```ini
poll_interval=15
```

#### private\_key_file

```ini
private_key_file=/path/to/file.pem
```

#### remote_port

```ini
remote_port = 22
```

#### remote_tmp

```ini
remote_tmp = $HOME/.ansible/tmp
```

#### remote_user

```ini
remote_user = root
```

#### retry\_files_enabled
Ansible에서 플레이북을 수행하다 실패하면 `.retry` 파일을 만듦니다.
이 파일을 만들 것인가 아닌가를 나타내는 플래그 입니다.

```ini
retry_files_enabled = False
```

#### retry\_files\_save_path

```ini
retry_files_save_path = ~/.ansible/retry-files
```

#### roles_path

`/etc/ansible/roles` 디폴트 역할 폴더 이외에 역할 폴더를 지정하는 플래그입니다.

```ini
roles_path = /opt/mysite/roles:/opt/othersite/roles
```

#### squash_actions
> 2.0 부터

```ini
squash_actions = apk,apt,dnf,package,pacman,pkgng,yum,zypper
```

#### strategy_plugins

```ini
strategy_plugins = ~/.ansible/plugins/strategy_plugins/:/usr/share/ansible_plugins/strategy_plugins
```

#### sudo_exe

```ini
sudo_exe=sudo
```

#### sudo_flags

```ini
sudo_flags=-H -S -n
```

#### sudo_user

```ini
sudo_user=root
```

#### system_warnings

```ini
system_warnings = True
```

#### timeout

SSH 타임아웃 시간(초) 입니다.

```ini
timeout = 10
```

#### transport

`ansible` 이나 ansible-playbook 에서 -c <transport_name> 을 지정하지 않는 한 사용할 전송 모듈입니다. 디폴트는 `smart`인데 만약 `ssh`로 충분하면 `ssh` 모듈로 하고 아니면 `paramiko` 전송 모듈을 사용합니다.

다른 전송 모듈로는 `local`, `chroot`, `jail` 등이 있습니다.


#### vars_plugins

```ini
vars_plugins = ~/.ansible/plugins/vars_plugins/:/usr/share/ansible_plugins/vars_plugins
```

#### vault_password_file
> 1.7 이후

```ini
vault_password_file = /path/to/vault_password_file
```

### 권한 상승 관련 설정

#### become

```ini
become=True
```

#### become_method

```ini
become_method=su
```

#### become_user

```ini
become_user=root
```

#### become\_ask_pass

```ini
become_ask_pass=True
```

#### become\_allow_same\_user

대부분의 경우 동일한 사용자가 `sudo` 명령을 일정시간내에 반복하면 암호를 물어보지 않습니다. 그러나 SELinux 등에서 *sudo* 명령 설정에 따라 이런 기능이 불가능할 수도 있습니다. 이런 경우 이 플래그를 True로 설정하면 됩니다.


### Paramiko 관련 설정

Paramiko 모듈은 Enterprise Linux 6 또는 그 이전 버전에 디폴트 SSH 연결을 위하여 사용되었습니다.  `[paramiko_connection]` 섹션에 다음과 같은 설정이 올 수 있습니다.

#### record_host_keys
최초 SSH 로 원격 호스트에 접속하면 host_key를 `~/.ssh/known_hosts 등`에 저장합니다. 이것은 보안 때문에 그 서버가 동일한 서버인지 매번 SSH 접속시 체크합니다. 하지만 너무 많은 수의 원격 호스트가 있다면 이 과정에서 많은 시간을 소요할 수 있으므로 이 항목 및 호스트 키 체크 항목을 False로 놓으면 더 빨라질 수 있습니다. (그만큼 보안은 더 떨어집니다)

```ini
record_host_keys=True
```

#### proxy_command 
> 2.1 에서 추가

실제 접속에 앞서 proxy 에 접속하도록 하는 명령을 내릴 수 있습니다.

```ini
proxy_command = ssh -W "%h:%p" bastion
```


### OpenSSH 관련 설정
`​[ssh_connection]` 섹션 안에 설정되는 항목들 입니다.


#### ssh_args

```ini
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```
성능향상을 위해서 *ControlPersist*를 30분까지 지정하기도 합니다. 만약 이 항목이 설정되었다면 `control_path` 설정은 사용되지 않습니다.


#### control_path
**ControlPath** 소켓을 저장할 위치 입니다. 디폴트는

```ini
control_path=%(directory)s/ansible-ssh-%%h-%%p-%%r
```
입니다.

특정 시스템에서 일반적인 소켓 이름 길이 (108자)를 넘어서면 오류가 발생할 수 있으므로 이런 경우에는,

```ini
control_path = %(directory)s/%%h-%%r
```
와 같이 지정하기도 합니다.

Ansible 1.4 이후부터는 `-vvvv` 옵션을 통해서 Contol Path 파일이름까지 확인할 수 있습니다.


#### scp_if_ssh
원격 시스템에 따라서 SSH는 지원하지만 SFTP를 지원하지 않는 경우도 있습니다. 이런 경우에만 True로 설정하기 바랍니다.


#### pipelining
파이프라이닝을 활성화 시키면 실제 원격 서버에서 수행되는 모듈을 구동시키는데 SSH 명령이 많이 줄어듭니다. 하지만 **sudo:** 명령을 만난다면 `/etc/sudoers` 파일에서 requiretty 를 빼야 정상적으로 동작 합니다. 
따라서 이 플래그는 sudo 명령 호환성을 위하여 디폴트로 `False`로 되어 있지만 속도를 위해서는 활성화 시키고 requiretty 를 비활성화 시키시기 바랍니다. (가속 모드와는 별도입니다)


### 가속 모드 설정
`[accelerate]` 섹션에 설정되는 것으로 가속 모드와 관련된 것입니다. **가속**은 파이프라이닝을 사용할 수 없는 경우 수행 속도를 높일 수 있는 것이지만 가능하면 사용안하는 편이... (???)

#### accelerate_port
> 1.3 이후

```ini
accelerate_port = 5099
```

#### accelerate_timeout
> 1.4 이후

```ini
accelerate_timeout = 30
```

#### accelerate\_connect_timeout
> 1.4 이후

```ini
accelerate_connect_timeout = 30
```

#### accelerate\_daemon_timeout
> 1.6 이후

```ini
accelerate_daemon_timeout = 30
```

#### accelerate\_multi_key
> 1.6 이후

```ini
accelerate_multi_key = yes
```

### SELinux 관련 설정

#### special\_context_filesystems
> 1.9 이후

```ini
special_context_filesystems = nfs,vboxsf,fuse,ramfs,myspecialfs
```

#### libvirt\_lxc_noseclabel
> 2.1 이후

```ini
libvirt_lxc_noseclabel = True
```

### Galaxy 설정
`[galaxy]' 세션에 정의되는 설정입니다.

#### server
[디폴트 갤럭시 서버](https://galaxy.ansible.com) 대신 사용할 서버를 지정합니다. 

#### ignore_certs
갤럭시 접속 시 TLS 인증서 확인을 하지 않습니다.


# BSD 지원
[원본](https://docs.ansible.com/ansible/intro_bsd.html) 참조하시기를...


# 윈도우 지원
[원본](https://docs.ansible.com/ansible/intro_windows.html) 참조하시기를...


# 네트워킹 (장비) 지원

## 네트워킹 장비 연동

Ansible 버전 2.1 이후부터 서로 다른 이기종 네트워크 장비를 지원합니다.

- Arista EOS
- Cisco NXOS
- Cisco IOS
- Cisco IOSXR
- Cumulus Linux
- Juniper JUNOS
- OpenSwitch

## 네트워크 자동화 설치

다음과 같은 설치가 필요합니다.


- [가장 최근 버전의 Ansible network release](http://docs.ansible.com/ansible/intro_installation.html) 설치
- [테스트용 네트워크 플레이북](https://github.com/ansible/test-network-modules)을 다운

## 네트워크 장치에도 사용가능한 모듈

대부분의 표준 Ansible 모듈은 Linux/Unix 또는 윈도우 머신과 동작하도록 되어 있기 때문에 네트워크 장치에는 사용할 수 없습니다만 `slurp`, `raw` 또는 `setup` 등과 같이 플랫폼을 타지 않는 모듈은 네트워크 장치에도 사용가능합니다.

네트워크 장치에 사용할 수 있는 모듈이 어떤 것들이 있는지 알기 위해서 [네트워크 장치 이용가능 모듈 색인](http://docs.ansible.com/ansible/list_of_network_modules.html#)을 참조하십시오.


## 네트워크 장치 연결

모든 핵심 네트워킹 모듈은 `provider` 라는 인자를 갖는데, 이것은 어떻게 장치에 연결되는가를 나타내는 특성을 나타냅니다.

각 핵심 네트워크 모듈은  해당 운영체제 지원과 전송 기능을 제공합니다. 운영 체제는 이런 모듈과 일대일 대응을 하며 전송 기능은 운영 체제와 일대다 관계로 구성됩니다.

각 핵심 네트워크 모듈은 전송기능을 담당하는 다음과 같은 기본 인자를 갖습니다.

- host : 원격 장비의 호스트명 또는 IP주소
- port : 연결되는 포트 번호
- username : 연결에 사용되는 사용자 ID
- password : 연결 사용되는 사용자 암호
- transport : 어떤 전송 형태인지 지정
- authorize : 장비에서의 권한 상승 방법
- auth_pass : 권한 상승 시 필요에 따른 함호

개별 모듈은 위와 같은 인자에 대하여 디폴트 값을 가질 수 있습니다. 예를 들어 일반적인 전송 방법은 ***CLI*** 입니다. 어떤 모듈은 `EOS(eapi)` 또는 `NXOS(nxapi)` 전송 방법을 지원하는 반면 어떤 모듈은 `CLI`만 지원하기도 합니다. 모든 인자는 각 모듈에서 자세히 설명합니다.

위와 같은 전송 인자를 설정하는 것은 개별적으로 가능하므로 각 모듈은 원하는 데로 서로 다른 전송 방법이나 인증 방식을 사용할 수 있습니다.

이와 같이 접속하는 방법의 한가지 단점은 모든 작업이 필요한 인자를 꼭 포함해야 한다는 것입니다. 이런 이유 때문에 `provider` 인자가 필요하게 되었습니다. `provider` 인자는 키워드 인자를 받아 접속 및 인증에 필요한 패러미터를 해당 작업으로 전달해 줍니다.

다음 두 개의 설정은 동일한 nxos_config 모듈을 사용하지만 모든 핵심 네트워크 모듈에도 적용됩니다.

```yaml
---
nxos_config:
   src: config.j2
   host: "{{ inventory_hostname }}"
   username: "{{ ansible_ssh_user }}"
   password: "{{ ansible_ssh_pass }}"
   transport: cli
```

```yaml
---
vars:
   cli:
   host: "{{ inventory_hostname }}"
   username: "{{ ansible_ssh_user }}"
   password: "{{ ansible_ssh_pass }} "
   transport: cli


nxos_config:
   src: config.j2
   provider: "{{ cli }}"
```
 
위의 두개의 예제는 동일하지만 인자가 우선권 및 디폴트 등을 지정할 수 있다는 것을 보여줍니다.


```yaml
---
vars:
    cli:
    host: "{{ inventory_hostname }}"
    username: operator
    password: secret
    transport: cli

tasks:
- nxos_config:
   src: config.j2
   provider: "{{ cli }}"
   username: admin
   password: admin
```
위의 예에서는, cli provider에 operator/secret 라는 사용자 이름/암호가 있지만 아랫부분의 task 접속 사용자와 암호 admin/admin를 대신 이용할 것입니다.

이런 방식은 전송 방법을 기술하는 provider의 모든 인자에 동일하게 적용됩니다. 따라서 CLI 또는 NXAPI를 지원하는 단일 작업을 다음과 같이 가질 수 있습니다.

```yaml
---
vars:
    cli:
    host: "{{ inventory_hostname }}"
    username: operator
    password: secret
    transport: cli

tasks:
  - nxos_config:
      src: config.j2
      provider: "{{ cli }}"
      transport: nxapi
```

하지만, 

```yaml
---
vars:
  conn:
  password: cisco_pass
  transport: cli

tasks:
- nxos_config:
  src: config.j2
  provider: "{{ conn }}"
```
위의 예제에서는 다음과 같은 오류가 발생할 겁니다.

```ini
"msg": "missing required arguments: username,host"
```

## 네트워킹 환경 변수

다음과 같은 환경 변수가 사용됩니다.

- ANSIBLE_NET_USERNAME : 사용자 ID
- ANSIBLE_NET_PASSWORD : 사용자 암호
- ANSIBLE_NET_SSH_KEYFILE : 키 파일
- ANSIBLE_NET_AUTHORIZE : 승인
- ANSIBLE_NET_AUTH_PASS : 승인 암호

변수는 다음과 같은 순서로 적용 됩니다. 가장 아래 부분이 가장 높은 우선 순위를 갖습니다.

1. 디폴트 값
2. 환경 변수
3. Provider
4. Task 인자


## 네트워킹 모듈에서 조건식

Ansible에서 플레이북의 명령이 수행될 때 조건식을 이용할 수 있습니다. 

- eq : 같다
- neq : 같지 않다
- gt : 크다
- ge : 크거나 같다
- lt : 작다
- le : 작거나 같다
- contains : 포함됨

조건문은 원격 장치에서 수행된 명령의 결과를 비교합니다. 일단 작업이 명령 세트가 수행되다 ***waitfor*** 인자를 만나게 되면 이전 수행되던 작업이 플레이북으로 제어를 리턴하기 전 그 결과를 조사할 수 있습니다.

예를 들어,

```yaml
---
- name: wait for interface to be admin enabled
  eos_command:
      commands:
          - show interface Ethernet4 | json
      waitfor:
          - "result[0].interfaces.Ethernet4.interfaceStatus eq connected"
```
위의 예제에서 `show interface Ethernet4 | json` 명령이 원격 장치에서 수행되고 나서 그 결과를 확인하는데, 만약 `(result[0].interfaces.Ethernet4.interfaceStatus)` 가 **connected** 가 될 때까지 이전 명령을 반복합니다. 이 과정은 재시도 횟수 만큼 반복합니다. (디폴트로 1초 간격으로 10번 재시도를 합니다)

command 모듈은 여러번 나올 수도 있습니다.

```yaml
---
- name: wait for interfaces to be admin enabled
  eos_command:
      commands:
          - show interface Ethernet4 | json
          - show interface Ethernet5 | json
      waitfor:
          - "result[0].interfaces.Ethernet4.interfaceStatus eq connected"
          - "result[1].interfaces.Ethernet4.interfaceStatus eq connected"
```

위의 예제에서는 Ethernet4, 5를 확인(show interface) 하는 명령을 수행하고 그 결과를 조사합니다. waitfor 모듈의 결과는 항상 result로 나오며 앞 명령의 순서대로 [0], [1], ... 과 같은 순서로 지정하여 결과를 확인합니다.

​













