# Trend Monitor — Setup Guide

> 구글 트렌드 자동 모니터링 시스템 셋업

---

## 🏗 아키텍처 한눈에 보기

```
┌─────────────────────────────────────────────────────────────┐
│                      GitHub Repository                       │
│                                                              │
│  data/keywords.json    ← 진우님이 수정 (모니터링 키워드)        │
│           ↓                                                  │
│  scripts/fetch_trends.py                                     │
│           ↓ (pytrends API 호출)                              │
│  data/trends.json      ← 자동 갱신 (트렌드 데이터)              │
│           ↓                                                  │
│  trends-monitor.html   ← JSON fetch해서 카드 그리드 렌더링     │
│                                                              │
│  .github/workflows/update-trends.yml                         │
│           └─ 6시간마다 자동 실행 (GitHub Actions cron)         │
└─────────────────────────────────────────────────────────────┘
                          ↓
            https://username.github.io/repo/trends-monitor.html
                  (GitHub Pages 자동 반영, 수 분 내)
```

---

## 📁 최종 폴더 구조

```
ir-research/
├── index.html
├── cosmetics-monitor.html
├── trends-monitor.html              ← 신규 (트렌드 모니터링 UI)
├── data/
│   ├── keywords.json                ← 신규 (모니터링 키워드 마스터)
│   └── trends.json                  ← 자동 생성·갱신
├── scripts/
│   └── fetch_trends.py              ← 신규 (pytrends 수집 스크립트)
└── .github/
    └── workflows/
        └── update-trends.yml        ← 신규 (cron job 정의)
```

---

## 🚀 셋업 단계

### Step 1 · 파일 업로드

Repository에 다음과 같이 4개 파일을 올림 (GitHub Web UI 사용 가능):

| 로컬 파일 | 저장소 위치 |
|---------|-----------|
| `trends-monitor.html` | `/trends-monitor.html` |
| `keywords.json` | `/data/keywords.json` |
| `fetch_trends.py` | `/scripts/fetch_trends.py` |
| `update-trends.yml` | `/.github/workflows/update-trends.yml` |

폴더가 없으면 GitHub UI에서 **Add file → Create new file** → 파일명에 `data/keywords.json`처럼 `/`를 넣으면 자동 생성됨.

### Step 2 · GitHub Actions 활성화 확인

- 저장소 상단 **Actions** 탭 클릭
- "Update Google Trends" workflow가 보이면 정상
- 처음 실행은 다음 cron 시각까지 기다리거나, **Run workflow** 버튼으로 수동 실행 가능

### Step 3 · 첫 데이터 생성

방법 A — **수동 실행 (권장)**:
- Actions 탭 → "Update Google Trends" → **Run workflow** 클릭
- 약 2~5분 후 완료, `data/trends.json` 자동 commit 됨

방법 B — **로컬에서 직접 실행**:
```bash
pip install pytrends pandas
python scripts/fetch_trends.py
git add data/trends.json
git commit -m "initial trends data"
git push
```

### Step 4 · 라이브 확인

`https://<username>.github.io/<repo>/trends-monitor.html` 접속.

처음에는 fallback 샘플 데이터가 보이고, GitHub Actions 첫 실행 후엔 실제 pytrends 데이터로 교체됨.

---

## ✏️ 키워드 추가/수정하기

`data/keywords.json` 파일만 수정하면 됨.

### 새 키워드 추가
```json
{
  "categories": {
    "4연대 hobby": {
      "5대대 팬덤/수집품": [
        ...,
        {"keyword": "magic the gathering", "ticker": "HAS", "geo": "US"}
      ]
    }
  }
}
```

### 새 카테고리(연대) 추가
```json
{
  "categories": {
    "5연대 finance": {
      "1대대 핀테크": [
        {"keyword": "robinhood", "ticker": "HOOD", "geo": "US"}
      ]
    }
  }
}
```

수정 후 commit & push → 다음 cron 실행 시 자동 반영. 즉시 반영 원하면 Actions에서 Run workflow.

### 필드 설명
- `keyword`: Google Trends 검색어 (영문 권장, 한글도 가능)
- `ticker`: 표시용 티커 (선택, 빈 문자열 OK)
- `geo`: `"US"`, `"KR"`, `"JP"` 등 국가 코드. 빈 문자열이면 worldwide
- `geo`가 다른 키워드는 자동으로 다른 batch로 분리되어 fetch됨

---

## ⚙ 업데이트 주기 변경

`.github/workflows/update-trends.yml`의 cron 라인 수정:

| 표현 | 의미 |
|------|------|
| `"0 */6 * * *"` | 6시간마다 ✅ 기본값 |
| `"0 */3 * * *"` | 3시간마다 |
| `"0 * * * *"` | 매시간 |
| `"*/30 * * * *"` | 30분마다 ⚠️ pytrends 차단 위험 |
| `"0 9 * * *"` | 매일 09:00 UTC (= 18:00 KST) |
| `"0 0 * * 1"` | 매주 월요일 00:00 UTC |

