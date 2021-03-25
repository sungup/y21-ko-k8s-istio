# Istio Installation

3월 3일 한국 쿠버네티스 유저그룹의 스터디 내용 정리입니다.

## Istioctl 설치

아래 명령어로 istioctl 을 설치

```bash
$ crul -sL https://istio.io/downloadIstioctl | sh -
$ export PATH=$PATH:$HOME/.istioctl/bin
```

## Istio Operator 배포

설치 후 K8s에서 활용하기 위해 Istio-Operator deploy와 IstioOperator CRD가 배포됨

```bash
$ istioctl operator init
Operator controller is already installed in istio-operator namespace.
Upgrading operator controller in namespace: istio-operator using image: docker.io/istio/operator:1.9.1
Operator controller will watch namespaces: istio-system
✔ Istio operator installed
✔ Installation complete
```

Istio의 CRD 확인

```bash
$ kubectl get crd  | grep istiooperator
istiooperators.install.istio.io                       2021-03-07T02:03:51Z
$ kubectl get all -n istio-operator
NAME                                  READY   STATUS    RESTARTS   AGE
pod/istio-operator-7b486b549f-jb5qv   1/1     Running   0          18d

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/istio-operator   ClusterIP   10.103.33.181   <none>        8383/TCP   18d

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-operator   1/1     1            1           18d

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-operator-7b486b549f   1         1         1       18d
```

## Istio System 네임스페이스 생성 및 Istio Control Plane 배포

Istio-System Namespace 생성. 이때 spec 상에서 선택할 수 있는 3종류의 기본 Profile이 있으며, 여기서 지원하는 Core Component의 조합은 아래와 같음

|                      | default | demo | minimal | preview | remote / empty |
|----------------------|:-------:|:----:|:-------:|:-------:|:--------------:|
| istio-egressgateway  |         |   O  |         |         |                |
| istio-ingressgateway |    O    |   O  |         |    O    |                |
| istiod               |    O    |   O  |    O    |    O    |                |

아래의 YAML 파일은 기본 Profile로 demo를 설정하여 egressgateway를 사용하도록 하였으며, 개별 component에 대한 세부적인 설정을 추가로 지정하였음.

- **pilot의 메모리 용량 (spec.components.k8s.resources.request.memory)**: 2GB
- **egressgateway 활성화 (spec.egressGateways.name & enabled)**: istio-egressgateway & true

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: example-istiocontrolplane
spec:
  profile: demo
  components:
    pilot:
      k8s:
        resources:
          requests:
            memory: 2048Mi
    egressGateways:
    - name: istio-egressgateway
      enabled: true
```

아래 kubectl 명령어를 통해 해당 내용을 적용. 위 설정 파일 내용을 적용한 파일명은 `02.istio-system-namespace.yaml`로 지정함. 적용한 후 해당 서비스와 pod 배포 여부를 확인함.

```bash
$ kubectl apply -f 02.istio-system-namespace.yaml 
istiooperator.install.istio.io/example-istiocontrolplane configured
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
istio-egressgateway    ClusterIP      10.109.135.88   <none>        80/TCP,443/TCP,15443/TCP                                                     18d
istio-ingressgateway   LoadBalancer   10.108.170.15   <pending>     15021:30912/TCP,80:32472/TCP,443:30405/TCP,31400:31671/TCP,15443:31771/TCP   18d
istiod                 ClusterIP      10.102.148.62   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        18d
$ kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-bd477794-x8qd5      1/1     Running   0          18d
istio-ingressgateway-79df7c789f-k44rr   1/1     Running   0          18d
istiod-864cc48f48-h9vjw                 1/1     Running   0          2m35s
```

## Istio-injection 라벨을 Namespace에 추가

Pod 생성시 envoy를 sidecar 패턴으로 **삽입**해야 하기 때문에 istio의 sidecar injection 기능을 활성화 시켜야 함. 이를 위해서는 injection 을 위한 해당 Namespace의 라벨에 `istio-injection=enabled` 라벨이 추가되어야 함.

```bash
$ kubectl label ns default istio-injection=enabled 
namespace/default labeled
$ kubectl get ns --show-labels
NAME              STATUS   AGE   LABELS
default           Active   18d   istio-injection=enabled
istio-operator    Active   18d   istio-injection=disabled,istio-operator-managed=Reconcile,istio.io/rev=default,operator.istio.io/component=IstioOperator,operator.istio.io/managed=Reconcile,operator.istio.io/version=1.9.1
istio-system      Active   18d   <none>
kube-node-lease   Active   18d   <none>
kube-public       Active   18d   <none>
kube-system       Active   18d   <none>
```
