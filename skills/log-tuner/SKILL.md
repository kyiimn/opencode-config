---
name: log-tuner
description: Strict rules for handling large outputs and binary dumps
---
## 컴팩션 및 로그 처리 규칙
- ImageMagick, Ghostscript, Playwright 등 외부 도구 실행 시 발생하는 긴 로그는 분석 직후 핵심 원인 1~2줄만 남기고 즉시 요약 상태로 전환할 것.
- CMYK 색상 매핑 배열이나 PDF 후처리 과정의 바이너리 덤프 데이터가 터미널에 출력될 경우, 절대 전체 데이터를 컨텍스트에 보존하지 말고 성공/실패 여부만 기록할 것.
- 컴팩션 시 수정된 파일 경로와 테스트 실행 결과의 요약본은 반드시 보존할 것.
