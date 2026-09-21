이사 준비 다이어리
부부가 함께 이사/인테리어 일정, 할일, 비용을 공유해서 관리하는 웹앱입니다.
데이터는 Supabase(무료 요금제)를 통해 실시간으로 저장·공유됩니다.
---
1. Supabase 설정 (데이터 저장/공유용)
이미 가지고 있는 Supabase 프로젝트를 그대로 사용하면 됩니다. 새 프로젝트를 만들어도 되고, 기존 프로젝트에 테이블만 추가해도 됩니다.
supabase.com 대시보드에서 사용할 프로젝트를 엽니다.
왼쪽 메뉴 SQL Editor → New query로 들어가서 아래 SQL을 전체 붙여넣고 Run을 누릅니다.
```sql
-- 1) 데이터를 저장할 테이블
create table if not exists households (
  id text primary key,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

-- 2) 보안 정책(RLS) 켜기
alter table households enable row level security;

-- 3) 익명 키(anon key)로 읽기/쓰기를 허용하는 최소한의 정책
create policy "Allow anon select" on households
  for select using (true);

create policy "Allow anon insert" on households
  for insert with check (true);

create policy "Allow anon update" on households
  for update using (true);

-- 4) 실시간 동기화를 위해 이 테이블을 realtime publication에 추가
alter publication supabase_realtime add table households;
```
왼쪽 메뉴 Project Settings → API로 이동해 다음 두 값을 복사합니다.
Project URL (예: `https://xxxxxxxx.supabase.co`)
anon public 키 (긴 문자열)
`index.html` 파일을 열어 상단의 `SUPABASE_URL`, `SUPABASE_ANON_KEY` 값에 그대로 붙여넣습니다.
같은 부분의 `HOUSEHOLD_ID` 값을 두 분만 아는 문자열로 바꿔주세요 (예: `kimpark-house-2026`). 이 값은 households 테이블에서 두 분의 행(row)을 구분하는 키입니다.
> ⚠️ 참고: 위 3번 정책은 "anon key를 아는 사람이라면 누구나" 해당 테이블에 접근 가능하게 합니다. `index.html`이 공개 저장소에 올라가면 `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `HOUSEHOLD_ID`가 모두 공개되므로, 이 값들을 아는 사람은 이론적으로 데이터에 접근할 수 있습니다. 더 강하게 보호하고 싶다면 아래 "저장소를 비공개로 만들기" 단계를 꼭 함께 진행하세요. 참고로 anon key는 Supabase가 브라우저에 그대로 노출되도록 설계된 키이고, 실제 보호는 RLS 정책(위 SQL의 3번)이 담당합니다 — 필요하면 나중에 이메일 로그인 기반 정책으로 강화할 수 있어요.
---
2. GitHub에 배포하기
github.com에 로그인합니다 (계정: `sh82.jeong@samsung.com`으로 가입/로그인).
오른쪽 위 + → New repository 클릭.
Repository name: 예) `our-moving-diary`
Private로 설정 (위의 보안 참고사항 때문에 권장)
"Add a README file" 체크 해제 (이미 있는 README를 올릴 것이므로)
저장소가 생성되면 "uploading an existing file" 링크를 클릭합니다.
이 폴더의 `index.html`과 `README.md` 두 파일을 끌어다 놓고 Commit changes를 누릅니다.
저장소 상단 메뉴 Settings → Pages로 이동합니다.
Source를 `Deploy from a branch`로, Branch를 `main` / `/(root)`로 선택하고 Save를 누릅니다.
1~2분 후 같은 페이지에 나오는 주소(예: `https://sh82jeong.github.io/our-moving-diary/`)로 접속하면 앱이 열립니다.
> Private 저장소에서도 GitHub Pages 링크 자체는 공개 URL입니다(저장소 코드만 비공개). 이 링크를 두 분만 아는 상태로 유지해주세요.
---
3. 사용 시작하기
배포된 링크를 본인과 남편 각자의 휴대폰/PC에서 엽니다.
오른쪽 위 설정을 눌러 이사 예정일, 총 예산, 두 사람의 이름을 입력하고, "나는 누구인가요"에서 본인 이름을 선택 후 저장합니다. (이 선택은 그 기기에만 저장되어, 남편이 열어도 남편 이름이 기본으로 남아있게 됩니다.)
이후로는 두 분이 각자 항목을 추가하면 Supabase Realtime을 통해 서로의 화면에 즉시 반영됩니다.
---
문제가 생겼을 때
화면에 **"설정이 필요해요"**가 보이면 → `SUPABASE_URL`, `SUPABASE_ANON_KEY` 값을 아직 채우지 않은 상태입니다. 1단계를 다시 확인해주세요.
화면에 **"Supabase 연결에 실패했어요"**가 보이면 → `households` 테이블이나 RLS 정책이 SQL대로 만들어졌는지, Project URL/anon key 오타가 없는지 확인하세요.
실시간으로 반영이 안 되면 → `alter publication supabase_realtime add table households;`가 정상적으로 실행됐는지 SQL Editor에서 다시 실행해보세요.
데이터가 서로 다르게 보이면 → 두 분이 `HOUSEHOLD_ID` 값을 다르게 입력했을 수 있습니다. `index.html`의 `HOUSEHOLD_ID`가 두 분 모두 동일한지 확인하세요.
