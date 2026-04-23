# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 소개

**RELO** — 서울 이사 후보지 탐색 + 재무 시뮬레이터. 아파트 실거래가를 지도에 표시하고, 관심 지점(POI) 반경 교집합으로 후보지를 좁힌 뒤, 매도/매수 세금과 자산 시뮬레이션을 계산한다.

## 실행 방법

빌드 없음. 바로 브라우저에서 열거나 간단히 서버로 띄우면 된다:

```bash
python3 -m http.server 8080
# http://localhost:8080 접속
```

테스트, 린터, 패키지 매니저 없음.

## 코드 구조

모든 코드가 **`index.html`** 파일 하나에 들어 있다 (~1,650줄).  
순서: `<style>` → `<body>` → `<script>`. 프레임워크·번들러·서버 없음.

### 외부 라이브러리 (CDN)

- **Leaflet.js** v1.9.4 — 지도
- **Turf.js** v6.5.0 — 원 교집합 영역 계산
- **Google Fonts** — Noto Sans KR 등

### 외부 API

| API | 용도 |
|---|---|
| `apis.data.go.kr` RTMSDataSvcAptTrade / AptRent | 아파트 매매·전월세 실거래가 (XML) |
| CoinGecko | XRP·BTC 시세 |
| allorigins / corsproxy.io | CORS 우회 프록시 |

API 서비스키는 스크립트 내 `SERVICE_KEY` 상수에 하드코딩되어 있다.

### 주요 모듈 (모두 `<script>` 안)

| 모듈 | 핵심 함수 | 역할 |
|---|---|---|
| 지도·공간 | `initMap()` `updateRadius()` `updateIntersection()` `renderMarkers()` | Leaflet 지도, POI 3개 원, Turf 교집합 폴리곤, 아파트 마커 |
| 데이터 로딩 | `loadApts()` `parseXML()` `groupApts()` | 6개월 × 5개 법정동코드 × 매매/전세 = 60개 요청(6개씩 청크), XML 파싱, 아파트별 그룹핑 |
| 필터·목록 | `filtered()` `renderList()` `focusApt()` | 가격·면적·교집합 필터, 최대 80건 렌더링 |
| 재무 시뮬레이터 | `buildFinance()` `calcFinance()` `calcTax()` `calcAcqTax()` `fetchCrypto()` | 양도세·취득세 계산, 장기보유특별공제, 코인 자산, 전세/매매 시나리오 |
| UI | `switchTab()` `toggleBS()` `showAptDetail()` | 탭 전환, 모바일 바텀시트 스와이프, 아파트 상세 패널 |

### 상태 관리

- **`localStorage`** — 재무 입력값(`fin_*`), POI 반경(`radius_*`) 저장
- **전역 변수** — `allGrouped`(로드된 전체 아파트), `filters`(가격·면적 범위), `POIS`(POI 좌표+반경), `tradeType`(`'buy'`/`'rent'`), `fin`(재무 시뮬레이터 상태)

### 주요 상수

```js
// 법정동 코드 (5개 구)
LAWD: [11650, 11680, 11200, 11710, 11560]

// 관심 동네
NBHD: ['잠원동','반포동','서초동','압구정동','논현동','신사동']

PYEONG = 3.3058  // 1평 = 3.3058㎡
// 가격 단위: 만원 / 면적 단위: 평
// groupApts() 후 2026년 이후 거래만 유지
```

### 레이아웃

- **데스크톱**: 좌측 사이드바(필터+목록) + 우측 지도/재무 패널
- **모바일(≤768px)**: 전체화면 지도 + 스와이프 바텀시트, 탭으로 지도↔재무 전환
