# OpenClaw 소프트웨어 구조 개요

## 개요

OpenClaw는 다양한 메시징 채널에서 작동하는 개인 AI 어시스턴트입니다. 이 문서는 프로젝트의 전체 소프트웨어 구조를 한국어로 설명합니다.

---

## 핵심 아키텍처

```
메시징 채널 (WhatsApp, Telegram, Slack, Discord 등)
        │
        ▼
┌───────────────────────────────┐
│            Gateway            │
│       (제어 평면)             │
│     ws://127.0.0.1:18789     │
└──────────────┬────────────────┘
               │
               ├─ Pi 에이전트 (RPC)
               ├─ CLI (openclaw ...)
               ├─ WebChat UI
               ├─ macOS 앱
               └─ iOS / Android 노드
```

---

## 주요 디렉토리 구조

### `/src/cli` - 명령줄 인터페이스

- **역할**: 사용자 명령줄 도구
- **주요 파일**:
  - `program/build-program.ts`: CLI 프로그램 구성
  - `command-registry.ts`: 명령 등록
  - `deps.ts`: 의존성 주입
- **명령**:
  - `openclaw gateway`: Gateway 서버 실행
  - `openclaw agent`: AI 에이전트 실행
  - `openclaw message send`: 메시지 전송
  - `openclaw channels`: 채널 관리
  - `openclaw onboard`: 온보딩 위저드

---

### `/src/gateway` - WebSocket 제어 평면

- **역할**: 모든 클라이언트, 툴, 이벤트를 위한 단일 WebSocket 제어 평면
- **주요 파일**:
  - `server.ts`: Gateway 서버 메인
  - `server-chat.ts`: 채팅 처리
  - `server-channels.ts`: 채널 연결 관리
  - `server-browser.ts`: 브라우저 제어
  - `server-node-events.ts`: 노드 이벤트 처리
  - `server-cron.ts`: 크론 작업 관리
  - `server-plugins.ts`: 플러그인 로드
- **기능**:
  - 세션 관리
  - 채널 라우팅
  - 툴 실행
  - 이벤트 브로드캐스팅
  - 웹훅 처리

---

### `/src/channels` - 채널 공통 인터페이스

- **역할**: 모든 메시징 채널의 공통 인터페이스 및 기능
- **주요 파일**:
  - `registry.ts`: 채널 레지스트리
  - `session.ts`: 채널 세션 관리
  - `command-gating.ts`: 명령 게이팅 (승인)
  - `allowlist-match.ts`: 허용 목록 매칭
  - `ack-reactions.ts`: 확인 리액션
  - `mention-gating.ts`: 언급 게이팅

---

### `/src/telegram` - Telegram 채널

- **역할**: Telegram 봇 통합 (grammy 라이브러리 사용)
- **주요 파일**:
  - `monitor.ts`: Telegram 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트
  - `audit.ts`: 그룹 멤버십 감사

---

### `/src/discord` - Discord 채널

- **역할**: Discord 봇 통합 (discord.js 사용)
- **주요 파일**:
  - `monitor.ts`: Discord 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트
  - `audit.ts`: 채널 권한 감사

---

### `/src/slack` - Slack 채널

- **역할**: Slack 봇 통합 (@slack/bolt 사용)
- **주요 파일**:
  - `monitor.ts`: Slack 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트

---

### `/src/signal` - Signal 채널

- **역할**: Signal 통합 (signal-cli 사용)
- **주요 파일**:
  - `monitor.ts`: Signal 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트

---

### `/src/whatsapp` - WhatsApp 채널

- **역할**: WhatsApp 통합 (Baileys Web 사용)
- **주요 파일**:
  - `normalize.ts`: 메시지 정규화
  - **참고**: 실제 구현은 `/src/web/outbound.ts`와 `/src/channel-web.ts`를 통해

---

### `/src/feishu` - Feishu (Lark) 채널

- **역할**: Feishu/비즈니스 봇 통합
- **주요 파일**:
  - `monitor.ts`: Feishu 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `bot.ts`: 봇 클라이언트
  - `client.ts`: Feishu API 클라이언트

---

### `/src/line` - LINE 채널

