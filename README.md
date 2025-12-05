
# 🎮 MLOps 기반 게임 추천 시스템  
**MLOps Game Recommendation Project — 개인 연구/실험 기반 작품**

팀 프로젝트: https://github.com/AIBootcamp16/mlops-cloud-project-mlops-3

## 💻 프로젝트 개요  
이 프로젝트는 **게임 추천 모델을 설계하고**, 이를 실제 운영 가능한 **MLOps 파이프라인**으로 구성하는 것을 목표로 했다.  
단순한 분석 수준의 추천 모델이 아니라:

- Docker로 환경 일관성 확보  
- Airflow로 파이프라인 자동화  
- Slack으로 모니터링 및 알림  
- GitHub Actions로 CI/CD 구축  

까지 적용한 **End-to-End ML 시스템 구축 경험**이 담겨 있다.

본 저장소는 원래 팀 프로젝트의 dev 버전에서 출발했지만, 이후 **개인적으로 실험하고 확장한 코드들**을 정리해 하나의 작품으로 재구성한 것이다.

---

## 🎯 목표  
- **사용자 게임 플레이 이력 기반 추천 모델(Item-CF) 구현**  
- **RAWG API 기반 게임 데이터 자동 수집/검증/전처리**  
- **Airflow DAG로 학습 → 검증 → 추론 → 알림 전체 자동화**  
- **Docker + GitHub Actions로 CI/CD 구성**  
- **W&B로 실험 버전 관리 및 성능 추적**

---

## 🔧 기술적 특징  

### **1) 데이터 수집 & 전처리**
- RAWG API를 통해 게임 메타데이터 자동 수집  
- Label Encoding, StandardScaler, 장르 임베딩 등을 활용한 특징 생성  
- Airflow 상에서 데이터 검증(Validation Task) 자동화  

### **2) 모델링**  
- Item-based Collaborative Filtering (아이템 기반 협업 필터링)  
- 사용자 게임 플레이 이력을 기반으로 유사도 추천  
- W&B로 실험 파라미터 기록 및 모델 버전 관리  

### **3) 환경 일관성 (Docker)**  
- Dockerfile 기반으로 학습/추론 환경을 통일  
- 어디서든 동일한 환경에서 모델 실행 가능  

### **4) 파이프라인 자동화 (Airflow)**  
- DAG 기반 데이터 파이프라인 구성  
- 데이터 수집 → 전처리 → 학습 → 추론 → Slack 알림 자동 실행  
- DockerOperator 사용  

### **5) CI/CD (GitHub Actions)**  
- main 또는 modeling 브랜치 push 시 자동 빌드  
- Docker Hub에 자동 배포  
- Airflow에서 최신 이미지 자동 사용 가능  

---

## 🕹 사용자 기능  
- `user_id`(1~100)를 입력하면 추천 게임 Top-N 출력  
- DAG 실행 후 결과 자동 생성  
- Slack 알림으로 결과 확인 가능  
- Docker run 하나로 동일한 환경에서 재현 가능  

---

## 📊 산출물  

| 항목 | 설명 |
|------|------|
| 추천 모델 결과 | 사용자별 유사 게임 Top-N 출력 |
| Airflow DAG | 수집 → 검증 → 학습 → 추론 → 알림 전체 파이프라인 |
| 로그 & 모니터링 | Slack 알림, Docker 로그, Airflow Web UI |
| CI/CD | GitHub Actions로 Docker 이미지 자동 빌드/배포 |

---

## 📁 프로젝트 구조

```
.
├── Airflow/
│   ├── dags/
│   └── yaml/
│
├── opt/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── data-prepare/
│   └── mlops/
│
├── mlops-research/
│   └── src/
│
├── tests/
├── .github/workflows/
│   └── ci.yml
└── README.md
```

---

## ⚙️ CI/CD 자동화 (GitHub Actions)
프로젝트 초기에는 dev 브랜치 기반으로 Docker 이미지를 자동 빌드하도록 구성했지만,  
개인 연구 버전으로 구조가 재정비되면서 브랜치 체계가 변경되어 Actions가 일시적으로 중단되었다.  
현재는 **main / modeling** 중심으로 트리거되도록 다시 정비하였다.

### 🔄 동작 방식
- main 또는 modeling push 시 자동 트리거  
- Docker 이미지 빌드 후 Docker Hub에 push  
- Airflow 환경에서 최신 이미지 자동 반영  

### 🧩 Workflow 예시
```yaml
on:
  push:
    branches:
      - main
      - modeling

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: moongs95/mlops-game-recommender:latest
```

---

## 🚨 트러블슈팅 요약  


🔹 Airflow Web UI(8080) 접속 불가

-문제: localhost:8080 접속 시 UI가 끊기거나 아예 접속 불가.

-원인: Docker Desktop 메모리 부족(3.8GB), 네트워크 브리징 충돌.

