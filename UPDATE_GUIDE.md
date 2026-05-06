# K-Beauty Pulse — Weekly Update Manual

> 매주 월요일 30분 안에 끝나는 데이터 업데이트 루틴

---

## 🔁 주간 업데이트 흐름 (요약)

```
월요일 09:00 → 데이터 수집 (6개 소스, ~20분)
월요일 09:20 → cosmetics-monitor.html의 dashData 객체 수정 (~5분)
월요일 09:25 → git commit & push (~2분)
월요일 09:27 → GitHub Pages 자동 반영, 라이브 사이트 확인
```

수정 대상은 **단 한 곳**: HTML 파일 하단 `<script>` 안의 `const dashData = { ... }` 객체.

---

## 1️⃣ 한국 화장품 수출 데이터

### 데이터 소스
| 소스 | URL | 업데이트 주기 |
|------|-----|---------------|
| **관세청 수출입무역통계** | [tradedata.go.kr](https://tradedata.go.kr) | 매월 15일경 전월 잠정 |
| **한국무역협회 K-stat** | [kstat.kita.net](https://kstat.kita.net) | 관세청 데이터 가공 |
| **식약처 화장품 수출 동향** | mfds.go.kr 보도자료 | 분기 |

### 추출 방법
- 관세청 → "통계검색" → HS Code `33` 입력 → 월별·국가별 다운로드
- HS Code 분해:
  - **3304**: 미용·화장품 (주력 카테고리)
  - **3305**: 모발용 제품
  - **3303**: 향수
  - **3307**: 면도·체취 방지제

### HTML 수정 위치
```javascript
exports: {
  timeseries: {
    labels: [..., '26·05'],   // ← 새 월 추가
    total:  [..., 870]        // ← 해당 월 수출액(USD M) 추가
  },
  byCountry: [
    {name: '中國',  value: 892, share: 26.4, yoy: -8.2},  // ← 매월 갱신
    ...
  ]
}
```

### KPI 카드도 함께 수정
```javascript
// HTML 상단 KPI 영역 (id="kpi-export" 등) 직접 수정
// 또는 dashData.exports.timeseries.total의 마지막 값을 자동 반영하도록 확장 가능
```

---

## 2️⃣ 美 수입 시장 점유율

### 데이터 소스
| 소스 | URL | 비고 |
|------|-----|------|
| **USITC DataWeb** | [dataweb.usitc.gov](https://dataweb.usitc.gov) | 무료 가입, 가장 정확 |
| **US Census Trade** | [usatrade.census.gov](https://usatrade.census.gov) | 대안 |
| **Trading Economics** | tradingeconomics.com | 빠른 차트 확인용 |

### 추출 방법
- USITC DataWeb → Import Statistics → HS 4-digit `3304` (Beauty/skincare) 입력
- Country: Korea, France, Japan, China, Canada 다중 선택
- Period: Quarterly, 최근 8분기
- Value 또는 Customs Value 선택

### HTML 수정 위치
```javascript
usImports: {
  timeseries: {
    labels: [..., '26·Q1'],
    Korea:  [..., 22.4],   // ← 신규 분기 점유율 추가
    France: [..., 18.1],
    Japan:  [..., 9.9],
    Canada: [..., 10.0],
    China:  [..., 11.4]
  },
  hsBreakdown: [...]   // 분기 단위 갱신
}
```

---

## 3️⃣ 구글 트렌드

### 데이터 소스
| 도구 | URL | 비고 |
|------|-----|------|
| **Google Trends** | [trends.google.com](https://trends.google.com) | Worldwide 또는 US 필터 |
| **Glimpse** (확장프로그램) | glimpse.com | 절대치 추정 |
| **pytrends** (Python) | github.com/GeneralMills/pytrends | 자동화 |

### 추출 방법
- Google Trends → "여러 검색어 비교" → 최대 5개 브랜드 입력
- Region: Worldwide 또는 United States
- Time: Past 12 months
- 우측 상단 다운로드 버튼 → CSV
- "급상승 검색어" 탭에서 Rising 키워드 캡처

### 권장 모니터링 브랜드 (5개씩 그룹)
- **Group A (성장 모멘텀)**: medicube · Anua · TIRTIR · Numbuzin · Mixsoon
- **Group B (성숙 브랜드)**: Cosrx · Beauty of Joseon · Laneige · Mediheal · Innisfree
- **Group C (한국 럭셔리)**: Sulwhasoo · Whoo · The History of Whoo · O HUI · Hera

### HTML 수정 위치
```javascript
googleTrends: {
  timeseries: {
    labels: [..., '26·19'],
    brands: {
      'medicube':         [..., 91],   // ← 새 주차 인덱스 추가
      'Anua':             [..., 76],
      ...
    }
  },
  trending: [
    {keyword: '...', growth: '+340%'}, ...   // ← 매주 교체
  ]
}
```

---

## 4️⃣ 아마존 美 — 한국 브랜드 BSR

### 데이터 소스
| 도구 | 가격 | 비고 |
|------|------|------|
| **Helium 10** | 유료 ($79/월~) | BSR 추적, 가장 정확 |
| **Jungle Scout** | 유료 ($49/월~) | 카테고리 분석 강점 |
| **Keepa** | 무료/유료 | BSR 히스토리 차트 |
| **SellerSprite** | 유료 | 중국계, 가성비 |
| **Amazon BSR 직접** | 무료 | amazon.com → Best Sellers → Beauty |

### 무료 옵션 (수기)
- amazon.com/bestsellers/beauty 페이지에서 한국 브랜드 검색·수기 기록
- 각 브랜드 대표 SKU 페이지 → "Best Sellers Rank" 확인
- 매주 같은 SKU의 BSR을 기록해 주간 변화 산출

### 모니터링 SKU 리스트 (12개)
| Brand | Hero SKU | URL 패턴 |
|-------|----------|----------|
| Anua | Heartleaf 77% Toner 250ml | /dp/B0CPDF... |
| Beauty of Joseon | Glow Sunscreen SPF50+ | /dp/B0BS78... |
| Cosrx | Snail 96 Mucin Power Essence | /dp/B00PBX... |
| medicube | Zero Pore Pad 2.0 | /dp/B0BTYL... |
| Mediheal | Tea Tree Mask Pack 24ea | /dp/B0BBWG... |
| ... | ... | ... |

### HTML 수정 위치
```javascript
amazon: {
  bsrTable: [
    {brand: 'Anua', sku: 'Heartleaf 77% Toner', bsr: 42, wow: 'up', delta: '▲ 8'},
    ...   // ← 12개 브랜드 BSR과 WoW 수정
  ],
  categoryLeaders: [
    {cat: 'Toner', leader: 'Anua Heartleaf', days: 42},
    ...
  ]
}
```
- `wow` 값: `'up'` / `'down'` / `'flat'` / `'new'` 중 선택 (색상·뱃지 자동 적용)

---

## 5️⃣ TikTok Shop 美 GMV 랭킹

### 데이터 소스
| 도구 | 가격 | 비고 |
|------|------|------|
| **FastMoss** | 무료 + 유료 | TikTok Shop 분석 표준 |
| **Kalodata** | 유료 | 더 깊은 데이터 |
| **Shoplus** | 무료 + 유료 | 대안 |
| **TikTok Creative Center** | 무료 | 트렌드만 |

### 추출 방법
- FastMoss → "Product Rankings" → Beauty & Personal Care
- Country: United States
- Time Range: Last 30 days / Last 7 days
- 한국 브랜드 필터링 (제조국 또는 브랜드명 검색)
- Top 15 GMV 기록

### HTML 수정 위치
```javascript
tiktok: {
  top15: {
    labels: ['medicube','TIRTIR','Anua', ...],   // ← 순위 순서대로
    gmv: [31.2, 18.5, 14.8, ...]                  // ← USD M 단위
  },
  movers: [
    {brand: 'TIRTIR', gmv7d: '$4.8M', wow: 'up', delta: '+47%'},
    ...
  ]
}
```

---

## 6️⃣ 큐텐 메가와리 — 일본

### 데이터 소스
| 소스 | URL | 비고 |
|------|-----|------|
| **Qoo10 Japan 메가와리** | [qoo10.jp/megawari](https://www.qoo10.jp) | 행사 기간 직접 확인 |
| **Qoo10 K-Beauty 카테고리 랭킹** | qoo10.jp/gmkt.inc | 상시 |
| **메가와리 결산 보도자료** | 큐텐 재팬 공식 PR | 행사 종료 후 |

### 메가와리 행사 일정 (참고)
- 메가와리는 분기별 약 1회씩 진행 (3·6·9·11월 경)
- 행사 기간: 약 6일
- 행사 종료 후 1~2주 내 랭킹 데이터 확정

### 추출 방법
- 메가와리 행사 페이지 → "ランキング" 탭
- 카테고리: 美容(뷰티) / スキンケア / メイクアップ
- Top 30 중 한국 브랜드 필터링
- 판매 지수는 1위 = 100 기준 정규화

### HTML 수정 위치
```javascript
qoo10: {
  rankings: [
    {brand: 'TIRTIR', hero: 'Mask Fit Cushion', index: 100, prev: 'up', delta: '▲ 1'},
    ...
  ],
  shareTrend: {
    labels: [..., '26·Q1'],
    Korea: [..., 61], Japan: [..., 25], Other: [..., 14]
  }
}
```

---

## 🚀 GitHub Pages 배포

저장소에 파일이 이미 push 된 상태라면:

```bash
# 로컬 작업 시
git add cosmetics-monitor.html
git commit -m "weekly update: W19 2026"
git push
```

또는 GitHub 웹 UI에서 직접 파일 편집 → commit changes.

배포 URL: `https://<username>.github.io/<repo>/cosmetics-monitor.html`

인덱스 페이지(`index.html`)에 링크 카드 추가하면 메인에서 진입 가능.

---

## 💡 권장 자동화 단계

처음 1~2개월은 수기 입력으로 운영 패턴 익힌 후, 점진적으로 자동화:

1. **1단계 (자동화 가능 즉시)**: 구글 트렌드 → pytrends Python 스크립트로 매주 CSV 자동 추출
2. **2단계**: 관세청 데이터 → API 또는 정기 다운로드 후 Python으로 JSON 변환
3. **3단계**: TikTok Shop · Amazon → FastMoss·Helium 10 API 활용 (유료 플랜 필요)
4. **4단계**: GitHub Actions cron job으로 매주 월요일 자동 실행 → JSON 빌드 → push

자동화 스크립트가 필요하시면 별도 요청 주세요.

---

## 📌 데이터 입력 시 체크리스트

- [ ] 모든 숫자가 같은 단위(USD M, %, count 등)로 통일됨
- [ ] `wow` 값은 `'up'`/`'down'`/`'flat'`/`'new'` 중 하나
- [ ] `delta` 문자열에 화살표(▲/▼/—) 포함
- [ ] YoY/WoW 부호(`+`/`-`) 명확
- [ ] 신규 진입 브랜드는 `wow: 'new'` 설정 → 자동으로 NEW 뱃지 표시
- [ ] 차트 라벨 길이가 너무 길면 화면 깨짐 (8자 이내 권장)

---

*— K-Beauty Pulse · Maintained by Jinwoo —*
