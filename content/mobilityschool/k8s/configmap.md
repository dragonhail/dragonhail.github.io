---
title: ConfigMap과 Secret
weight: 4
---
- Kubenetes 객체가 사용하기 위한 데이터를 저장하기 위한 객체
- 환경변수 와 같은 데이터는 대부분 실행 중에는 변하지 않고 실행하기 직전에 변할 수 있는 문자열이나 외부로 노출되어서는 안되는 데이터
- ConfigMap 이나 Secrets 은 스스로 어떤 기능을 하지는 않고 적은 양의 데이터를 저장하는 것이 목적

### ConfigMap
- 문자열 상수를 이용해서 만들 수 있고 파일의 내용을 읽어서 만들 수 있음
- 생성형식
  - `kubectl create configmap <map-name> <data-source> <arguments>`
- 문자열 상수(리터럴)를 가지고 생성
  - `kubectl create configmap [map name] --from-literal=[키]=[값]`
```bash
kubectl create configmap my-config --from-literal=JAVA_HOME=/usr/java

kubectl delete configmap my-config

kubectl create configmap my-config --from-literal=JAVA_HOME=/usr/java --from-literal=URL=http://localhost:8000

kubectl get configmap my-config

kubectl describe configmap my-config
```
- 파일에서 읽어서 만들기
  - 데이터를 저장하는 yaml 파일 생성하고 작성
  ```bash
  echo Hello Config >> configmap_test.html

  kubectl create configmap configmap-file --from-file configmap_test.html
  ```