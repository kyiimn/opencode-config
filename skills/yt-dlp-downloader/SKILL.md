---
name: yt-dlp-downloader
description: >
  yt-dlp 명령어를 사용하여 YouTube 영상을 최고 화질로 다운로드하는 스킬.
  사용자가 "유튜브 다운로드", "YouTube 영상 저장", "yt-dlp로 영상 받기", "최고 화질 다운로드",
  "동영상 다운로드", "음악/오디오만 추출", "자막 포함 다운로드" 등을 요청할 때 반드시 이 스킬을 사용하세요.
  URL이 포함되고 다운로드/저장/추출 등의 동작이 언급된 경우에도 반드시 이 스킬을 활용하세요.
---

# yt-dlp 최고화질 다운로드 스킬

## 개요

yt-dlp를 이용해 YouTube(및 기타 영상 사이트) 영상을 최고 화질로 다운로드하는 워크플로우.

---

## Step 0: yt-dlp 설치 확인 및 설치 (Arch Linux)

> 이 스킬은 **Arch Linux** 환경을 기준으로 합니다. 패키지 관리자는 `yay` (AUR) 또는 `pacman`을 사용합니다.

```bash
# yt-dlp 설치 확인 → 없으면 yay로 AUR에서 설치
if ! which yt-dlp &>/dev/null; then
  if which yay &>/dev/null; then
    yay -S --noconfirm yt-dlp
  else
    echo "yay가 없습니다. yay를 먼저 설치하거나, pacman으로 대체 시도합니다."
    sudo pacman -S --noconfirm yt-dlp
  fi
fi

# ffmpeg 확인 (영상+오디오 병합에 필요)
if ! which ffmpeg &>/dev/null; then
  sudo pacman -S --noconfirm ffmpeg
fi
```

**패키지 정보:**
- `yt-dlp` — AUR 패키지: https://aur.archlinux.org/packages/yt-dlp
- `ffmpeg` — 공식 저장소: `sudo pacman -S ffmpeg`

---

## Step 1: 사용자 요청 분석

다운로드 전 사용자 요청에서 다음을 파악하세요:

| 항목 | 내용 |
|------|------|
| **URL** | 다운로드할 영상 URL |
| **목적** | 영상(기본) / 오디오만 / 자막 포함 등 |
| **출력 경로** | `/tmp/` (기본값) |
| **특수 요청** | 플레이리스트, 특정 포맷, 자막 언어 등 |

---

## Step 2: 기본 최고화질 다운로드 명령

### ▶ 표준: 영상+오디오 최고 화질 (권장)

```bash
yt-dlp \
  -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio/best" \
  --merge-output-format mp4 \
  -o "/tmp/%(title)s.%(ext)s" \
  --no-playlist \
  "<URL>"
```

**포맷 선택 우선순위 설명:**
- `bestvideo[ext=mp4]+bestaudio[ext=m4a]` — mp4/m4a 최고 화질 병합 (ffmpeg 필요)
- `bestvideo+bestaudio` — 포맷 무관 최고 화질 병합
- `best` — 병합 없이 단일 최고화질 스트림 (fallback)

---

## Step 3: 상황별 명령어

### 🔊 오디오만 추출 (MP3 최고 품질)

```bash
yt-dlp \
  -x \
  --audio-format mp3 \
  --audio-quality 0 \
  -o "/tmp/%(title)s.%(ext)s" \
  "<URL>"
```

### 📋 자막 포함 다운로드

```bash
yt-dlp \
  -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best" \
  --merge-output-format mp4 \
  --write-sub \
  --write-auto-sub \
  --sub-langs "ko,en" \
  --embed-subs \
  -o "/tmp/%(title)s.%(ext)s" \
  "<URL>"
```

### 📂 플레이리스트 전체 다운로드

```bash
yt-dlp \
  -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best" \
  --merge-output-format mp4 \
  -o "/tmp/%(playlist_title)s/%(playlist_index)02d - %(title)s.%(ext)s" \
  --yes-playlist \
  "<PLAYLIST_URL>"
```

### 🔍 다운로드 전 포맷 목록 확인

```bash
yt-dlp -F "<URL>"
```

### 📌 특정 포맷 ID로 다운로드

```bash
# 예: 포맷 ID 137(video) + 140(audio)
yt-dlp -f "137+140" --merge-output-format mp4 -o "/tmp/%(title)s.%(ext)s" "<URL>"
```

---

## Step 4: 실행 후 처리

1. **다운로드 성공 확인**: 파일이 `/tmp/`에 생성되었는지 확인
2. **파일 정보 출력**: 파일명, 크기, 해상도 등 안내
3. **present_files 호출**: 사용자가 파일을 다운로드할 수 있도록 제공

```bash
# 파일 목록 및 크기 확인
ls -lh /tmp/*.mp4 2>/dev/null || ls -lh /tmp/
```

---

## 주요 옵션 레퍼런스

| 옵션 | 설명 |
|------|------|
| `-f FORMAT` | 포맷 지정 (`bestvideo+bestaudio` = 최고화질) |
| `--merge-output-format mp4` | 병합 후 mp4로 저장 |
| `-o TEMPLATE` | 출력 파일 경로/이름 템플릿 |
| `--no-playlist` | 단일 영상만 다운로드 |
| `--yes-playlist` | 플레이리스트 전체 다운로드 |
| `-x` | 오디오만 추출 |
| `--audio-format mp3` | 오디오 포맷 지정 |
| `--audio-quality 0` | 오디오 최고 품질 (0=최상) |
| `--write-sub` | 수동 자막 다운로드 |
| `--write-auto-sub` | 자동 생성 자막 다운로드 |
| `--embed-subs` | 자막을 영상에 내장 |
| `--cookies-from-browser chrome` | 브라우저 쿠키 사용 (연령 제한 등) |
| `--rate-limit 5M` | 다운로드 속도 제한 (예: 5MB/s) |
| `-F` | 사용 가능한 포맷 목록 확인 |
| `--no-check-certificates` | SSL 인증서 검사 생략 |

---

## 주의사항

- **ffmpeg 필요**: `bestvideo+bestaudio` 병합은 ffmpeg가 필수. 없으면 `best` 단일 스트림으로 fallback.
- **저작권**: 개인적 용도로만 사용. 저작권법을 준수하세요.
- **연령 제한 영상**: `--cookies-from-browser chrome` 옵션 추가 필요.
- **출력 경로**: 기본값은 `/tmp/`. 사용자가 다른 경로를 지정하면 해당 경로를 우선 사용할 것.
- **파일명**: `%(title)s` 템플릿 사용 시 특수문자 자동 처리됨.