- **역할**: LINE 봇 통합
- **주요 파일**:
  - `monitor.ts`: LINE 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트
  - `template-messages.ts`: 템플릿 메시지

---

### `/src/imessage` - iMessage 채널

- **역할**: iMessage 통합 (레거시, macOS 전용)
- **주요 파일**:
  - `monitor.ts`: iMessage 메시지 모니터링
  - `send.ts`: 메시지 전송
  - `probe.ts`: 연결 테스트

---

### `/src/agents` - AI 에이전트 런타임

- **역할**: Pi 에이전트 런타임 (RPC 모드)와 관련 기능
- **주요 파일**:
  - `workspace.ts`: 워크스페이스 관리
  - `skills.ts`: 스킬 시스템
  - `bash-tools.ts`: Bash 명령 툴
  - `channel-tools.ts`: 채널 액션 툴
  - `auth-profiles/`: 인증 프로필 관리
  - `cli-backends.ts`: CLI 백엔드 (Claude, OpenAI 등)
  - `claude-cli-runner.ts`: Claude CLI 실행기
  - `subagent-registry.ts`: 서브에이전트 레지스트리
- **기능**:
  - 시스템 프롬프트 관리
  - 툴 실행 및 정책
  - 컨텍스트 창 관리
  - 세션 압축
  - 모델 카탈로그

---

### `/src/media` - 미디어 처리 파이프라인

- **역할**: 이미지, 오디오, 비디오 처리
- **주요 파일**:
  - `store.ts`: 미디어 저장소
  - `parse.ts`: 미디어 파싱
  - `fetch.ts`: 원격 미디어 가져오기
  - `mime.ts`: MIME 타입 감지
  - `image-ops.ts`: 이미지 조작
  - `audio.ts`: 오디오 처리
  - `audio-tags.ts`: 오디오 메타데이터
  - `server.ts`: 미디어 서버
  - `host.ts`: 미디어 호스팅

---

### `/src/plugins` - 플러그인 시스템

- **역할**: 확장 채널 및 기능을 위한 플러그인 시스템
- **주요 파일**:
  - `loader.ts`: 플러그인 로더
  - `runtime/`: 플러그인 런타임 API
  - `registry.ts`: 플러그인 레지스트리
  - `install.ts`: 플러그인 설치
  - `manifest.ts`: 플러그인 매니페스트
- **플러그인 런타임 API** (`runtime/index.ts`):
  - `config`: 구성 관리
  - `system`: 시스템 기능
  - `media`: 미디어 처리
  - `tts`: 텍스트 음성 변환
  - `tools`: 툴 생성
  - `channel`: 채널 통합 API
  - `logging`: 로깅
  - `state`: 상태 디렉토리

---

### `/src/routing` - 메시지 라우팅

- **역할**: 메시지를 적절한 에이전트로 라우팅
- **주요 파일**:
  - `resolve-route.ts`: 에이전트 라우트 해결
  - `bindings.ts`: 라우팅 바인딩
  - `session-key.ts`: 세션 키 생성
- **라우팅 순서** (우선순위):
  1. 피어 바인딩 (DM/그룹)
  2. 상위 피어 바인딩 (스레드)
  3. 길드 바인딩 (Discord)
  4. 팀 바인딩 (Slack)
  5. 계정 바인딩
  6. 채널 바인딩 (모든 계정)
  7. 기본값

---

### `/src/acp` - Agent Client Protocol

- **역할**: 에이전트 클라이언트 프로토콜 서버
- **주요 파일**:
  - `server.ts`: ACP 서버
  - `session.ts`: 세션 관리
  - `client.ts`: ACP 클라이언트
  - `event-mapper.ts`: 이벤트 매핑
  - `session-mapper.ts`: 세션 매핑

---

### `/src/config` - 구성 관리

- **역할**: 사용자 구성 관리
- **주요 파일**:
  - `config.ts`: 구성 로드/저장
  - `sessions.ts`: 세션 스토어 관리
  - `paths.ts`: 파일 경로 해결
  - `group-policy.ts`: 그룹 정책

---

### `/src/wizard` - 온보딩 위저드

