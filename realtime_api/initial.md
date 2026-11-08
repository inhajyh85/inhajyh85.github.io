오늘 만들 프로젝트 : 우리동네 날씨 실시간 파악 & 미세먼지 알림판.
필요한 것들 : 아래 정리

---

# 우리동네 날씨 & 미세먼지 알림판 — MVP 설계서

## 1. MVP 목표
접속하면 **내 위치(동네 이름)** 기준으로 **현재 날씨**와 **미세먼지 등급**을 한 화면에 보여주고, 미세먼지가 "나쁨" 이상이면 눈에 띄게 경고한다.

- 형태: **HTML 파일 1개** (HTML + CSS + JavaScript)
- 서버: **없음** (GitHub Pages에 그대로 배포)
- API 키: **없음** (회원가입·키 발급 불필요)
- 비용: **0원**

## 2. 사용할 API (모두 무료 · 키 불필요 · 브라우저 직접 호출 가능)

> 2026-11-08 실제 호출로 확인: 세 API 모두 키 없이 응답하고 `access-control-allow-origin: *` 헤더가 있어 브라우저 `fetch()`로 바로 호출 가능.

| 용도 | API | 키 | 비고 |
|---|---|---|---|
| 내 위치 좌표 | 브라우저 **Geolocation API** (`navigator.geolocation`) | 불필요 | 브라우저 내장. HTTPS 필요 (GitHub Pages는 HTTPS라 OK) |
| 좌표 → 동네 이름 | **BigDataCloud Reverse Geocode Client** | 불필요 | `localityLanguage=ko`로 "서울특별시 · 명동"처럼 한글 반환 |
| 현재 날씨 | **Open-Meteo Forecast API** | 불필요 | 기온·체감온도·습도·풍속·강수·날씨코드 |
| 미세먼지 | **Open-Meteo Air Quality API** | 불필요 | PM10, PM2.5 (μg/m³), 1시간 단위 갱신 |

### 2-1. 동네 이름 (BigDataCloud)
```
GET https://api.bigdatacloud.net/data/reverse-geocode-client
    ?latitude=37.5665&longitude=126.978&localityLanguage=ko
```
응답에서 사용할 필드:
```json
{ "principalSubdivision": "서울특별시", "city": "서울특별시", "locality": "명동" }
```
- 표시: `${principalSubdivision} ${locality}` → "서울특별시 명동"
- 주의: 클라이언트(브라우저) 전용 무료 엔드포인트. 서버에서 대량 호출 금지.

### 2-2. 현재 날씨 (Open-Meteo Forecast)
```
GET https://api.open-meteo.com/v1/forecast
    ?latitude=37.5665&longitude=126.978
    &current=temperature_2m,apparent_temperature,relative_humidity_2m,weather_code,wind_speed_10m,precipitation
    &timezone=Asia%2FSeoul
```
응답 예시 (실제 호출 결과):
```json
"current": {
  "time": "2026-11-08T16:30",
  "temperature_2m": 5.2,
  "apparent_temperature": 2.8,
  "relative_humidity_2m": 83,
  "weather_code": 53,
  "wind_speed_10m": 4.7,
  "precipitation": 0.20
}
```
- 데이터는 15분 간격으로 갱신됨 (`interval: 900`)
- `weather_code`는 WMO 코드(숫자) → 아래 표로 한글/이모지 변환

| weather_code | 표시 |
|---|---|
| 0 | ☀️ 맑음 |
| 1, 2 | 🌤️ 구름 조금 |
| 3 | ☁️ 흐림 |
| 45, 48 | 🌫️ 안개 |
| 51, 53, 55, 56, 57 | 🌦️ 이슬비 |
| 61, 63, 65, 66, 67, 80, 81, 82 | 🌧️ 비 |
| 71, 73, 75, 77, 85, 86 | 🌨️ 눈 |
| 95, 96, 99 | ⛈️ 뇌우 |

### 2-3. 미세먼지 (Open-Meteo Air Quality)
```
GET https://air-quality-api.open-meteo.com/v1/air-quality
    ?latitude=37.5665&longitude=126.978
    &current=pm10,pm2_5
    &timezone=Asia%2FSeoul
```
응답 예시 (실제 호출 결과):
```json
"current": { "time": "2026-11-08T16:00", "pm10": 27.1, "pm2_5": 23.9 }
```

**등급은 한국 환경부 기준으로 직접 계산** (μg/m³):

| 등급 | PM10 (미세먼지) | PM2.5 (초미세먼지) | 색 |
|---|---|---|---|
| 😊 좋음 | 0 ~ 30 | 0 ~ 15 | 파랑 `#3b82f6` |
| 🙂 보통 | 31 ~ 80 | 16 ~ 35 | 초록 `#22c55e` |
| 😷 나쁨 | 81 ~ 150 | 36 ~ 75 | 주황 `#f97316` |
| 🚨 매우나쁨 | 151 이상 | 76 이상 | 빨강 `#ef4444` |

- 종합 등급 = PM10 등급과 PM2.5 등급 중 **더 나쁜 쪽**
- 위 예시: PM10 27.1(좋음), PM2.5 23.9(보통) → 종합 **보통**

