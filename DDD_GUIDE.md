# DDD 프로젝트 구조 가이드

## DDD가 처음이신가요?

**핵심 개념**: DDD는 코드를 "무엇을 하는지"가 아니라 "어떤 역할을 하는지"로 나눕니다.

```
전통적인 구조 (기능 중심):
├── routes/      ← 모든 라우트
├── services/    ← 모든 비즈니스 로직
├── lib/         ← 모든 유틸리티

DDD 구조 (역할 중심):
├── domain/          ← 비즈니스 핵심 규칙
├── application/     ← 사용자 요청 처리
├── infrastructure/  ← 외부 시스템 연동
├── interfaces/      ← HTTP 요청/응답
```

---

## 디렉터리 구조 상세

```
api/src/
│
├── 📁 domain/                      ← ⭐ 비즈니스 핵심 (가장 중요)
│   │
│   ├── 📁 shared/                  # 여러 도메인에서 공통 사용
│   │   └── 📁 value-objects/
│   │       ├── file-id.ts          # 파일 ID (불변 값 객체)
│   │       ├── stored-file.ts      # 저장된 파일 정보
│   │       └── image-metadata.ts   # 이미지 메타데이터
│   │
│   ├── 📁 newspaper/               # 신문 변환 도메인
│   │   ├── conversion.ts           # CMYK 변환 옵션, 결과 엔티티
│   │   ├── interfaces.ts           # 도메인 서비스 인터페이스
│   │   └── newspaper-conversion.service.ts  # 변환 로직
│   │
│   └── 📁 image/                   # 이미지 편집 도메인
│       ├── edited-image.ts         # 편집된 이미지 엔티티
│       ├── interfaces.ts           # 도메인 서비스 인터페이스
│       └── image-edit.service.ts   # 편집 로직
│
├── 📁 application/                 ← ⭐ 요청 처리 (유스케이스)
│   └── 📁 usecases/
│       ├── convert-to-cmyk.usecase.ts   # CMYK 변환 요청 처리
│       ├── edit-image.usecase.ts        # 이미지 편집 요청 처리
│       └── retrieve-file.usecase.ts     # 파일 조회 요청 처리
│
├── 📁 infrastructure/              ← ⭐ 외부 시스템 연동
│   │
│   ├── 📁 storage/
│   │   └── file-storage.repository.ts   # 파일 시스템 저장소
│   │
│   ├── 📁 processing/
│   │   ├── imagemagick.processor.ts  # ImageMagick 명령 실행
│   │   └── eps.converter.ts          # EPS 변환기
│   │
│   ├── 📁 ai/
│   │   └── gemini.client.ts          # Gemini API 클라이언트
│   │
│   ├── 📁 config/
│   │   └── allow-ip-list.ts          # 허용 IP 목록
│   │
│   └── 📁 utils/
│       └── sleep.ts                  # 유틸리티 함수
│
├── 📁 interfaces/                  ← ⭐ HTTP 요청/응답 처리
│   │
│   ├── 📁 routes/
│   │   ├── newspaper.routes.ts        # /newspaper/* 엔드포인트
│   │   ├── image.routes.ts            # /image/* 엔드포인트
│   │   └── data.routes.ts             # /data/* 엔드포인트
│   │
│   └── 📁 middleware/
│       └── allow-ip.middleware.ts     # IP 필터링 미들웨어
│
└── app.ts                          ← 진입점 (의존성 조립)
```

---

## 각 계층의 역할 (한마디로)

| 계층 | 역할 | 비유 |
|------|------|------|
| **domain** | 비즈니스 규칙 | 요리사 (레시피와 요리법을 앎) |
| **application** | 요청 흐름 제어 | 웨이터 (주문을 받아 요리사에게 전달) |
| **infrastructure** | 외부 시스템 연동 | 식자재 공급업체 (재료를 제공) |
| **interfaces** | HTTP 처리 | 홀 서빙 직원 (손님에게 응대) |

---

## 요청 흐름 예시: CMYK 변환