- **역할**: 새 사용자를 위한 온보딩 프로세스
- **주요 파일**:
  - `onboarding.ts`: 온보딩 메인
  - `prompts.ts`: 사용자 프롬프트
  - `session.ts`: 위저드 세션
  - `onboarding.gateway-config.ts`: Gateway 설정
  - `onboarding.finalize.ts`: 최종화

---

### `/src/browser` - 브라우저 제어

- **역할**: 브라우저 자동화 툴
- **주요 파일**:
  - `server.ts`: 브라우저 서버
  - `screenshot.ts`: 스크린샷
  - `target-id.ts`: 타겟 ID 관리

---

### `/src/canvas-host` - Canvas 호스트

- **역할**: 에이전트 주도 시각적 작업 공간
- **주요 파일**:
  - `server.ts`: Canvas 서버
  - `a2ui.ts`: A2UI 컴포넌트

---

### `/src/node-host` - 노드 호스트

- **역할**: iOS/Android/macOS 노드 연결
- **주요 파일**:
  - 노드 연결 및 통신 관련 파일

---

### `/src/logging` - 로깅

- **역할**: 구조화된 로깅 시스템
- **주요 파일**:
  - `logger.ts`: 로거
  - `console.ts`: 콘솔 캡처
  - `levels.ts`: 로그 레벨
  - `subsystem.ts`: 서브시스템 로깅
  - `redact.ts`: 민감 정보 레다이션

---

### `/src/cron` - 크론 작업

- **역할**: 스케줄링된 작업 관리
- **주요 파일**:
  - 크론 작업 관련 파일

---

### `/src/memory` - 메모리 시스템

- **역할**: 벡터 기반 메모리 저장
- **주요 파일**:
  - 메모리 검색 및 저장 관련 파일

---

### `/src/pairing` - 페어링

- **역할**: 알 수 없는 발신자 페어링
- **주요 파일**:
  - `pairing-messages.ts`: 페어링 메시지
  - `pairing-store.ts`: 페어링 스토어

---

### `/src/security` - 보안

- **역할**: 보안 정책 및 검증
- **주요 파일**:
  - 보안 관련 파일

---

### `/src/compat` - 호환성

- **역할**: 이전 버전과의 호환성
- **주요 파일**:
  - 호환성 계층 파일

---

### `/src/daemon` - 데몬

- **역할**: 백그라운드 서비스 관리
- **주요 파일**:
  - 데몬 관련 파일

---

### `/src/docker-setup` - Docker 설정

- **역할**: Docker 환경 설정
- **주요 파일**:
  - `docker-setup.test.ts`: Docker 설정 테스트

---

### `/src/auto-reply` - 자동 응답

- **역할**: 자동 응답 시스템
- **주요 파일**:
  - `reply/`: 응답 디스패처
  - `chunk.ts`: 텍스트 청킹
  - `command-detection.ts`: 명령 감지

---

### `/src/tui` - 터미널 UI

- **역할**: 터미널 기반 사용자 인터페이스
- **주요 파일**:
  - TUI 관련 파일

---

### `/src/macOS` - macOS 특정

- **역할**: macOS 전용 기능
- **주요 파일**:
  - macOS 관련 파일

---

### `/src/tts` - 텍스트 음성 변환

- **역할**: TTS (Text-to-Speech) 기능
- **주요 파일**:
  - `tts.ts`: 텍스트 음성 변환

---

### `/src/infra` - 인프라

- **역할**: 기반 인프라 유틸리티
- **주요 파일**:
  - `ports.ts`: 포트 관리
  - `runtime-guard.ts`: 런타임 보호
  - `env.ts`: 환경 변수
  - `binaries.ts`: 바이너리 관리

---

## 데이터 흐름

### 1. 메시지 수신

```
채널 (Telegram/Discord/Slack 등)
    │
    ▼
채널 모니터 (monitor.ts)
    │
    ▼
채널 공통 레이어 (channels/)
    │
    ▼
Gateway (server-chat.ts)
    │
    ▼
라우팅 (routing/resolve-route.ts)
    │
    ▼
에이전트 세션 선택
    │
    ▼
Pi 에이전트 (agents/)
```