## 3. 화면 구성 (한 페이지)
```
┌──────────────────────────────┐
│ 📍 서울특별시 명동    [새로고침] │
│ 마지막 갱신: 16:30             │
├──────────────────────────────┤
│  🌦️ 이슬비                    │
│  5.2°C  (체감 2.8°C)           │
│  습도 83% · 바람 4.7km/h · 강수 0.2mm │
├──────────────────────────────┤
│  미세먼지 종합: 🙂 보통         │  ← 등급색 배경
│  PM10  27 μg/m³  좋음          │
│  PM2.5 24 μg/m³  보통          │
├──────────────────────────────┤
│ 🚨 (나쁨 이상일 때만) 마스크 챙기세요! │
└──────────────────────────────┘
```

## 4. 동작 흐름
1. 페이지 열림 → `navigator.geolocation.getCurrentPosition()`으로 위도·경도 획득
   - 위치 권한 거부/실패 시 → **기본 위치(서울시청 37.5665, 126.978)** 사용 + 안내 문구
2. 좌표로 3개 API를 **동시에** 호출 (`Promise.all`)
3. 결과를 화면에 표시, 미세먼지 등급 계산 → 색상/경고 표시
4. **10분마다 자동 새로고침** (`setInterval`) + 수동 새로고침 버튼
   - 날씨 15분, 미세먼지 1시간 단위 갱신이라 그보다 자주 호출할 필요 없음

## 5. 핵심 코드 뼈대
```html
<script>
const DEFAULT = { lat: 37.5665, lon: 126.978 }; // 서울시청

function getPosition() {
  return new Promise((resolve) => {
    if (!navigator.geolocation) return resolve(DEFAULT);
    navigator.geolocation.getCurrentPosition(
      (p) => resolve({ lat: p.coords.latitude, lon: p.coords.longitude }),
      () => resolve(DEFAULT),           // 거부/실패 시 기본 위치
      { timeout: 8000 }
    );
  });
}

async function load() {
  const { lat, lon } = await getPosition();
  const q = `latitude=${lat}&longitude=${lon}`;
  const [place, weather, air] = await Promise.all([
    fetch(`https://api.bigdatacloud.net/data/reverse-geocode-client?${q}&localityLanguage=ko`).then(r => r.json()),
    fetch(`https://api.open-meteo.com/v1/forecast?${q}&current=temperature_2m,apparent_temperature,relative_humidity_2m,weather_code,wind_speed_10m,precipitation&timezone=Asia%2FSeoul`).then(r => r.json()),
    fetch(`https://air-quality-api.open-meteo.com/v1/air-quality?${q}&current=pm10,pm2_5&timezone=Asia%2FSeoul`).then(r => r.json()),
  ]);
  render(place, weather.current, air.current);
}

// 등급: 0=좋음 1=보통 2=나쁨 3=매우나쁨
const gradePM10 = (v) => v <= 30 ? 0 : v <= 80 ? 1 : v <= 150 ? 2 : 3;
const gradePM25 = (v) => v <= 15 ? 0 : v <= 35 ? 1 : v <= 75 ? 2 : 3;

load();
setInterval(load, 10 * 60 * 1000);
</script>
```

## 6. 작업 순서 (체크리스트)
- [ ] `realtime_api/index.html` 생성, 레이아웃/카드 UI 작성
- [ ] 위치 가져오기 + 기본 위치 대체 처리
- [ ] 3개 API 호출 및 화면 표시
- [ ] 날씨코드 → 한글/이모지 변환
- [ ] 미세먼지 등급 계산, 색상, "나쁨 이상" 경고 배너
- [ ] 로딩 표시 / API 실패 시 에러 문구
- [ ] 10분 자동 갱신 + 새로고침 버튼 + 마지막 갱신 시각
- [ ] 모바일 화면 확인
- [ ] GitHub Pages 배포 → `https://inhajyh85.github.io/realtime_api/`

## 7. 알아둘 한계 & 다음 단계
- **미세먼지 값은 "측정값"이 아니라 "모델 추정값"**: Open-Meteo는 유럽 CAMS 대기질 모델(격자 단위) 기반이라 에어코리아 측정소 수치와 차이가 날 수 있음. MVP용으로는 충분.
- **정확한 측정소 값이 필요해지면 (MVP 이후)**: 공공데이터포털 **에어코리아 대기오염정보 API**로 교체
  - 서비스키 발급 필요, 위경도 → TM 좌표 변환 후 근처 측정소 조회 → 측정소별 실시간 값 조회의 2단계
  - 브라우저 직접 호출 시 CORS 문제 + 서비스키 노출 → **서버/프록시(예: Cloudflare Workers) 필요** → MVP에서는 제외
- **사용 조건**: Open-Meteo는 비상업적 용도 무료(출처 표기 권장, 데이터 CC BY 4.0) → 화면 하단에 `Weather & Air data by Open-Meteo.com` 표기
- **알림 확장 아이디어**: 브라우저 Notification API로 "나쁨" 전환 시 푸시 알림, 내일 예보(`hourly=pm10,pm2_5`) 추가

## 참고 출처
- Open-Meteo 소개/이용조건: https://open-meteo.com/en/about
- Open-Meteo Air Quality API: https://open-meteo.com/en/docs/air-quality-api
- Open-Meteo Forecast API: https://open-meteo.com/en/docs
- BigDataCloud 무료 클라이언트 역지오코딩: https://www.bigdatacloud.com/docs/api/free-reverse-geocode-to-city-api
- 에어코리아 측정소별 실시간 대기오염정보 (공공데이터포털): https://www.data.go.kr/data/15156659/openapi.do
