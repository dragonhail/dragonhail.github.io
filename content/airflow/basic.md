### Airflow on Kubernetes
- Airflow Executer
  - 필요할 때만 Kubernetes 자원을 사용
  - KubernetesPodOperator를 사용하면 개발자가 필요한 Docker 컨테이너만을 골라 POD로 실행시킬 수 있음