### istio 공부를 시작하게 된 이유

카카오 클라우드 스쿨이 끝나고 휴식을 취하던 중 교육 과정에서 만난 분들과 팀을 이루게 되었습니다. 프로젝트로 Service mesh를 하게 되었고 이를 위해 istio를 공부하게 되었습니다. 아래의 내용은 istio를 공부하면서 정리 한 내용입니다.

---

## 왜 istio인가?

### Monolithic Application
과거의 software개발은 process지향적이였으며 느렸습니다. 이를 해결하기 위해 Module의 개념을 도입함으로써 App의 기능 별로 나누어 deploy하는 Monolithic구조가 탄생했습니다. Module별로 나누어 코드를 작성하다 보니 유지 관리도 이전 보다 좋아졌습니다. 하지만 모든 기능들이 동일한 code를 기반으로 작성되었고 기능들 사이엔 명확한 경계가 존재하지 않았으며 Module끼리 영향을 미치다 보니 Module하나가 망가지면 전체적인 App까지도 영향을 미치게 되었습니다.

### Microservice Application
Monolithic의 가장 큰 단점인 Module사이의 경계가 없다는 점을 보완했습니다. 각 Module들이 독립적으로 분리되었으며, 이젠 더이상 동일한 code로 작성을 안합니다. Microservice Application의 장점으로는 확장성이 좋고 빠르며, release가 적습니다. 또한 복원력과 독립적입니다.

### Service Mesh
Service Mesh란 code를 변경하지 않고도 service간 통신을 처리하는 구성 가능한 전용 인프라 구조 계층입니다. 각각의 microservice에 proxy들이 data plane을 통해 서로 communication을 하며 server site의 구성 요소인 control plane과 communication을 합니다.

Service mesh의 역할로는 아래와 같습니다.
  
- **Traffic Management** - service가 서로 통신하는 방식을 동적으로 구성
- **Security**                       - service가 통신할 때 mutual TLS를 사용
- **Observability**              - app의 종단 간 수행 방식이나 병목 현상을 관측
- **Service Discovery**      - Discovery, Health Check, Load Balancing

