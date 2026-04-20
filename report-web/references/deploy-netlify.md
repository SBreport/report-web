# Netlify Drop 배포 가이드

Phase 6에서 사용자가 배포를 원할 때 적용하는 체크리스트 + 절차.

## 🚨 배포 전 필수 체크 — 민감 정보

**공개 URL에 올라가면 안 되는 정보**가 콘텐츠에 포함됐는지 점검한다.

### 스캔 대상
- 사무실·회의실 출입문 비밀번호
- 와이파이 SSID·비밀번호
- 공용 계정 ID/PW
- 고객사 실명 (NDA 위반 가능)
- 개인 연락처·이메일
- 내부 시스템 URL (노션 등) 중 공개 불가 링크
- API 키·토큰

### 발견 시 처리 방향 (사용자에게 선택 요청)

| 처리 | 언제 |
|---|---|
| A. 해당 섹션 제거 후 "내부 노션에서 확인" 링크 대체 | 공개 배포 원할 때 |
| B. 비밀번호 보호 호스팅 (Cloudflare Access 등) | 내부 직원만 접근 |
| C. 배포 안 함, 로컬 사용만 | 최고 보안 필요 |

A 선택 시 `notion-link-btn` 컴포넌트로 대체.

### 검증 커맨드

빌드 후 `index.html`에서 민감 패턴이 사라졌는지 grep으로 확인:
```bash
grep -c "비밀번호패턴\|와이파이값" index.html
```
0이어야 통과.

## Netlify Drop 절차

### 준비 — 불필요 파일 제외

배포 시 아래 파일은 **사이트 렌더링에 불필요**하므로 업로드 안 하는 게 좋다 (추가 오버헤드·혼란 방지):

- `src/` — 빌드 소스, 결과물만 필요
- `sections/` — 빌드에 사용, 결과는 index.html에 포함됨
- `build.bat`, `build.sh` — 빌드 스크립트
- `README.md` — 개발 문서
- `*.bak` — 백업 파일

배포용 폴더를 복사해서 위 파일·폴더를 지우거나, 필요 최소만 새 폴더로 복사:
```
deploy_tmp/
├── index.html
├── css/
│   └── styles.css
└── js/
    └── app.js
```

### 배포

1. 브라우저에서 https://app.netlify.com/drop 접속
2. 폴더를 드래그 앤 드롭 영역에 끌어다 놓기
3. 10~30초 내 URL 발급 (형식: `https://<random-name>.netlify.app`)
4. 즉시 HTTPS 자동 적용

### 계정 연결 (선택)

"Claim this site" 클릭 → 이메일 가입 → 사이트 소유권 귀속.

계정 연결 시 가능:
- URL 커스터마이즈 (예: `smartbranding-onboarding.netlify.app`)
- 커스텀 도메인 연결
- 재배포 (재업로드)
- 사이트 삭제

## 업데이트 배포

내용 수정 후:
1. sections 편집
2. `build.sh` 또는 `build.bat`으로 index.html 재생성
3. 배포용 폴더 재준비
4. Netlify 대시보드 → 해당 사이트 → Deploys 탭 → Drag-and-drop 영역에 새 폴더 드롭

## 대안 플랫폼 (참고)

Netlify Drop이 사용자 상황에 안 맞을 때:

| 플랫폼 | 장점 | 단점 |
|---|---|---|
| Vercel | Netlify와 유사, CLI 편함 | 가입 필요 |
| Surge.sh | `npm i -g surge && surge` 한 줄 | CLI 설치 필요 |
| GitHub Pages | 버전 관리 병행 | Public repo = 소스 공개 |
| Cloudflare Pages | 무료 비밀번호 보호 가능 (Zero Trust) | 셋업 복잡 |

민감 정보가 있는데 꼭 웹으로 공유해야 하면 **Cloudflare Pages + Access** 추천.

## 배포 완료 후 사용자 안내

- 발급받은 URL 공유
- 업데이트 방법 설명 (재빌드 + 재드롭)
- 사이트 관리는 Netlify 계정에 연결 권유
- 민감 정보 제거·우회 처리 내역 요약 (A 선택한 경우)
