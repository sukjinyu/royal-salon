# Cloudflare 무료 배포 가이드

이 게임은 Cloudflare Pages에 무료로 배포할 수 있습니다. 화면은 Pages가 제공하고, 별명별 저장/순위/상점 저장은 Cloudflare D1 데이터베이스가 맡습니다.

## 0. 먼저 알아둘 것

로컬 개발에서는 Express와 `data/salon.sqlite`를 사용합니다. Cloudflare에 올릴 때는 Express 서버를 직접 올리지 않고, `functions/` 폴더의 Pages Functions가 `/api/save`, `/api/players`, `/api/rankings` 같은 API를 대신 처리합니다.

Cloudflare D1은 SQLite 방식의 서버리스 데이터베이스이고 Free 플랜에서 사용할 수 있습니다. 단, 무료 사용량 제한을 넘기면 과금 설정이 필요할 수 있으니 개인 테스트 규모로 쓰는 것을 권장합니다.

## 1. Cloudflare 계정 만들기

1. https://dash.cloudflare.com 에 접속합니다.
2. 무료 계정으로 가입합니다.
3. 카드 등록을 요구하지 않는 범위에서 Pages와 D1만 사용합니다.

## 2. 터미널에서 로그인하기

프로젝트 폴더에서 아래 명령을 실행합니다.

```powershell
cd C:\Users\admin\Documents\Codex\2026-09-11\new-chat-3\outputs\royal-salon
npx wrangler login
```

브라우저가 열리면 Cloudflare 로그인을 허용합니다.

## 3. D1 데이터베이스 만들기

```powershell
npx wrangler d1 create royal-salon-db
```

명령 결과에 `database_id`가 나옵니다. 예시는 이런 모양입니다.

```toml
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

그 값을 `wrangler.toml` 파일의 아래 줄에 붙여넣습니다.

```toml
database_id = "여기에-d1-database-id-붙여넣기"
```

## 4. D1 테이블 만들기

```powershell
npx wrangler d1 execute royal-salon-db --remote --file migrations/0001_player_saves.sql
```

이 단계가 끝나면 별명별 저장, 리셋, 순위가 Cloudflare에서도 동작합니다.

## 5. 빌드하기

```powershell
npm run build
```

성공하면 `dist/` 폴더가 만들어집니다.

## 6. Cloudflare Pages에 배포하기

GitHub 저장소를 Cloudflare Pages에 연결했다면 Cloudflare 대시보드에서 다음처럼 설정합니다.

- Build command: `pnpm run build`
- Build output directory: `dist`
- Deploy command: 비워 두기

Pages는 빌드가 끝난 뒤 `dist/`를 자동으로 배포합니다. GitHub에 새 커밋을 올리면 자동으로 다시 배포됩니다.

터미널에서 수동 배포할 때만 아래 명령을 사용합니다. `royal-salon` 부분은 Cloudflare Pages에 표시되는 실제 프로젝트 이름으로 바꾸세요.

```powershell
npx wrangler pages deploy dist --project-name royal-salon
```

## 7. 배포 후 확인하기

브라우저에서 배포 주소를 열고 아래를 확인합니다.

1. 게임 첫 화면이 뜨는지 확인합니다.
2. 별명을 입력하고 저장 버튼을 누릅니다.
3. 순위 탭에서 별명이 보이는지 확인합니다.
4. 상점에서 구매한 장식이 저장되는지 확인합니다.

## 자주 막히는 부분

### `Missing entry-point to Worker script or to assets directory` 오류가 나요

Cloudflare가 Pages 배포 명령이 아니라 Workers 배포 명령을 실행한 상황입니다.

Cloudflare Pages 배포 설정에서 Deploy command가 아래처럼 되어 있으면 실패합니다.

```powershell
npx wrangler deploy
```

GitHub 자동 배포를 계속 사용할 경우에는 Deploy command를 통째로 지우는 것이 가장 간단합니다.

수동 배포 명령을 입력해야 하는 화면이라면 아래처럼 `pages`를 포함한 명령으로 바꾸세요.

```powershell
npx wrangler pages deploy dist --project-name royal-salon
```

Cloudflare Pages 화면에서 프로젝트 이름이 `ssalon1`처럼 다르게 보이면 마지막 이름만 실제 프로젝트 이름으로 바꾸면 됩니다. GitHub 저장소 이름과 Pages 프로젝트 이름은 서로 다를 수 있습니다.

### `database_id` 오류가 나요

`wrangler.toml`의 `database_id`가 아직 `여기에-d1-database-id-붙여넣기`로 남아 있으면 배포가 실패합니다. `npx wrangler d1 create royal-salon-db` 결과에서 받은 실제 ID로 바꿔주세요.

### 저장이 안 돼요

D1 테이블 생성 명령을 실행했는지 확인하세요.

```powershell
npx wrangler d1 execute royal-salon-db --remote --file migrations/0001_player_saves.sql
```

### 로컬에서는 되는데 Cloudflare에서 안 돼요

Cloudflare에서는 `server/index.ts`가 실행되지 않습니다. 대신 `functions/` 폴더가 API 역할을 합니다. 그래서 `functions/` 폴더와 `wrangler.toml`을 같이 배포해야 합니다.
