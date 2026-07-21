# ShowMeTheMoney-infra

## 프로젝트 개요
1차 프로젝트에서 구축한 개인 자산 관리 플랫폼 "Show Me The Money"의 인프라를 AWS 기반으로 전면 전환하는 2차 프로젝트입니다.

- 1차 프로젝트: https://github.com/YeonWoojuice/ShowMeTheMoney/tree/main

- **인프라 전환**: VMware Fusion 기반 온프레미스 Kubernetes → AWS EKS
- **데이터베이스**: RDS MySQL 적용으로 백업·복구 체계 강화
- **IaC**: Terraform 기반으로 인프라 재현성 확보
- **CI/CD**: GitHub Actions + ECR + Helm으로 이미지 빌드·패키징 자동화, ArgoCD 기반 GitOps 배포
- **모니터링**: CloudWatch 도입으로 운영 안정성 강화
- **서비스 기능**: 1차 MVP(회원가입/로그인, 수입·지출 CRUD, 예산, 대시보드 등)를 유지하되, 1차에서 미흡했던 프론트-백엔드 API 스펙 사전 합의로 통합 안정성 개선

단순 가계부 MVP를 넘어 운영 가능한 클라우드 기반 금융 관리 서비스로 확장하는 것이 목표입니다.

## 진행 상태
- 최종 목표(EKS/Terraform/ArgoCD)로 가기 전 임시 단계로, 현재는 **EC2 + Docker Compose + RDS + nginx** 조합으로 배포되어 있습니다.
- 1차 프로젝트의 backend/frontend 코드를 이관했고, DB만 컨테이너 대신 RDS를 바라보도록 구성을 바꿨습니다.
- 세부 구현(k8s manifest 작성, Terraform 코드, CI/CD 파이프라인 구체화 등)은 팀 논의 후 진행 예정입니다.

## 폴더 구조
- `frontend/`, `backend/` — 1차 프로젝트에서 이관한 애플리케이션 코드
- `docs/` — 아키텍처 문서, 운영 가이드 (추후 작성)

## 현재 배포 구조 (EC2 + Docker Compose + RDS)

```
Browser
  │  HTTPS (443)
  ▼
nginx (EC2 호스트에 직접 설치, Let's Encrypt 인증서)
  ├─ /api/*  → 127.0.0.1:8080 (backend 컨테이너)
  └─ /*      → 127.0.0.1:3000 (frontend 컨테이너)

backend 컨테이너 ──▶ RDS MySQL (AWS 관리형, docker-compose 밖에 존재)
```

- `db` 서비스(mysql 컨테이너)는 없습니다. backend가 RDS에 직접 접속합니다.
- nginx는 **저장소에 포함되어 있지 않고 EC2 호스트에 직접 설정**되어 있습니다 (`/etc/nginx/conf.d/*.conf`). 서버를 새로 구성할 경우 아래 블록이 반드시 있어야 합니다.

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/api/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### 실행 방법 (서버)

1. `.env.example` → `.env` 복사 후 값 채우기 (루트)
2. `backend/.env.example` → `backend/.env` 복사 후 값 채우기
3. 빌드 및 실행
   ```bash
   docker compose up -d --build
   docker compose logs -f backend   # RDS 연결/Flyway 마이그레이션 확인
   ```

### 환경변수

**루트 `.env`** (frontend 빌드용, `docker-compose.yml`이 직접 참조)

| 변수 | 설명 |
| --- | --- |
| `NEXT_PUBLIC_API_URL` | 브라우저가 API를 호출할 base URL. **반드시 도메인만** 넣기 (예: `https://team4.mang.pe.kr`). `/api`를 붙이면 안 됨 — 프론트 코드가 `/api/auth/login`처럼 각 호출 경로에 이미 `/api`를 포함하고 있어서, 여기 붙이면 `/api/api/...`로 중복돼 404가 남 |

**`backend/.env`**

| 변수 | 설명 |
| --- | --- |
| `DB_HOST` | RDS 엔드포인트만 입력 (전체 JDBC URL 아님) |
| `DB_PORT`, `DB_NAME` | 기본값(3306 / showmethemoney) 그대로 둬도 됨 |
| `DB_USERNAME`, `DB_PASSWORD` | RDS 계정 |
| `JWT_SECRET`, `JWT_EXPIRATION` | JWT 설정 |
| `CORS_ALLOWED_ORIGINS` | 운영 도메인 (예: `https://team4.mang.pe.kr`) |

### 자주 겪는 함정

- **`NEXT_PUBLIC_*` 값은 빌드 타임에 번들에 박힘** — `.env` 고친 뒤 컨테이너만 재시작하면 반영 안 됨. 반드시 `docker compose build frontend`로 재빌드 필요.
- **nginx에 `/api/` 프록시 블록이 없으면** 모든 요청이 frontend(3000)로만 흘러가서, 존재하지 않는 라우트에 대해 Next.js 자체 404 페이지가 뜸 (nginx/backend 에러가 아니라 프론트 서버가 낸 404라 원인 파악이 헷갈리기 쉬움).
- **MySQL 8 인증 방식** 때문에 `DB_URL`(application.yml에서 조합됨)에 `allowPublicKeyRetrieval=true`가 빠지면 RDS 연결 시 `Public Key Retrieval is not allowed` 에러 발생.
