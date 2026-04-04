README에 이번 프로젝트의 **아키텍처와 흐름을 시각적으로 설명**하는 내용을 넣으면, 보는 사람이 "이 개발자는 전체 시스템을 완벽히 이해하고 있구나"라는 인상을 받게 됩니다.

가장 전문적으로 보일 수 있는 **README 상세 구성 가이드**를 짜드릴게요. 아래 내용을 그대로 복사해서 수정해 보세요!

---

# 🚀 AWS ECS Fargate CI/CD Pipeline Project

이 저장소는 **GitHub, AWS CodePipeline, ECS Fargate**를 이용한 서버리스 컨테이너 배포 자동화 아키텍처를 담고 있습니다.

## 🏗️ 1. 전체 아키텍처 (Architecture)

본 프로젝트는 코드의 수정부터 실제 배포까지 사람의 개입 없이 이루어지는 **Full CI/CD 파이프라인**을 구현합니다.



1.  **Source:** GitHub Main 브랜치에 `push` 발생 시 Webhook을 통해 파이프라인 트리거.
2.  **Build:** **AWS CodeBuild**가 `buildspec.yml`을 참조하여 Docker 이미지 빌드 및 **Amazon ECR** 푸시.
3.  **Deploy:** **Amazon ECS**가 `imagedefinitions.json`을 바탕으로 서비스의 이미지를 최신화 (Rolling Update).

---

## 📂 2. 핵심 파일 및 디렉토리 구조 (File Structure)

```text
.
├── app.py              # Flask 기반 메인 애플리케이션 소스
├── requirements.txt    # Python 의존성 라이브러리 목록
├── Dockerfile          # 컨테이너 이미지 빌드 정의서
├── buildspec.yml       # CodeBuild 단계별 실행 명령어 (CI 로직)
├── network.yml         # VPC, Subnet, ALB 등 인프라 구성을 위한 CloudFormation
└── README.md           # 프로젝트 가이드라인
```

---

## 🛠️ 3. CI/CD 상세 프로세스 (CI/CD Pipeline)

### **A. 빌드 단계 (CodeBuild)**
`buildspec.yml`을 통해 다음 작업이 자동 수행됩니다.
* **Pre-build:** ECR 저장소 로그인을 위한 인증 토큰 획득.
* **Build:** `docker build`를 통한 이미지 생성 및 태깅.
* **Post-build:** 생성된 이미지를 ECR에 `push`하고, ECS 배포용 `imagedefinitions.json` 파일을 아티팩트로 생성.

### **B. 배포 단계 (ECS Rolling Update)**
* **Zero-Downtime:** 새로운 버전의 컨테이너가 정상 실행(Healthy)될 때까지 기존 버전을 유지하여 서비스 중단을 방지합니다.
* **Fargate:** 서버 관리 부담 없이 컨테이너 단위로 리소스를 운영합니다.

---

## 🔐 4. 보안 설정 (Security)

현업 수준의 보안을 위해 **보안 그룹(Security Group) 체이닝** 기법을 적용했습니다.



* **ALB SG:** 외부(0.0.0.0/0)로부터 80포트 유입 허용.
* **ECS SG:** 오직 **ALB 보안 그룹**으로부터 오는 5000포트 트래픽만 허용하여 컨테이너 직접 노출 방지.

---

## 🚀 5. 시작하기 (How to Run)

1. **인프라 구축:** `network.yml`을 사용하여 CloudFormation 스택을 생성합니다.
2. **ECR 준비:** 저장소를 생성하고 최초 이미지를 수동으로 푸시하여 ECS 서비스를 기동합니다.
3. **파이프라인 연결:** CodePipeline을 생성하여 GitHub 레포지토리와 연동합니다.

---

## 💡 6. 주요 트러블슈팅 (Troubleshooting)

### **1. Mac(M1/M2) 포트 충돌 이슈**
* **문제:** Mac의 AirPlay 기능이 5000번 포트를 선점하여 로컬 테스트 불가.
* **해결:** `defaults write com.apple.controlcenter "NSStatusItem Visible AirPlayReceiver" -bool false` 명령으로 수신 모드 해제 또는 도커 포트 포워딩(`5001:5000`) 활용.

### **2. IAM 권한 에러**
* **문제:** ECS가 ECR 이미지를 가져오지 못하는 에러 발생.
* **해결:** **Task Execution Role**에 `AmazonECSTaskExecutionRolePolicy` 정책을 연결하여 이미지 풀링 및 로그 기록 권한 부여.

---

### ✍️ 작성 팁
* **이미지 추가:**  라고 적힌 부분에 실제 사용자님의 AWS 콘솔 화면이나 아키텍처 다이어그램 캡처본을 넣으면 완벽합니다.
* **결과 확인:** 마지막에 서비스가 정상 작동하는 페이지의 URL이나 캡처본을 "Result" 섹션으로 추가해 보세요!

이렇게 작성하시면 포트폴리오로 활용하기에도 아주 훌륭한 문서가 됩니다. 🚀 다른 세부 내용을 추가하고 싶으신가요?