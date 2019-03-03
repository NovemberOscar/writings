마이뮤직테이스트에서 정말 감명깊게 사용했던 도구중 하나는 바로 Hashicorp Vault 였다.
모든 설정과 보안 정보들이 한곳에서 관리되고 Github토큰 하나로 정보를 받아올 수 있는건 정말 센세이셔널한 경험이였다.

이제 Vault가 무엇인지에 관해 알아보고, 설정을 해보도록 하자

# Vault란?

> ***Manage Secrets and Protect Sensitive Data***
> \- Hashicorp Vault catchpraise

Vault는 민감하고 중요한 비밀 정보를 관리 하기 위한 관리 도구이다. 

많은 사람들도 그렇겠지만 나도 이때까지는 데이터베이스 커넥션 등의 보안 정보를 그냥 소스코드에 하드코딩해서 넣거나, cfg 파일 등으로 옮겨다니고, 환경변수에서 가져다 쓰는 경우가 많았다. 당연히 쓰는 정보의 포맷이 달라지거나, 정보에 변경이 발생하면 해당 정보를 사용하는 모든 앱의 환경에 들어가서 정보를 바꿔줘야 했다. 당연히 하드코딩해서 넣은 경우는 소스코드 저장소에 해당 정보가 올라가게 되는데 당연히 전혀 좋을것이 없다.

이렇게 불편하고 불안정한 보안 관리의 어려움을 해결해 주는것이 바로 hashicorp Vault이다.

Vault의 개념은 간단하다. 

모든 정보를 볼트에서 관리하고, 클라이언트(개발자 또는 앱)은 볼트 접근 권한을 받은 뒤 볼트에서 주는 정보를 받아 사용하면 된다. 
이러한 콘셉트를 기반으로 단순히 안전한 KV 스토어 이외에도 데이터베이스나 SSH 정보를 TTL을 지정해서 동적으로 생성해 주는 등 다양한 기능을 제공한다. 

# Vault를 설치해봅시다

```bash
$ wget https://releases.hashicorp.com/vault/1.0.3/vault_1.0.3_linux_amd64.zip(또는 최신 다운로드 주소)
$ mkdir vault_server
$ unzip <다운로드 받은 바이너리> -d vault_server
```
볼트를 해시코프 서버에서 다운로드 받고 압축을 풀면 볼트를 사용할 준비가 된다.

볼트를 서버로 작동시키기 위해서는 정보를 저장할 스토리지가 필요하다. 고가용성을 살리고 싶다면 DynamoDB나 Hashicorp Consul을 사용해야 하지만 여기서는 고가용성이 필요하지 않기 때문에 간단히 파일시스템을 사용하도록 한다

이제 vault 의 환경 설정을 저정할 HCL 파일을 만들어보자
```bash
$ cd vault_server
$ touch config.hcl
$ nano config.hcl
```

config.hcl 파일에 다음의 내용을 저장하자.

```hcl
storage "file" {
  path = "/mnt/vault/data"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = false
}

ui = true
```
TLS는 볼트 또한 대부분의 WAS 같이 nginx같은 웹서버를 한번 거치는 경우가 많아서 비활성화 하도록 했다.  TLS 설정은 웹서버 단에서 하도록 하자.
UI를 비활성화 하기 위해서는 ui 옵션을 false로 주면 된다. 

이제 vault 서버를 시작시켜 보자.
```bash
$ sudo ./vault server -config=config.hcl
```
root 권한으로 실행하지 않으면 메모리 보안 관련 에러가 나기 때문에 sudo로 실행해 준다.

서버 설치 과정은 이게 끝이다. 이제 클라이언트에서 Vault를 초기화 하고 언씰해야 볼트를 사용할수 있게 된다.

클라이언트 환경에서 볼트의 주소를 환경변수에 등록하자
```bash
$ echo 'export VAULT_ADDR=<주소>' >> ~/.bash_profile
```

볼트를 초기화하기 위해 vault init 명령어를 수행한다.
```bash
$ ./vault operator init
Unseal Key 1: ____________________________________________
Unseal Key 2: ____________________________________________
Unseal Key 3: ____________________________________________
Unseal Key 4: ____________________________________________
Unseal Key 5: ____________________________________________

Initial Root Token: ______________________________

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```
볼트를 초기화 하면 볼트를 언씰하기 위한 5개의 언씰 키와 루트 토큰을 준다.
언씰키가 없으면 볼트가 씰링됬을때 언씰을 할 수가 없고 저장된 정보의 복호화 또한 담당하기 때문에 조심히 관리해야 한다. 루트 토큰은 말 그대로의 권한을 가지고 있기 때문에 조심히 보관하도록 하자.

볼트가 초기화 되었다면 볼트가 씰링된 상태이기 떄문에 볼트를 사용하려면 5개 중 3개의 언씰 키를 사용하여 볼트를 언씰할수 있다.

미사일 발사처럼 볼트를 열기 위해서는 여러개의 키가 필요하다고 생각하면 된다

```bash
$ ./vault operator unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1/3
```
기본적으로 쓰레시홀드가 3번이기 떄문에 언씰 명령어를 3개의 다른 키를 가지고 수행하면 볼트가 언씰될 것이다.

# Vault Github 토큰으로 접근하기
Vault는 자체적으로 발급하는 토큰을 사용해 접근할 수도 있지만 깃허브 토큰이나 LDAP 등 다양한 인증 방법을 제공한다.

볼트에 root 권한으로 접근하기 위해 클라이언트 환경에 환경변수로 토큰을 저장하자.
```bash
$ echo "export VAULT_TOKEN=<root토큰>" >> ~/.bash_profile
```
github auth를 활성화 한 후 접근 권한을 가진 조직을 설정하자
```bash
$ ./vault auth enable github
$ ./vault write auth/github/config organization=<github organization 이름>
```
이제 Github 토큰을 통해 볼트에 접근할 수 있다.

Github 토큰은 https://github.com/settings/tokens 에서 read:org 권한을 가진 토큰을 만들어서 사용하면 된다.

만약 ui 옵션을 활성화 했다면 볼트에 웹 브라우저로 접속해 확인해 보자
![image.png](https://images.velog.io/post-images/novemberoscar/450eaf20-3cdd-11e9-ad7e-19e7a9c6b541/image.png)

이제 볼트를 사용할 준비가 완료되었다!









