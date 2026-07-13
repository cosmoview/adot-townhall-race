# A.Biz 임원 달리기 — 프로젝트 핸드오프 (새 대화 이어가기용)

> 이 문서를 새 대화 맨 처음에 붙여넣으면 이어서 작업할 수 있습니다.
> 나(사용자)는 SKT 에이닷 마케팅팀이고, 사내 타운홀용 **실시간 투표 + 픽셀 캐릭터 레이스 웹앱**을 만들고 있습니다.
> 진행 방식: 제작 전 텍스트로 기획/구조 먼저 제안 → 선택지(A/B/C)+추천 제시 → 내가 고르면 제작. 낯선 툴은 메뉴/단계 단위 안내. 한국어로. '대리' 등 직급 호칭 안 씀.

---

## 1. 개요
- 타운홀에서 참가자(최대 300명)가 폰으로 투표 → 대형 화면에서 5명 임원 픽셀 캐릭터가 득표수만큼 달림.
- **5라운드**, 라운드당 **20초** 투표. 라운드 우승자에게 트로피, 최다 트로피 = 시즌 챔피언.
- 백엔드: **Firebase Realtime Database**(Spark 무료, 싱가포르 리전). 프론트: 단일 HTML 3개 파일, GitHub Pages 배포.

## 2. 파일 구성 (모두 단일 HTML, `/mnt/user-data/outputs/` 및 로컬)
- **screen.html** — 대형 화면(레이스). **이 화면 하나로 전체 진행 가능**(숨은 호스트 컨트롤: 마우스 움직이면 하단 바 등장/3.5초 후 숨김, 키보드 `Space`=진행(상황별)/`R`=리셋). ~452KB.
- **vote.html** — 폰 투표 화면.
- **control.html** — 호스트 원격 컨트롤(선택). 접속키: `?key=adot2026`. screen과 동시에 열어도 중복 채점 방지됨.
- Firebase compat SDK 9.23.0(CDN), 폰트 Pretendard + Press Start 2P.

## 3. Firebase 설정 (project: `adot-townhall-race`, Spark 무료)
```js
const firebaseConfig={
  apiKey:"AIzaSyAwqdsX9-HYr4xcniuD97dY2XIjw1dQ3fk",
  authDomain:"adot-townhall-race.firebaseapp.com",
  databaseURL:"https://adot-townhall-race-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId:"adot-townhall-race",
  storageBucket:"adot-townhall-race.firebasestorage.app",
  messagingSenderId:"699342448682",
  appId:"1:699342448682:web:4336db8103ffe6e7c13b53"
};
```
- ⚠️ 보안 규칙: 현재 **테스트 모드(누구나 읽기/쓰기)**. 실전 전 조여야 함(아래 TODO).

## 4. 데이터 구조 (Realtime DB)
```
control/state   = { phase, roundId(1~5), endTs(ms), remaining(ms, 일시정지용) }
                  phase: 'idle'|'open'|'paused'|'closed'|'finale'
control/session = 타임스탬프 (전체 리셋 시 갱신 → 폰들이 로컬 잠금 해제)
rounds/{n}/votes/{clientId} = teamKey   ("green"|"red"|"yellow"|"blue"|"black")  ← 1인1표(키로 보장)
rounds/{n}/result = winnerKey           (마감 시 기록, 중복 채점 방지 가드)
trophies = { green:n, red:n, yellow:n, blue:n, black:n }
presence/{clientId} = true              (접속 표시; onDisconnect로 자동 제거)
```

## 5. 임원 ↔ 색 매핑 (얼굴 확인 완료, 절대 바꾸지 말 것)
| 색 | 이름 | 직책 | 레인(위→아래) | bottom% |
|---|---|---|---|---|
| 🟩 green | 김지훈 | AI사업본부장 | 1 | 46.5 |
| 🟥 red | 장성국 | AI서비스개발담당 | 2 | 35.0 |
| 🟨 yellow | 이정룡 | Agent개발담당 | 3 | 23.5 |
| 🟦 blue | 윤현상 | 에이닷기획담당 | 4 | 12.0 |
| ⬛ black | 조현덕 | 에이닷전화담당 | 5 | 3.0 |
- 색상값: green `#2E9E4F`, red `#D93A3A`, yellow `#E0A008`, blue `#2E74D0`, black `#3A3A3A`.

## 6. 질문 5개 (현재값)
1. 신입사원 시절 가장 늦게까지 야근 했을 거 같은 임원은?
2. 회식에서 끝까지 자리를 지킬거 같은 임원은?
3. 길에서 우연히 만났는데 커피 사줄 거 같은 임원은?
4. 갑자기 무인도에 떨어져도 살아남을 거 같은 임원은?
5. 유튜버로 데뷔한다면 가장 성공할거 같은 임원은?
- 질문은 `screen.html`·`vote.html` **양쪽**의 `const QUESTIONS=[...]` 에 동일하게 들어감(둘 다 수정 필요).

## 7. 배포
- GitHub repo: **`cosmoview/adot-townhall-race`** (Public), GitHub Pages 켜짐(main / root).
- 로컬 폴더: `C:\Users\SKTelecom\Desktop\AI Work\townhall_race`
- 배포 명령(VS Code 터미널):
  ```bash
  git add .
  git commit -m "메시지"
  git push
  ```