```
┌─────────────────────────────────────────────────────────────────────┐
│  POST /newspaper/cmyk                                               │
│  (이미지 파일 + 변환 옵션)                                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  interfaces/routes/newspaper.routes.ts                              │
│  ├── multer로 파일 업로드 처리                                       │
│  ├── 요청 데이터 파싱                                                │
│  └── ConvertToCmykUseCase 호출                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  application/usecases/convert-to-cmyk.usecase.ts                    │
│  ├── ConversionOptions 값 객체 생성                                  │
│  ├── NewspaperConversionService 호출                                │
│  └── 결과 반환                                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  domain/newspaper/newspaper-conversion.service.ts                   │
│  ├── 이미지 메타데이터 확인 (Gray/CMYK 판단)                         │
│  ├── ImageMagickProcessor로 TIFF 변환                                │
│  ├── EpsConverter로 EPS 변환                                         │
│  └── ConvertedFile 엔티티 반환                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  infrastructure/processing/imagemagick.processor.ts                 │
│  ├── ImageMagick 명령어 실행                                         │
│  └── 실제 파일 변환                                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 파일별 역할 요약

### 📁 domain/ (비즈니스 핵심)

| 파일 | 역할 | 언제 수정? |
|------|------|-----------|
| `conversion.ts` | 변환 옵션, 결과 타입 정의 | 옵션 추가/변경 시 |
| `interfaces.ts` | 서비스 인터페이스 정의 | 새 기능 추가 시 |
| `newspaper-conversion.service.ts` | CMYK 변환 핵심 로직 | 변환 알고리즘 변경 시 |

### 📁 application/usecases/

| 파일 | 역할 | 언제 수정? |
|------|------|-----------|
| `convert-to-cmyk.usecase.ts` | CMYK 변환 요청 처리 | 요청/응답 형식 변경 시 |
| `edit-image.usecase.ts` | 이미지 편집 요청 처리 | 요청/응답 형식 변경 시 |
| `retrieve-file.usecase.ts` | 파일 조회 요청 처리 | 조회 로직 변경 시 |

### 📁 infrastructure/

| 파일 | 역할 | 언제 수정? |
|------|------|-----------|
| `file-storage.repository.ts` | 파일 저장/조회 | 저장소 변경 시 (S3 등) |
| `imagemagick.processor.ts` | ImageMagick 실행 | 변환 옵션 변경 시 |
| `eps.converter.ts` | EPS 변환 | EPS 변환 로직 변경 시 |
| `gemini.client.ts` | Gemini AI 호출 | AI 모델/프롬프트 변경 시 |

### 📁 interfaces/routes/

| 파일 | 역할 | 언제 수정? |
|------|------|-----------|
| `newspaper.routes.ts` | /newspaper/* 엔드포인트 | 새 엔드포인트 추가 시 |
| `image.routes.ts` | /image/* 엔드포인트 | 새 엔드포인트 추가 시 |
| `data.routes.ts` | /data/* 엔드포인트 | 새 엔드포인트 추가 시 |

---

## 새 기능 추가 시 작업 순서

### 예: "PDF 변환 기능 추가"

```
1️⃣ domain/
   ├── pdf/                          ← 새 도메인 폴더
   │   ├── conversion.ts             # 변환 옵션, 결과
   │   ├── interfaces.ts             # IPdfConverter 인터페이스
   │   └── pdf-conversion.service.ts # 변환 로직

2️⃣ infrastructure/
   └── processing/
       └── pdf.converter.ts          # 실제 PDF 변환 구현

3️⃣ application/usecases/
   └── convert-to-pdf.usecase.ts     # 요청 처리

4️⃣ interfaces/routes/
   └── pdf.routes.ts                 # /pdf/* 엔드포인트

5️⃣ app.ts
   └── 새 서비스 등록
```

---

## 핵심 규칙

### ✅ 올바른 의존성 방향
```
interfaces → application → domain
                    ↓
              infrastructure
```

### ❌ 절대 하지 말아야 할 것
```
# domain이 infrastructure를 import ❌
import { GeminiClient } from '@/infrastructure/ai/gemini.client';

# 대신 interface 사용 ✅
import { IAIImageClient } from './interfaces';
```

---

## 빠른 참조: 파일 찾기

| 하고 싶은 것 | 파일 위치 |
|------------|----------|
| CMYK 변환 로직 수정 | `domain/newspaper/newspaper-conversion.service.ts` |
| 변환 옵션 추가 | `domain/newspaper/conversion.ts` |
| API 엔드포인트 추가 | `interfaces/routes/*.routes.ts` |
| 요청 데이터 처리 방식 변경 | `application/usecases/*.usecase.ts` |
| 파일 저장 방식 변경 | `infrastructure/storage/file-storage.repository.ts` |
| AI 프롬프트 수정 | `infrastructure/ai/gemini.client.ts` |
| ImageMagick 옵션 수정 | `infrastructure/processing/imagemagick.processor.ts` |
| IP 필터링 수정 | `infrastructure/config/allow-ip-list.ts` |

---

## 테스트

```bash
# 모든 테스트 실행
npm test

# 특정 파일만 테스트
npm test -- conversion.test.ts

# 테스트 커버리지 확인
npm run test:coverage
```

테스트 파일은 각 도메인 폴더의 `__tests__/` 안에 있습니다:
```
domain/
├── shared/__tests__/file-id.test.ts
└── newspaper/__tests__/conversion.test.ts
```