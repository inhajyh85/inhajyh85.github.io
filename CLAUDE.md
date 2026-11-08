# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

GitHub Pages 사용자 사이트 저장소다 (`origin` = `inhajyh85/inhajyh85.github.io`). `main`에 푸시하면 그대로 `https://inhajyh85.github.io/`에 배포된다. 빌드 도구, 패키지 매니저, 테스트, 린터는 없다. 모든 페이지는 **HTML 파일 1개(HTML + CSS + JS 인라인)** 로 만들고 서버 없이 정적으로 동작한다.

- `index.html` — 루트 자기소개 페이지 (`https://inhajyh85.github.io/`)
- `<프로젝트>/index.html` — 미니 프로젝트 하나당 폴더 하나 (`https://inhajyh85.github.io/<프로젝트>/`)
- `<프로젝트>/initial.md` — 해당 프로젝트의 MVP 설계서 (한국어)

## 작업 흐름: initial.md → index.html

새 프로젝트는 항상 폴더 안의 `initial.md` 설계서를 먼저 읽고 그에 맞춰 같은 폴더에 `index.html`을 만든다. `realtime_api/initial.md`가 기준 예시다. 설계서는 보통 다음을 담는다: MVP 목표, 사용할 API(실제 호출로 확인한 요청/응답 예시), 화면 와이어프레임, 동작 흐름, 코드 뼈대, 체크리스트, 한계·다음 단계. 설계서의 수치(등급 기준표, 색상 코드, 갱신 주기, 기본값 등)는 그대로 구현하고, 체크리스트 항목을 모두 충족시킨다.

기능을 추가할 때도 같은 순서다: 먼저 해당 `initial.md`에 기능·데이터 형식·체크리스트 항목을 추가하고, 구현 후 체크리스트를 체크한다. 설계서가 비어 있으면 사용자에게 요구사항을 확인한다.

프로젝트: `realtime_api/`(날씨·미세먼지 알림판, 외부 API), `diary/`(일기장, localStorage 저장 + JSON 백업 파일 내보내기/불러오기).

## 지켜온 제약과 관례

- **API 키·서버 없이** 브라우저 `fetch()`로 직접 호출 가능한 무료 API만 쓴다 (CORS `access-control-allow-origin: *` 확인). 키나 프록시가 필요한 것은 MVP 범위 밖으로 두고 설계서의 "다음 단계"에 적는다.
- 외부 데이터를 쓰면 페이지 하단에 출처 표기 (예: `Weather & Air data by Open-Meteo.com`).
- UI 텍스트·주석·커밋 메시지는 한국어. 폰트는 Google Fonts `Noto Sans KR`.
- 색상은 `:root` CSS 변수로 정의하고 다크 모드를 지원한다. 루트 `index.html`의 패턴: `@media (prefers-color-scheme: dark)` 안의 `:root:not([data-theme="light"])` + 수동 선택용 `:root[data-theme="dark"]`, 선택값은 `localStorage('theme')`에 저장(try/catch로 감쌈).
- 모바일 화면에서도 깨지지 않게 반응형으로 작성.
- 새 프로젝트를 추가하면 커밋 단위는 "프로젝트 폴더의 `initial.md` + `index.html`" 한 번이 관례다.

## 로컬 확인

빌드 없이 파일을 브라우저로 열면 된다. 다만 Geolocation 등 보안 컨텍스트가 필요한 API는 `file://`에서 막힐 수 있으므로 로컬 서버로 확인한다. 이 PC에는 Python 3.14가 사용자 범위로 설치되어 있다. `python`이 Microsoft Store 별칭으로 잡힐 수 있으므로 `py` 런처를 쓴다:

```
py -m http.server 8000   # → http://localhost:8000/<프로젝트>/
```

Playwright MCP로 화면을 확인할 수 있으며, 그 산출물 폴더 `.playwright-mcp/`는 `.gitignore`에 들어 있다.
