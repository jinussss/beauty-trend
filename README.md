# IR Research Notes

> 국내·해외 주식, 바이오·테크 리서치 노트 아카이브

[![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-Live-success)](https://USERNAME.github.io/REPO-NAME/)
[![Updated](https://img.shields.io/badge/Updated-2026--05-blue)](#)

---

## 🌐 Live Site

👉 **https://USERNAME.github.io/REPO-NAME/**

> ⚠️ 위 URL의 `USERNAME`과 `REPO-NAME`을 본인 계정·저장소명으로 교체하세요.

---

## 📁 폴더 구조

```
.
├── index.html              # 메인 인덱스 페이지
├── README.md               # 이 파일
├── reports/                # 리서치 리포트 HTML
│   ├── domestic/           # 국내 기업
│   ├── overseas/           # 해외 기업
│   ├── biotech/            # 바이오·제약 테마
│   └── tech/               # 테크·반도체 테마
├── widgets/                # 인터랙티브 위젯
│   ├── value-chain/
│   └── earnings-comparison/
└── assets/                 # 공통 리소스
    ├── css/
    ├── js/
    └── images/
```

---

## ✏️ 새 리포트 추가하는 법

1. 해당 카테고리 폴더(`reports/domestic/` 등)에 새 HTML 파일 저장
2. `index.html`을 열어 해당 카테고리 섹션의 `.grid` 안에 새 `<article class="card">` 추가
3. `git add . && git commit -m "add: [기업명] 리포트" && git push`
4. 1~2분 후 GitHub Pages에 자동 반영됨

---

## 🛠️ 빌드 정보

- 정적 사이트, 빌드 도구 없음 (vanilla HTML/CSS/JS)
- 폰트: Fraunces (display) · Pretendard (한글) · JetBrains Mono (mono) — Google Fonts CDN
- 의존성: 없음
- 호스팅: GitHub Pages

---

## 📜 License

본 저장소는 개인 리서치 아카이브임. 외부 인용 시 출처 표기 부탁드림.

---

*Crafted with research, conviction, and Claude.*