![image](https://github.com/ojwsakin/istio_study/assets/93701762/38aadde2-e0d0-463f-bf19-00a92d32c86a)

---

## ISTIO
Istio는 Open-source Service Mesh로 서비스를 보호, 연결 및 모니터링하는 효율적인 방법을 제공합니다. K8S및 기존 workload와 함께 동작하며 traffic을 관리하고 보안 등 복잡한 Microservice deploy에 적용합니다. Istio는 open-source proxy인 Envoy proxy를 사용합니다.

### Envoy Proxy
사용자는 proxy를 통해서 application에 요청을 합니다. Envoy proxy는 분산 architecture와 microservice를 위해 디자인 되었습니다.
![image](https://github.com/ojwsakin/istio_study/assets/93701762/1b948f18-297a-4861-905a-517ffdbc0856)



### Istio Control plane
Control plane에 위치하는 istiod는 관리형 인증서를 생성하고 serivce를 discovery하며 configuration file을 검증합니다.
istiod의 구성 요소로는 3가지가 있습니다. 원래는 istiod로 묶여있지 않고 개별적으로 존재했으나 현재는 istiod로 묶여있습니다.

istiod의 3가지 구성 요소

- **Citadel** : 관리형 인증서 생성
- **Pilot** : Service discovery
- **Galley** : Configuration files 검증

![image](https://github.com/ojwsakin/istio_study/assets/93701762/51d68909-93fa-43a9-8c20-fe7e51945c1c)

### Istio Gateway
istio gateway는 inbound, outbound의 트래픽을 관리합니다. istio-gateway는 Envoy proxy를 이용해 gateway를 배포합니다.

### Virtual Service
Virtual Service란 gateway에서 service mesh로 들어오는 traffic에 대한 routing rule을 정의합니다. traffic을 다양한 service로 분산하거나, version upgrade, A/B test등을 수행합니다.

### Destination Rules
traffic이 특정 service로 routing된 이후의 routing 정책입니다.

### Fault injection
특정 상황에서 network나 service에 의도적으로 장애를 주입함으로써 service mesh환경에서 장애 처리 및 견고성을 검증하고 테스트 할 수있습니다.

### Circuit Breaking
service간의 통신에서 과부하나 장애 상황에 대비해서 traffic을 조절하고 격리시킵니다.

## Istio Security
Istio는 mTLS와 세분화된 access정책을 사용해서 보안을 사용하며 누가 접근을 했는지 audit logs를 제공합니다.

### Istio Security Architecture
- Envoy proxies는 istio 에이전트에서 인증서와 키를 요청하고 서비스간의 보안 통신을 지원
- API 서버 구성 요소는 모든 인증, 권한 부여 및 보안 이름 지정 정책을 proxy에 배포
- sidecar와 ingress/egress proxy는 정책 시행 지점으로 작동

### mTLS 작동 원리
- **디지털 인증서 발급**: 각 엔티티(클라이언트 및 서버)는 디지털 인증서를 생성하고, 해당 인증서에는 고유한 공개키와 개인키가 포함됩니다.
- **서버의 인증서 전송**: 서버는 클라이언트에게 자신의 디지털 인증서를 제공합니다.
- **클라이언트의 인증서 전송**: 클라이언트는 서버에게 자신의 디지털 인증서를 제공합니다.
- **클라이언트 및 서버 간의 인증 및 키 교환**: 클라이언트는 서버의 인증서를 사용하여 서버의 공개키를 가져오고, 서버는 클라이언트의 인증서를 사용하여 클라이언트의 공개키를 가져옵니다. 이후 양방향으로 암호화된 통신이 시작됩니다.
- **통신의 보안 및 암호화**: 통신은 클라이언트 및 서버 간의 암호화된 연결을 통해 이루어지며, 중간자 공격 등을 방지하기 위해 신뢰할 수 있는 통신 경로를 확립합니다.

### Certificate Management
여기서는 간단하게 인증서가 어떻게 생성 되는지 확인합니다.
![image](https://github.com/ojwsakin/istio_study/assets/93701762/c53e6b67-10ec-41da-8548-9fe31d343294)

1. istio-agent가 private key와 CSR(Certificate Signing Request)를 create
2. 자격 증명과 CSR을 위해 istiod로 send
3. istiod의 CA는 CSR에 포함된 자격 증명의 유효성 검사를 진행
4. 유효성 검사를 성공적으로 마치게 되면 CSR에 서명을하여 cert(인증서)를 생성
5. 서명이 된 certificate와 private key를 Envoy proxy에 전달한다
6. isito-agent는 workload 인증서의 만료를 모니터링하며 인증서 및 key 순환에 주기적으로 반복

---

## Istio 실습 해보기

### 실습 환경
실습을 진행한 환경은 macOS로 docker Desktop을 사용했습니다. 실습 진행을 위해 istio-study라는 디렉토리를 생성합니다.

### istio 설치 하기

```bash
## istio install
# sh는 script를 실행하는 명령어 이며 - 이므로 스크립트에 넘겨줄 인자는 없다
curl -L https://istio.io/downloadIstio | sh -

## istio 디랙토리로 이동하게 되면 bin 아래에는 다양한 example들이 존재

## istioctl을 사용하기 위한 설정
# istio 디랙토리 위치에서 진행
export PATH=$PWD/bin:$PATH

istioctl version
```
<img width="888" alt="image" src="https://github.com/ojwsakin/istio_study/assets/93701762/d9de8364-e6f7-4eaa-a4bc-247a1a42a751">

```bash
## install & profile 지정
# profile에는 여러가지가 있으며 환경에 따라 선택 (실습이므로 demo profile 사용)
# istiod는 istio-system이라는 namespace에 배포된다
# istiod와 istio-ingreessgateway와 istio-egressgateway도 배포된다
istioctl install --set profile=demo -y
```
<img width="956" alt="image" src="https://github.com/ojwsakin/istio_study/assets/93701762/656c9b07-c2e0-4f28-960e-b128df548c5f">


```bash
## install을 확인 (verify the installation)
# CustomResourceDefintions(CRD)를 확인할 수 있다
istioctl verify-install

## analyze
# istio injection을 사용할 수 있는 상태인지 확인
istioctl analyze
```
### istio-injection
instio-injection이란 지정한 namespace에 deploy하는 container에 자동적으로 proxy를 side-car로 배포할 수 있도록 설정하는 것입니다. 설정은 아래와 같이 namespace별로 할 수 있고 마찬가지로 disable도 할 수 있습니다. 확인을 위해 injection-test라는 ns를 생성하여 확인하겠습니다.

```bash
## sidecar injection
kubectl label namespace {{ns_name}} istio-injection=enabled

## disabled injection
kubectl label namespace {{ns_name}} istio-injection=disabled

## injection 조회 command
kubectl get namespace -L istio-injection
```
<img width="1084" alt="image" src="https://github.com/ojwsakin/istio_study/assets/93701762/ad6aa9f9-fc88-4c1c-b655-99f6cf511e71">

image를 nginx로 하는 test pod를 deploy해보면 1개가 되어야 하지만 2개인 것을 확인할 수 있으며 describe를 통해 확인해보면 자동적으로 istio-proxy까지 배포 된 것을 확인 할 수 있습니다.

<img width="901" alt="image" src="https://github.com/ojwsakin/istio_study/assets/93701762/7fbef0f8-0e1e-41ae-a11a-3ad76ca1d882">

<img width="882" alt="image" src="https://github.com/ojwsakin/istio_study/assets/93701762/e4a4ff23-0451-4599-934e-ea9594741ee2">

### Certificate Management 실습
Certificate Management실습 입니다. istio에서 제공하는 selfsigned를 사용했습니다.

```yaml
# ca-cert directory 생성
mkdir ca-certs

# 이동
cd ca-certs

**## 생성한 ca-certs directory에 root-ca 생성**
# istio의 Makefile.selfsigned.mk를 이용해서 자체 서명된 root 인증기관 인증서를 생성
make -f ../tools/certs/Makefile.selfsigned.mk root-ca
>>> root-ca.conf root-cert.csr root-cert.pem root-key.pem

## root-ca.conf
# root 인증서를 생성하기 위한 open SSL

## root-cert.csr
# root 인증서에 대해 생성된 CSR

## root-cert.pem
# 생성된 root 인증서

## root-key.pem
# 생성된 root key

## 우리 서비스를 위한 인증서 생성
make -f ../tools/certs/Makefile.selfsigned.mk localcluster-cacerts

## 만약 인증서를 생성하기 전에 istio-system에 서비스가 있었다면 삭제
kubectl create namespace istio-system

## cd localcluster

## secret 생성
kubectl create secret generic cacerts -n istio-system \
--from-file=ca-cert.pem \
--from-file=ca-key.pem \
--from-file=root-cert.pem \
--from-file=cert-chain.pem

## istio install
istioctl install --set profile=demo

kubectl apply -f samples/addons
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml

istioctl analyze

## mTLS traffic만 허용
kubectl apply -f - <<EOF
apiVersion:security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "default"
spec:
  mtls:
    mode: STRICT
EOF

## product page에 연결하려고 할 때 open SSL을 사용하여 app의 인증서 chain을 확인
kubectl exec "$(kubectl get pod -l app=details -o jsonpath={.items.metadata.name})" -c istio-proxy -- openssl s_client -showcerts -connect productpage:9080 > httpbin-proxy-cert.txt
>>> depth =2 0 =Istio, Root CA
>>> verify error:num19:self signed certificate in certificate chain
>>> Done

sed -ne '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' httpbin-proxy-cert.txt | sed 's/^\s*//' > certs.pem

# 위의 certs.pem을 4개로 나눈다
split -p "-----BEGIN CERTIFICATE-----" cedrts.pem proxy-cert-

# root 인증서의 인증서 정보를 덤프
openssl x509 -in ca-certs/localcluster/root-cert.pem -text -noout > /tmp/root-cert.crt.txt

# 트래픽 자체에서 추출한 정보
openssl x509 -in ./proxy-cert-3.pem -text -noout > /tmp/pod-root-cert.crt.txt

# 위 두 정보가 같은지 확인
diff -s /tmp/root-cert-crt.txt /tmp/pod-root-cert.crt.txt
>>> are identical

# CA 인증서가 우리가 지정한 인증서와 동일한지 확인
openssl x509 -in ca-certs/localcluster/ca-cert.pem -text -noout > /tmp/ca-cert.crt.txt
opensll x509 -in ./proxy-cert-2.pem -text -noout > /tmp/pod-cert-chain-ca.crt.txt

# root 인증서에서 워크로드 인증서까지의 인증서 chain 확인
openssl verify -CAfile <(cat ca-certs/localcluster/ca-cert.pem ca-certs.localcluster/root-cert.pem) ./proxy-cert-1.pem
>>> ./proxy-cert-1.pem OK

istioctl dashboard kiali
istioctl dashboard promethaus
```



