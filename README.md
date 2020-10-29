# Reserve

## Check Points

### CQRS
![cqrs_관련 정보 조회]!(https://user-images.githubusercontent.com/28075892/97527578-ffd51e80-19ee-11eb-87d4-e62b5b075676.png)

### Polyglot
* pom.xml 설정<br>
![polyglot_](https://user-images.githubusercontent.com/73535272/97516628-df996580-19d6-11eb-8154-5bd7d55ec697.png)

### Correlation

### Req / Resp
1. Req-Res 호출<br>
![req-res_1_호출 소스](https://user-images.githubusercontent.com/73535272/97381900-64b84800-190d-11eb-991e-c18770bd9f04.JPG)

2. Req-Res 처리<br>
![req-res_2_처리 controller](https://user-images.githubusercontent.com/73535272/97381905-6a159280-190d-11eb-88b8-c2212245e8c4.JPG)

### Gateway
1. Gateway 설정<br>
![gateway-설정](https://user-images.githubusercontent.com/12227092/97509615-20d54980-19c6-11eb-85fe-59a6a256368c.JPG)

2. Gateway rental  설정<br>
![Gateway_rental생성](https://user-images.githubusercontent.com/73535272/97379165-7f87be00-1907-11eb-9f6f-9f4ab3f7cb7e.JPG)

3. Gateway 정보조회<br>
![Gateway_정보조회](https://user-images.githubusercontent.com/73535272/97379181-8adae980-1907-11eb-80b2-1595781751d0.JPG)

4. Gateway 직접정보조회<br>
![Gateway_직접정보조회](https://user-images.githubusercontent.com/73535272/97379198-975f4200-1907-11eb-96f2-94b3a6e50938.JPG)

### Deploy/pipeline

### Curcuit Breaker
* reserve 서비스로 과다한 이벤트가 발생 시 해당 서비스와 연결된 다른 서비스들에 영향을 주지 않도록 curcuit breaker 가 동작하게 구현 할 예정.
### HPA
1. HPA 설정
#kubectl autoscale deploy rent --min=1 --max=10 --cpu-percent=15<br>
![HPA_1_실행전 상태](https://user-images.githubusercontent.com/12227092/97423776-1d54aa80-1953-11eb-82b8-03db56590378.JPG)

#kubectl get hpa rent -o yaml  로 설정 확인<br>
![HPA_0_yaml 설정](https://user-images.githubusercontent.com/12227092/97424249-cc918180-1953-11eb-9e4a-9a66c1968f3d.JPG)

2. HPA 실행<br>
시즈 사용한 부하 발생<br>
![HPA_2_siege 실행](https://user-images.githubusercontent.com/12227092/97423872-4412e100-1953-11eb-98b3-948b1f27bb84.JPG)

![HPA_3_실행 후 상태](https://user-images.githubusercontent.com/12227092/97423925-5856de00-1953-11eb-931c-e3fbf6bb5b5e.JPG)

rent의 POD가 10개까지 확장되어 생성됨을 확인

### Liveness
1. deploy.yaml 설정<br>
![Liveness_ deploy yaml](https://user-images.githubusercontent.com/12227092/97439664-ca3a2200-1969-11eb-9260-03676374e560.JPG)

2. Liveness가 스케쥴 돌면서 재기동함을 확인<br>
![Liveness_Restarts 확인](https://user-images.githubusercontent.com/12227092/97439856-0f5e5400-196a-11eb-848f-8f9b98575bd7.JPG)

3. rental 서비스가 동작중임을 확인<br>
![Liveness_ 동작중](https://user-images.githubusercontent.com/12227092/97439765-f5247600-1969-11eb-8633-22103708e3dd.JPG)

3. rent deploy에 대한 describe 확인<br>
![Liveness_ describe로 확인](https://user-images.githubusercontent.com/12227092/97439959-2dc44f80-196a-11eb-8d6b-799142c4521d.JPG)

### Readiness 
1. 서비스에 seige로 부하를 주고, kubectl apply -f non-readiness.yaml 로 이미지 배포 (po, deploy에 설정된 autoscale 제거 후 실행함)<br>
#siege -c255 -t120S -r10 --content-type "application/json" 'http://rent:8080/rental POST {"bookId":"5", "qty":1}'<br>
![readiness_1](https://user-images.githubusercontent.com/73535272/97518369-64d24980-19da-11eb-9115-cfe615cd946d.jpg)

약 성공률이 21%로 확인되어, 신규 이미지 배포 중 request가 넘어가지 못하는 것을 확인

2. 서비스에 seige로 부하를 주고, kubectl apply -f readiness.yaml 로 이미지 배포 (po, deploy에 설정된 autoscale 제거 후 실행함)<br>
#siege -c255 -t120S -r10 --content-type "application/json" 'http://rent:8080/rental POST {"bookId":"5", "qty":1}'
![readiness_2](https://user-images.githubusercontent.com/73535272/97518371-66037680-19da-11eb-9090-446a9996a27c.jpg)<br>
성공률이 100%로 확인되어, 신규 이미지 배포 중 Request가 모두 처리되는 것을 확인


3. 서비스에 seige로 부하를 주어 Availablity를 확인
#siege -c255 -t120S -r10 --content-type "application/json" 'http://rent:8080/rental POST {"bookId":"5", "qty":1}'<br>
![Readiness_설정 후 seige결과](https://user-images.githubusercontent.com/12227092/97463074-b8b14400-1982-11eb-9682-edea8a0f895b.JPG)

### Config Map/ Persistence Volume
1. Config Map 설정<br>
configMap.yaml으로 설정<br>
![configMap_yaml](https://user-images.githubusercontent.com/12227092/97428391-44ae7600-1959-11eb-8f76-9b03920a75e6.JPG)

#kubectl create -f configMap.yaml  -> configMap 생성<br>
#kubectl describe configmap superuser -> 추가한 configMap 정보 조회<br>
![configMap 설정](https://user-images.githubusercontent.com/12227092/97428626-9820c400-1959-11eb-806b-809b01044a84.JPG)

2. config Map 적용<br>
deploy.yaml에 해당 config Map 설정<br>
![ConfigMap_ deploy yaml](https://user-images.githubusercontent.com/12227092/97428733-c4d4db80-1959-11eb-942d-3848a7eb1fee.JPG)

application.yaml에 아래의 config Map 설정<br>
![ConfigMap_ application yaml](https://user-images.githubusercontent.com/12227092/97428874-f352b680-1959-11eb-8147-61d5bd1be8ee.JPG)

java단에 해당 config Map 적용<br>
![ConfigMap RentalController](https://user-images.githubusercontent.com/12227092/97437338-a75a3e80-1966-11eb-9101-1feb4ef4958d.JPG)

운영환경에서 실행확인<br>
![ConfigMap 실행 결과](https://user-images.githubusercontent.com/12227092/97437394-be009580-1966-11eb-9f8d-e5a07134a02e.JPG)