-해결: Docker 메모리 4GB 이상 할당, docker-compose 재기동.

-결과: 안정적으로 Web UI 접속 가능.



🔹 Docker Host 연결 오류

-문제: DockerOperator 실행 시 “Docker Daemon not found” 에러.

-원인: docker_url을 unix://var/run/docker.sock와 tcp://host.docker.internal:2375 혼용.

-해결: Windows 환경 맞게 tcp://host.docker.internal:2375로 통일.

-결과: DAG 내 DockerOperator 정상 실행.



🔹 DAG 알림 태스크 Skipped 혼동

-문제: DAG 성공 후에도 실패 알림 태스크가 skipped로 표시돼서 실패로 오인.

-원인: Trigger Rule(all_success, one_failed) 동작 이해 부족.

-해결: 정상 동작임을 확인하고 팀 내 공유.

-결과: 불필요한 코드 수정 방지.



🔹 RAWG API ↔ main.py CSV 의존성 불일치

-문제: Airflow는 API에서 데이터를 수집했지만, Docker 컨테이너 내부 학습 코드는 CSV(Top-40 Video Games.csv) 기반.

-해결 방향 (v6 반영 예정):DAG에서 API 응답을 CSV로 저장. 컨테이너는 해당 CSV를 읽어 학습/추론 수행.

-결과(예정): 데이터 파이프라인 일관성 확보.



🔹 Airflow 로그 출력 불가

-문제: 태스크 실행 후 Web UI에서 로그가 비어있거나 “symlink error” 경고.

-원인: Airflow 로그 디렉토리가 Windows 환경에서 심볼릭 링크를 제대로 생성하지 못함.

-해결:로그 디렉토리(./logs)를 미리 생성하고 권한 수정. AIRFLOW__CORE__BASE_LOG_FOLDER와 AIRFLOW__CORE__DAGS_FOLDER 환경 변수 경로를 절대경로로 지정.

-결과: 로그 정상 출력, 디버깅 가능해짐.

---

## 📌 프로젝트 회고  
과거 머신러닝 프로젝트에서는 모델링 자체를 설계하고 구현하는 과정만으로도 큰 어려움이 따랐다. 데이터 전처리, 피처 엔지니어링, 모델 학습 및 평가를 반복하는 과정에서 수많은 시행착오를 겪으며, 당시에도 벅차다는 느낌을 강하게 받았다. 그러나 이번 프로젝트는 그러한 경험 위에서 다시 출발했기 때문에, 추천 모델링(Item-CF 기반) 단계까지는 이전 경험을 활용하여 상대적으로 안정적으로 수행할 수 있었다.

하지만 이번 프로젝트의 핵심은 단순한 모델링이 아니었다. Docker를 통한 컨테이너화, Airflow 기반 파이프라인 구축, CI/CD와 AWS 연계라는 새로운 과제가 뒤따랐다. 이 과정에서 기술적 난관이 이어졌다.

1.Docker: 이미지 빌드, 공유 볼륨 마운트, 환경 변수 전달 등 기본 설정조차 꼬이는 경우가 많아, 컨테이너 실행을 반복적으로 실패하며 디버깅에 많은 시간을 쏟았다.
2.Airflow: Windows/WSL 환경 특성으로 인한 로그 출력 불가, DockerOperator의 호스트 연결 문제, Slack Webhook 연동 이슈 등 복합적인 문제들이 발생했다. 특히 docker_url과 네트워크 브리징 오류는 해결에 상당한 시간이 소요되었다.
3.AWS: 클라우드 환경에서 Airflow 및 Docker를 어떻게 안정적으로 연계할지 감을 잡는 데 어려움이 있었고, 온전히 이해하기에는 시간이 부족했다.

이처럼 모델링 이후의 단계는 또 다른 거대한 장벽이었다. 기존의 경험으로는 쉽게 풀리지 않는 문제들이었고, 때로는 학습曲線(learning curve)을 감당하기 버거울 만큼 가파르게 느껴졌다.

그럼에도 불구하고 이번 프로젝트는 단순히 “모델을 만드는 경험”을 넘어, 실제 서비스 환경에서 머신러닝을 운영하기 위해 무엇이 필요한지를 직접 체험할 수 있는 소중한 기회였다. 아직 Docker, Airflow, AWS 모두 능숙하다고 말하기는 어렵지만, 적어도 문제를 정의하고 해결 방법을 탐색하며, 제한된 환경에서도 파이프라인을 끝까지 구현해낸 경험은 나의 역량을 한층 확장시켰다.

앞으로는 이 경험을 토대로 더 다양한 프로젝트를 통해 MLOps 실무 능력을 보완하고, 궁극적으로는 데이터 사이언스와 엔지니어링을 잇는 다리 역할을 수행할 수 있는 전문성을 키워가고자 한다.

---

