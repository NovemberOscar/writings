# UI?
0.10버전 정도 까지는 볼트에 기본적으로 웹 ui가 제공되지 않았다고 한다.

그래서 react를 기반으로 하는 [비공식 vault ui](https://github.com/djenriquez/vault-ui)를 이용해 웹 에서 볼트를 사용했고 공식 ui가 나온 지금도 익숙함 때문에 비공식 ui를 사용하고 있다 

특히나 비공식 ui가 JSON포맷팅 기능등을 잘 제공하고 볼트 서버에서 작동되는게 아니라 볼트는 따로 띄워 놓고 외부에서는 ui만 접근하게 할 수도 있기 때문에 일부러 사용하기도 한다.

간단히 도커 이미지로 배포되기 때문에  다음의 명령어를 사용해 도커 이미지를 띄울 수 있다.

```bash
docker run -d \
-p 8000:8000 \
-e VAULT_URL_DEFAULT=http://vault.server.org:8200 \
-e VAULT_AUTH_DEFAULT=GITHUB \
--name vault-ui \
djenriquez/vault-ui
```

도커 이미지가 실행되었다면 도커를 띄운 호스트에 8000 포트로 접근해보자. 다음과 같은 화면이 반겨줄 것이다.


![image.png](https://images.velog.io/post-images/novemberoscar/0beec660-3d70-11e9-810f-0defcc823505/image.png)

옆에 설정 아이콘을 누르면 로그인 방법등을 설정할 수 있다. 이전 포스트에서 나온 대로 깃허브 토큰을 사용하고 있다면 깃허브 토큰을 사용하여 로그인 해 보자.

로그인하면 다음과 같은 화면을 볼 수 있을것이다.
(secret backend의 service-secret은 내가 추가한 것이다)
![image.png](https://images.velog.io/post-images/novemberoscar/67d14430-3d70-11e9-810f-0defcc823505/image.png)

이제 Vault에 편리한 UI를 붙여서 사용할 수 있다!

다음 포스트에서는 기본적인 정책 설정과 KV 스토어 사용 방법에 대해 알아볼 것이다