- 접속 URL:
  - 대형 화면: `https://cosmoview.github.io/adot-townhall-race/screen.html`
  - 폰 투표: `https://cosmoview.github.io/adot-townhall-race/vote.html`
  - 호스트: `https://cosmoview.github.io/adot-townhall-race/control.html?key=adot2026`
- ⚠️ **캐시 주의**: 배포 후 화면이 안 바뀌면 URL 뒤에 `?v=2`(숫자 증가) 붙이거나 `Ctrl+Shift+R`/시크릿창.

## 8. 동작 규칙 (구현 완료)
- **진행 흐름**: idle(대기) → [ROUND n 시작] → open(20초 투표) → 자동/수동 마감 → closed(우승 배너) → [다음 라운드] … R5 후 [🏆 최종 결과]=finale.
- **1인 1표 + 재투표**: `votes/{clientId}` 키라 덮어쓰기. open 동안 폰에서 **카드 유지형으로 자유롭게 변경 가능**(마지막 선택만 집계, 표 안 늘어남). 마감되면 잠김.
- **자동 마감**: `endTs` 도달 시. 우승 = 최다 득표(동점 시 목록 앞=green 방향 우선). 우승 시 `trophies` +1 (트랜잭션 + `result` 가드로 중복 채점 방지, screen·control 동시에 열어도 안전).
- **최종 결과(finale)**: 트로피 최다 = 챔피언. 동점이면 **공동 우승**, 5명 전부 동점이면 **"전원 우승!"** 표시.
- **일시정지(B안)**: `[⏸ 일시정지]` → 타이머 그 자리에서 멈춤 + **폰은 "잠시 멈춤" 화면으로 투표 잠금** + 캐릭터 정지 + 자동마감 안 됨. `[▶ 재개]` → 남은 시간부터 이어짐, 폰 다시 열림.
- **전체 리셋**: 표·트로피·상태 초기화 + `control/session` 갱신 → 모든 폰이 로컬 잠금(voted_/team_) 해제하고 재투표 가능.
- **대형 화면 요소**: 상단 중앙 **대형 타이머**(open일 때만 표시, 5초 이하 빨강 깜빡임), 좌상단 질문(폭 제한), **우상단 👥접속 인원 / 🗳️투표 인원**(presence 기반/이번 라운드 기준). 트로피 배지는 화면에서 제거(집계는 내부 유지).
- **캐릭터**: 새 캐리커처 5종(1포즈), **오른쪽(결승선) 향해** 달림. open 동안 CSS **바운스 모션**(상하+살짝 기울임). 진짜 다리 사이클은 없음(1포즈 한계).
- **폰 대기화면(A안)**: "✅ 참여 준비 완료!" + 깜빡이는 점 + 임원 5색 미리보기(이름 숨김) + "질문 뜨면 한 명 골라 투표 · 총 5라운드".

## 9. 남은 작업 (TODO)
1. **[중요] 300명 동시 접속 대비** — Spark 무료는 **동시 연결 100개 한도**(폰마다 연결 1개 유지 = 접속인원). 300명이면 초과분 거부됨.
   - 해결 A(추천): **Blaze 요금제 전환** + 예산 알림. 연결 자체는 과금 0, 데이터도 무료범위라 이 행사엔 사실상 0원. 코드 변경 없음. (카드 등록 필요)
   - 해결 B: Spark 유지 + 폰을 **무연결화**(실시간 구독 대신 REST 쓰기 + 폴링). 코드 재작성 필요.
2. **Firebase 보안 규칙 조이기** — 현재 테스트 모드. 실전 전 "라운드 열림일 때만 쓰기" 등으로 제한.
3. **부하 테스트** — 가짜 clientId 다수 접속/투표 시뮬레이션 스크립트로 300 연결 검증.
4. **현장 리허설** — 실제 폰 다수(WiFi+LTE) 동시 접속, 프로젝터 스케일/해상도, Firebase 도달성 점검.

## 10. 주의사항 / 교훈
- **캐릭터 방향**: 원본 이미지 방향 판단이 여러 번 헷갈렸음. 최종 확정 = **오른쪽(결승선/체크무늬 깃발) 향함**. "안 바뀐다"는 대부분 **캐시**(배포 미반영) 문제 → `?v=` / 시크릿창 / 강력새로고침으로 확인.
- 캐릭터 프레임은 base64로 HTML에 내장. 새로 만들려면 배경(마젠타 권장) 제거 + 가장 큰 덩어리만 남기기 + 우향 정렬 + 공통 캔버스 정규화 파이프라인 사용.
- **주관식 의견 기능은 추가했다가 취소(제거)함** — 현재 없음.
- 파일 3개는 서로 `control/state`, `rounds/*`, `trophies`, `presence`를 공유. 질문/임원/색은 세 파일에 동일하게 유지.

## 11. 새 대화에서 이어서 할 때 (요청 예시)
- "위 핸드오프 기준으로 이어가자. 먼저 **Blaze 전환**을 콘솔 메뉴 단위로 안내해줘."
- 또는 "**Firebase 보안 규칙**을 라운드 열림일 때만 쓰기로 조이는 규칙 만들어줘."
- 또는 "**300 연결 부하 테스트** 스크립트 만들어줘."
- 파일을 계속 수정하려면, 현재 `screen.html`/`vote.html`/`control.html`을 **업로드**해서 주면 그 파일 기준으로 편집함(캐릭터 방향은 화면에서 본 그대로 신뢰).
