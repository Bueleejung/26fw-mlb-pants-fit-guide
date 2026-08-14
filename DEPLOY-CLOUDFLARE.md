# Cloudflare Pages 배포 가이드 — 26fw-mlb-pants-fit-guide (26FW 단독)

기존 GitHub Pages 배포( https://bueleejung.github.io/26fw-mlb-pants-fit-guide/ )는 **그대로 유지**된다.
같은 저장소를 Cloudflare Pages 에도 연결해 이중 배포하는 방법을 정리한다.

| 항목 | 값 |
| --- | --- |
| GitHub 저장소 | `Bueleejung/26fw-mlb-pants-fit-guide` |
| 배포 브랜치 | `main` |
| 빌드 도구 | 없음 (순수 정적 파일) |
| GitHub Pages URL | https://bueleejung.github.io/26fw-mlb-pants-fit-guide/ |
| Cloudflare 예상 URL | https://26fw-mlb-pants-fit-guide.pages.dev/ |

---

## 왜 코드 수정이 필요 없나

- GitHub Pages 는 `/26fw-mlb-pants-fit-guide/` 하위 경로로, Cloudflare Pages 는 루트 `/` 로 서빙한다.
- `index.html`, `sw.js`, `manifest.json` 의 모든 참조가 **상대경로**(`./sw.js`, `./manifest.json`, `./wide-uni.mp4` 등)라서 두 경로 구조 모두에서 동일하게 동작한다.
- 서비스 워커도 `navigator.serviceWorker.register('./sw.js')` 로 등록되어 스코프가 자동으로 맞춰진다.

즉 **저장소를 그대로 연결하기만 하면 된다.**

---

## ⚠ 로컬 작업 폴더에 대한 안내

OneDrive 의 `26FW/TCL 태블릿/fit-guide-deploy-update` 폴더는 이 저장소의 작업본이 **아니다.**
그 폴더의 git 이력(`e39a116`)은 GitHub 저장소 이력(`141ee9ac` — 웹 UI 업로드로 생성)과 무관한 별개 이력이고 remote 도 없다. **그 폴더에서 `git push` 하면 강제 푸시가 되므로 하지 말 것.**

단, 파일 내용 자체는 검증 결과 **원격과 100% 동일**하다 (`index.html`, `sw.js`, `manifest.json`, mp4 6개 전부 git blob 해시 일치). 눈에 보이는 바이트 크기 차이는 체크아웃 시 CRLF 변환 때문이며 실제 내용 차이가 아니다. 즉 동기화할 콘텐츠는 없다.

앞으로 이 저장소를 수정할 때는 깨끗한 작업본을 새로 받아서 쓴다.

```powershell
git clone https://github.com/Bueleejung/26fw-mlb-pants-fit-guide.git
```

### Jekyll 관련 참고

이 저장소에는 `.nojekyll` 이 없어 GitHub Pages 가 Jekyll 빌드로 동작한다. Jekyll 은 밑줄로 시작하는 `_headers` 를 산출물에서 제외하므로, `_headers` 는 **github.io 사이트에는 노출되지 않는다.** Cloudflare Pages 는 GitHub Pages 산출물이 아니라 git 저장소를 직접 읽으므로 정상적으로 적용된다. 양쪽 모두 문제 없다.

---

## 방법 A — Git 연동 (권장)

`main` 에 push 할 때마다 GitHub Pages 와 Cloudflare Pages 가 **동시에** 갱신된다.

1. Cloudflare 대시보드 → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. GitHub 계정 인증 후 `Bueleejung/26fw-mlb-pants-fit-guide` 저장소 선택
   (Cloudflare GitHub App 설치 시 해당 저장소 접근 권한 허용)
3. 빌드 설정 — **반드시 아래대로 비워두거나 지정**

   | 설정 | 값 | 근거 |
   | --- | --- | --- |
   | Project name | `26fw-mlb-pants-fit-guide` | pages.dev 호스트명이 됨 |
   | Production branch | `main` | — |
   | Framework preset | `None` | 프리셋 표에 정적 HTML 행이 없음 |
   | Build command | `exit 0` | 공식 정적 사이트 가이드 지정값 |
   | Build output directory | `/` | `index.html` 이 repo 루트에 있음 |
   | Root directory | *(건드리지 않음)* | 미지정 시 repo 루트로 간주됨 |

   - **Build command 를 `exit 0` 으로 두는 이유** — Pages 는 빌드 명령의 종료 코드로 성공/실패를 판정하며(0 이 아니면 실패), `exit 0` 을 넣어야 Pages Functions 같은 기능 경로가 열린다. 공백으로 둬도 배포 자체는 된다.
   - **Root directory 는 monorepo 전용 설정**이다. 공식 문서상 미지정이면 연결된 저장소 루트를 그대로 사이트 루트로 본다. 굳이 값을 넣지 않는다.
   - **Build output directory 주의** — 공식 문서는 이 값을 `<YOUR_BUILD_DIR>`("콘텐츠가 실제로 있는 위치")로만 정의하고 루트를 뜻하는 리터럴 값을 명시하지 않는다. 이 프로젝트는 `index.html` 이 저장소 루트에 있으므로 대시보드 기본값인 `/` 를 그대로 둔다.

4. **Save and Deploy** → 1~2분 후 `https://26fw-mlb-pants-fit-guide.pages.dev/` 에서 확인

## 방법 B — Wrangler 직접 업로드

Git 연동 없이 로컬 폴더를 바로 올린다. (Node.js 필요)

```powershell
# 이 폴더(fit-guide-deploy-update)에서 실행
npx wrangler@latest login
npx wrangler@latest pages project create 26fw-mlb-pants-fit-guide --production-branch=main
npx wrangler@latest pages deploy . --project-name=26fw-mlb-pants-fit-guide --branch=main --commit-dirty=true
```

`.git/`, `.claude/`, `node_modules/`, `_headers` 는 업로드 대상에서 자동 제외된다.

---

## 배포 후 점검

1. `https://26fw-mlb-pants-fit-guide.pages.dev/` 접속 → 6개 핏 전환 확인
2. DevTools → Application → Service Workers 에 `sw.js` 가 activated 인지 확인
3. Network 탭에서 `.mp4` 요청이 **206 Partial Content** 로 응답하는지 확인 (Range 요청 정상)
4. `curl -I https://26fw-mlb-pants-fit-guide.pages.dev/sw.js` → `cache-control: no-cache` 확인
5. 태블릿 실기기에서 오프라인 재생(비행기 모드) 확인

## 제한사항 확인 (현재 프로젝트는 모두 통과)

| Cloudflare Pages 제한 | 현재 값 | 판정 |
| --- | --- | --- |
| 단일 파일 최대 25 MiB | 최대 `wide-uni.mp4` 15.6 MiB | OK |
| 배포당 최대 20,000 파일 (Free) | 9개 | OK |
| 빌드 타임아웃 20분 | 빌드 없음 | OK |
| 월 500회 빌드 (Free) | — | OK |

> 향후 영상 파일이 25 MiB 를 넘으면 Cloudflare Pages 업로드가 실패한다.
> 그 경우 인코딩 비트레이트를 낮추거나 해당 영상만 R2 로 분리해야 한다.

---

## 운영 시 주의

- **두 도메인은 서비스 워커 캐시가 서로 독립적이다.** 태블릿을 github.io 에서 pages.dev 로 옮기면 영상을 처음부터 다시 받는다. 전환은 Wi-Fi 환경에서 진행할 것.
- 영상을 같은 파일명으로 교체할 때는 기존 절차대로 `sw.js` 의 `CACHE_NAME` 값을 올려야 (`fit-guide-v8` → `v9`) 태블릿이 새 영상을 받는다. 이 규칙은 Cloudflare 에서도 동일하다.
- 커스텀 도메인을 붙이려면 Pages 프로젝트 → **Custom domains** 에서 추가한다. 도메인이 Cloudflare DNS 에 있어야 한다.