### 2. 메시지 전송

```
Pi 에이전트 응답
    │
    ▼
Gateway (server-chat.ts)
    │
    ▼
채널 전송 레이어 (send.ts)
    │
    ▼
채널 API 호출
    │
    ▼
사용자에게 전송
```

### 3. 툴 실행

```
에이전트 툴 요청
    │
    ▼
Gateway (server-browser.ts, server-node-events.ts 등)
    │
    ▼
툴 실행기 (bash-tools.ts, browser/ 등)
    │
    ▼
시스템/브라우저/노드
    │
    ▼
결과 반환
```

---

## 통합 지점

### 1. 채널 플러그인

- **위치**: `extensions/` (외부 플러그인)
- **인터페이스**: `src/plugin-sdk/`
- **등록**: 플러그인 매니페스트 통해

### 2. 채널 레지스트리

- **위치**: `src/channels/registry.ts`
- **역할**: 모든 채널을 등록하고 관리

### 3. 에이전트 백엔드

- **지원**:
  - Claude (Anthropic)
  - OpenAI
  - OpenRouter
  - Bedrock (AWS)
  - 기타 OpenAI 호환

### 4. 노드 연결

- **macOS 노드**: 메뉴 바 앱, Voice Wake, Canvas
- **iOS 노드**: Canvas, Voice Wake, 카메라
- **Android 노드**: Canvas, 카메라, 화면 녹화

---

## 주요 아키텍처 패턴

### 1. 제어 평면 (Gateway)

- 단일 WebSocket 제어 평면
- 세션 관리 중앙화
- 이벤트 브로드캐스팅

### 2. 채널 추상화

- 공통 인터페이스 (`src/channels/`)
- 개별 채널 구현 (`src/telegram/`, `src/discord/` 등)
- 플러그인 확장 지원

### 3. 세션 모델

- `main` 세션 (직접 대화)
- 그룹 세션 격리
- 라우팅 기반 세션 선택

### 4. 플러그인 런타임

- 확장 가능한 아키텍처
- 런타임 API 제공
- 매니페스트 기반 설치

### 5. 미디어 파이프라인

- 원격 미디어 가져오기
- 로컬 저장
- MIME 타입 감지
- 이미지 조작

---

## 보안 고려사항

1. **DM 페어링**: 기본값으로 알 수 없는 발신자에게 페어링 코드 전송
2. **허용 목록**: 채널별 허용 목록
3. **명령 게이팅**: 명령 실행 권한 관리
4. **인증 프로필**: 여러 인증 방법 (OAuth, API 키)
5. **세션 격리**: 그룹 세션 분리

---

## 의존성 주요 라이브러리

- **프레임워크**:
  - `commander`: CLI 프레임워크
  - `hono`: 웹 프레임워크
  - `express`: 웹 서버

- **채널**:
  - `grammy`: Telegram
  - `discord.js`: Discord
  - `@slack/bolt`: Slack
  - `signal-utils`: Signal
  - `@whiskeysockets/baileys`: WhatsApp

- **에이전트**:
  - `@mariozechner/pi-agent-core`: Pi 에이전트 런타임
  - `@mariozechner/pi-ai`: Pi AI
  - `@mariozechner/pi-coding-agent`: Pi 코딩 에이전트

- **미디어**:
  - `sharp`: 이미지 처리
  - `file-type`: 파일 타입 감지

- **기타**:
  - `zod`: 유효성 검사
  - `ws`: WebSocket
  - `yaml`: YAML 구성

---

## 설정 파일 위치

- **구성**: `~/.openclaw/openclaw.json`
- **자격 증명**: `~/.openclaw/credentials/`
- **세션**: `~/.openclaw/sessions/`
- **워크스페이스**: `~/.openclaw/workspace/`

---

## 추가 문서

- [시작하기](https://docs.openclaw.ai/start/getting-started)
- [아키텍처 개요](https://docs.openclaw.ai/concepts/architecture)
- [채널](https://docs.openclaw.ai/channels)
- [구성 참조](https://docs.openclaw.ai/gateway/configuration)
- [Gateway 운영](https://docs.openclaw.ai/gateway)
