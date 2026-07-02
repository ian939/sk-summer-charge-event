# 여름 휴게소 충전 이벤트 — 동선 위 SK일렉링크 충전소 찾기

일렉링크 앱 이벤트 페이지의 **웹뷰**로 띄우는 정적 서비스. 출발지·도착지를 넣으면 경로 위 SK일렉링크 고속도로 휴게소 충전소를 지도에 보여주고, 충전소 상세(용량별 기수·대표메뉴·리뷰 블로그)를 제공한다.

**배포:** https://ian939.github.io/sk-summer-charge-event/

## 구성

| 경로 | 역할 |
|---|---|
| `index.html` | 이벤트 랜딩(프로모션). 지도앱을 `<iframe src="app.html">`로 임베드 |
| `app.html` | 경로 탐색 지도앱(핵심). Kakao 지도 + 하단 "경로 찾기" 시트 |
| `sk_chargers.json`, `food.json` | 지도앱 데이터(빌드 산출물) |
| `route/` | 로컬 개발 소스 — `index.html`(app 소스, `__JSKEY__` 치환 전), `promo.html`, `server.py`(로컬 서버·Kakao 길찾기 프록시), 빌드 스크립트 |
| `worker/` | Cloudflare Worker — `/route`(Kakao 길찾기), `/blogs`(네이버 블로그) 프록시. REST 키 은닉 |

## 배포 (GitHub Pages)
`master` push → Pages가 루트를 서빙. 사이트 수정은 `route/*` 수정 후 루트로 반영·커밋.

- `app.html`은 `route/index.html`에서 `__JSKEY__`→JS키, `/route`→Worker URL, 절대경로→상대경로로 치환해 생성.
- `index.html`(랜딩)은 `route/promo.html` 사본.

## 데이터 갱신
- `python route/build_sk.py` — `SK일렉링크_..._현황.ver3_주말혼잡도.xlsx` → `sk_chargers.json` (route/·루트 이중 저장)
- `EX_API_KEY=... python route/build_food.py` — 도로공사 OpenAPI → `food.json`

## Worker
```
cd worker && npx wrangler deploy
```
시크릿: `KAKAO_REST_KEY`, `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET` (`wrangler secret put`). 이미 배포됨: `highway-ev-route-proxy.ian-1f3.workers.dev`

## 키
`.env`(커밋 금지)에 `KAKAO_JS_KEY`(지도)·`KAKAO_REST_KEY`(길찾기)·`EX_API_KEY`(메뉴). 클라이언트 노출은 JS 키뿐(도메인 제한으로 보호). 웹뷰 서빙 도메인은 Kakao Developers 콘솔에 등록 필요.