⚠️ **GitHub Actions 무료 한도**:
- Public repo: 무제한
- Private repo: 월 2,000분. 6시간 cron(120회/월) × 5분 = 600분으로 여유 있음. 매시간 cron은 한도 초과 가능.

---

## 🛠 트러블슈팅

### pytrends 429 (Too Many Requests) 에러
- 가장 흔한 문제. Google이 자동 호출 차단함.
- **해결책 1**: `BATCH_SIZE`를 줄이고 `SLEEP_SECONDS`를 늘림 (`fetch_trends.py` 상단 상수)
- **해결책 2**: 키워드 수 자체를 줄임 (50개 이하 권장)
- **해결책 3 (가장 확실)**: SerpApi로 전환 (아래 참조)

### GitHub Actions 실패
- Actions 탭에서 실패한 run 클릭 → 로그 확인
- 흔한 원인: pytrends 차단 / 의존성 설치 실패 / 권한 부족
- 권한 부족이면 Settings → Actions → General → Workflow permissions → "Read and write" 활성화

### 카드가 모두 빈 sparkline으로 나옴
- `data/trends.json`이 비어있거나 잘못된 구조
- Actions 로그에서 실제로 데이터를 받았는지 확인
- 로컬에서 `python scripts/fetch_trends.py` 실행해 디버깅

### "trends.json 로드 실패" 메시지
- HTML이 fallback 샘플 데이터를 보여주는 중
- `data/trends.json` 파일이 실제로 commit 됐는지 확인
- 경로가 정확히 `data/trends.json`인지 확인 (대소문자 주의)

---

## 💎 더 안정적인 대안: SerpApi

pytrends가 자주 차단되면 SerpApi 유료 서비스가 가장 안정적.

### SerpApi 전환 (코드 수정)

`fetch_trends.py`의 fetch 부분만 교체:

```python
import requests

SERPAPI_KEY = os.environ.get("SERPAPI_KEY")  # GitHub Secrets에 저장

def fetch_serpapi(keyword: str, geo: str, timeframe: str = "today 12-m"):
    params = {
        "engine": "google_trends",
        "q": keyword,
        "data_type": "TIMESERIES",
        "geo": geo,
        "date": timeframe,
        "api_key": SERPAPI_KEY,
    }
    res = requests.get("https://serpapi.com/search.json", params=params)
    data = res.json()
    timeline = data.get("interest_over_time", {}).get("timeline_data", [])
    return [pt["values"][0]["extracted_value"] for pt in timeline]
```

### GitHub Secrets 등록
- Settings → Secrets and variables → Actions → New repository secret
- Name: `SERPAPI_KEY`, Value: SerpApi 키
- workflow yaml의 step에 `env: SERPAPI_KEY: ${{ secrets.SERPAPI_KEY }}` 추가

### 비용
- 무료: 100 requests/month → 키워드 50개면 한 달에 2회 fetch 가능
- $75/month: 5,000 requests → 6시간 cron으로 50개 키워드 운영 충분
- DataForSEO도 유사한 가격대 대안

---

## 📊 활용 팁

### 1. 즐겨찾기로 핵심 키워드 추적
- 카드 우상단 ♡ 클릭 → localStorage에 저장됨
- 정렬 옵션 ★ 클릭 → 즐겨찾기 우선 표시

### 2. 정렬 활용 시나리오
- **YoY↑**: 가장 큰 모멘텀 키워드 (alpha 시그널)
- **YoY↓**: 모멘텀 꺼지는 키워드 (short 후보)
- **값↑/값↓**: 절대 인기도 순

### 3. 카드 클릭 시 Google Trends 페이지로 이동
- 더 자세한 지역별·시기별 분석 필요할 때 활용

### 4. 카테고리 전략
- **연대 = 대분류 (food/fashion/house/hobby)**
- **대대 = 세부 카테고리 (피트니스/모빌리티/...)**
- 분류 체계는 진우님 자유롭게 변경 가능

---

## 🔮 향후 확장 아이디어

- [ ] WoW(Week-over-Week) 변화율도 카드에 표시
- [ ] 즐겨찾기 키워드는 별도 알림 (이메일·텔레그램)
- [ ] 임계치 초과 시 강조 (예: YoY > +200% 카드 골드 테두리)
- [ ] Notion DB에 주간 스냅샷 자동 push
- [ ] Korean stock ticker 매핑 시 한국 시간대 KOSPI 시세 함께 표시
- [ ] Google Trends 외에 Naver 검색 트렌드, Reddit mention 등 멀티 소스

필요한 확장 알려주시면 구현 도와드림.

---

*— Trend Monitor v1.0 · pytrends + GitHub Actions + GitHub Pages —*
