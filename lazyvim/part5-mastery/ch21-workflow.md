# Chapter 21: 실전 워크플로우 (LazyVim)

지금까지 Vim의 편집 언어, 설정, 플러그인을 하나씩 배워왔다. 이 챕터에서는 그 모든 조각을 **하나의 흐름으로 연결**한다. 실제 개발에서 마주치는 상황별로 어떤 도구를 어떤 순서로 사용하는지, 손에 익을 때까지 반복할 수 있는 워크플로우를 제시한다.

LazyVim은 이 워크플로우에 필요한 키맵과 플러그인을 **이미 내장**하고 있으므로, 별도의 키맵 설정 없이 바로 활용할 수 있다.

## 프로젝트 탐색 워크플로우

코드를 작성하는 시간보다 **읽고 찾는 시간**이 더 길다. Telescope, LSP, Treesitter를 조합하면 프로젝트 안에서 원하는 코드에 빠르게 도달할 수 있다.

### 파일 찾기 -> 심볼 검색 -> 정의 이동 -> 참조 찾기

LazyVim의 기본 키맵으로 구성하는 전형적인 탐색 흐름:

```
1. 파일 찾기          <leader><space>  (Telescope find_files)
2. 텍스트 검색        <leader>/        (Telescope live_grep)
3. 문서 심볼 검색     <leader>ss       (Telescope lsp_document_symbols)
4. 워크스페이스 심볼   <leader>sS       (Telescope lsp_workspace_symbols)
5. 정의로 이동        gd               (LSP go to definition)
6. 참조 찾기          gr               (LSP references)
7. 이전 위치로 복귀   <C-o>            (jump list)
```

#### 실전 시나리오: "이 함수를 누가 호출하고 있지?"

1. `<leader><space>`로 파일을 열거나, `<leader>/`로 함수 이름을 검색한다
2. 함수 정의 위에 커서를 놓는다
3. `gr`로 참조 목록을 확인한다 (Telescope로 표시)
4. 목록에서 원하는 위치를 선택하여 이동한다
5. `<C-o>`로 원래 위치로 돌아온다

#### 실전 시나리오: "이 타입의 구조를 파악하고 싶다"

1. `<leader>ss`로 현재 파일의 심볼 목록을 연다
2. 또는 `<leader>sS`로 워크스페이스 전체 심볼을 검색한다
3. 타입 정의로 이동한 뒤 `K`로 문서를 확인한다
4. `gd`로 관련 타입의 정의를 따라간다

### 탐색 키맵 정리 (LazyVim 기본 제공)

| 키 | 기능 | 분류 |
|----|------|------|
| `<leader><space>` | 파일 찾기 | 파일 |
| `<leader>/` | 텍스트 검색 (live_grep) | 검색 |
| `<leader>,` | 열린 버퍼 전환 | 버퍼 |
| `<leader>fr` | 최근 파일 | 파일 |
| `<leader>ss` | 문서 심볼 | 검색 |
| `<leader>sS` | 워크스페이스 심볼 | 검색 |
| `<leader>sw` | 커서 아래 단어 검색 | 검색 |
| `gd` | 정의로 이동 | LSP |
| `gr` | 참조 찾기 | LSP |
| `gI` | 구현으로 이동 | LSP |
| `gy` | 타입 정의로 이동 | LSP |
| `K` | 호버 문서 | LSP |

## 코드 편집 워크플로우

효율적인 편집은 "올바른 도구를 올바른 상황에 사용하는 것"이다.

### 빠른 변수명 변경: LSP rename vs ciw + n.

변수명을 변경하는 방법은 크게 두 가지다:

**방법 1: LSP rename (의미 기반)**

커서를 변수 위에 놓고 `<leader>cr` -> 새 이름 입력 -> `<CR>`. LSP가 프로젝트 전체에서 해당 심볼의 모든 참조를 정확하게 변경한다. 단순한 텍스트 치환이 아니라 **의미를 이해한** 변경이다.

**방법 2: ciw + n. (수동, 선택적)**

1. `*`로 현재 단어를 검색한다
2. `ciw`로 단어를 변경하고 새 이름을 입력한다
3. `<Esc>` -> `n`으로 다음 일치로 이동
4. `.`으로 변경을 반복하거나, `n`으로 건너뛴다

LSP rename이 더 정확하지만, **선택적으로 변경하고 싶을 때**는 `ciw` + `n.` 패턴이 유용하다.

### 여러 파일 동시 변경: quickfix list 활용

quickfix list는 여러 파일에 걸친 검색 결과를 한 곳에 모아 순차적으로 처리할 수 있는 도구다.

LazyVim에서는 quickfix 탐색 키맵이 기본 제공된다:

| 키 | 기능 |
|----|------|
| `]q` | 다음 quickfix 항목 |
| `[q` | 이전 quickfix 항목 |
| `<leader>xq` | quickfix 리스트 열기 |

#### 실전 시나리오: 여러 파일에서 API 엔드포인트 변경

1. `<leader>/`로 `/api/v1/users`를 검색한다
2. 결과 목록에서 변경할 항목을 `<Tab>`으로 선택한다 (또는 전체 선택)
3. `<C-q>`로 quickfix list에 보낸다
4. `:cdo s/\/api\/v1/\/api\/v2/g | update`로 일괄 변경한다

### 코드 블록 이동: LazyVim 기본 키맵

LazyVim은 줄 이동 키맵을 기본으로 제공한다:

| 키 | 동작 |
|----|------|
| `<A-j>` | 줄/선택 영역 아래로 이동 |
| `<A-k>` | 줄/선택 영역 위로 이동 |

Normal 모드와 Visual 모드 모두에서 동작한다. 별도 설정 불필요.

## 리팩토링 워크플로우

### 함수 추출, 변수 추출

Neovim에서의 리팩토링은 Vim의 편집 기능과 LSP 코드 액션(code action)을 조합하여 수행한다.

**함수 추출 (수동)**

1. `V`로 추출할 코드 블록을 줄 단위로 선택한다
2. `d`로 잘라낸다
3. 적절한 위치로 이동하여 새 함수를 작성한다
4. `p`로 코드를 붙여넣는다
5. 원래 위치에 함수 호출을 추가한다

**변수 추출 (수동)**

1. 반복되는 표현식 위에서 `viw` 또는 적절한 텍스트 오브젝트로 선택한다
2. `c`로 변경하고 새 변수명을 입력한다
3. 윗줄에 변수 선언을 추가한다 (`O`로 위에 새 줄 추가)
4. `n.`으로 나머지 동일 표현식도 변경한다

### LSP 코드 액션 활용

커서를 코드 위에 놓고 `<leader>ca`를 누르면 사용 가능한 액션 목록이 표시된다:

- 누락된 import 추가
- 사용하지 않는 import 제거
- 함수 추출 (일부 LSP 서버 지원)
- 타입 어노테이션 추가
- Quick fix 제안

`<leader>cA`로 소스 코드 액션(source action)에도 접근할 수 있다.

### 검색/치환으로 대규모 변경

프로젝트 전체에 걸친 대규모 변경은 검색/치환(search and replace)이 효율적이다.

```vim
" 현재 파일에서 치환
:%s/oldName/newName/gc     " c 플래그: 각 항목 확인

" 여러 파일에서 치환 (quickfix + cdo)
" <leader>/ 로 검색 후 <C-q>로 quickfix에 보내기
:cdo s/oldName/newName/g | update

" 정규표현식 활용
:%s/console\.log(.*)/\/\/ console.log(\1)/g   " 로그 주석 처리
```

## 디버깅 워크플로우: nvim-dap

LazyVim은 nvim-dap을 **extras로 제공**한다. VSCode와 동일한 DAP(Debug Adapter Protocol)를 사용하므로 다양한 언어를 지원한다.

### DAP extras 활성화

```vim
:LazyExtras
```

에서 `dap.core`를 활성화하거나, `lua/config/lazy.lua`에서:

```lua
{ import = "lazyvim.plugins.extras.dap.core" },
```

언어별 DAP 어댑터도 extras로 제공된다:

```lua
{ import = "lazyvim.plugins.extras.dap.nlua" },    -- Lua (Neovim)
```

또는 lang extras에 DAP가 포함된 경우도 있다 (예: `lang.python`, `lang.go`).

### 기본 제공 키맵

DAP extras를 활성화하면 다음 키맵이 자동으로 설정된다:

| 키 | 기능 |
|----|------|
| `<leader>db` | 브레이크포인트 토글 |
| `<leader>dB` | 조건부 브레이크포인트 |
| `<leader>dc` | 디버그 계속 (Continue) |
| `<leader>da` | Run with Args |
| `<leader>dC` | Run to Cursor |
| `<leader>dg` | Go to Line (실행 없이) |
| `<leader>di` | 스텝 인투 (Step Into) |
| `<leader>dj` | 아래로 (Down) |
| `<leader>dk` | 위로 (Up) |
| `<leader>dl` | 마지막 실행 반복 |
| `<leader>do` | 스텝 아웃 (Step Out) |
| `<leader>dO` | 스텝 오버 (Step Over) |
| `<leader>dp` | 일시 정지 (Pause) |
| `<leader>dr` | REPL 토글 |
| `<leader>ds` | 세션 |
| `<leader>dt` | 디버그 종료 (Terminate) |
| `<leader>dw` | 위젯 (Widgets) |

### 디버깅 흐름

```
1. <leader>db  -> 브레이크포인트 설정
2. <leader>dc  -> 디버그 시작/계속
3. <leader>dO  -> 한 줄씩 실행 (step over)
4. <leader>di  -> 함수 안으로 진입 (step into)
5. <leader>do  -> 함수 밖으로 나가기 (step out)
6. <leader>dc  -> 다음 브레이크포인트까지 계속
7. <leader>dt  -> 디버그 종료
```

### dap-ui

DAP extras에는 nvim-dap-ui가 포함되어 있다. 디버깅 시작 시 자동으로 열리며, 변수, 콜 스택, 브레이크포인트 목록을 시각적으로 보여준다.

```
┌─────────────────┬──────────────────────────┬─────────────────┐
│   Variables     │                          │   Call Stack    │
│                 │                          │                 │
│ x = 42          │     소스 코드             │ main()          │
│ name = "hello" │     (브레이크포인트 표시)    │  └ calculate()  │
│ result = nil   │                          │    └ add()      │
│                 │                          │                 │
├─────────────────┤                          ├─────────────────┤
│   Watches       │                          │   Breakpoints   │
│                 │                          │                 │
│ x + y = ?      │                          │ main.py:10      │
│                 │                          │ utils.py:25     │
└─────────────────┴──────────────────────────┴─────────────────┘
│                        REPL / Console                        │
└──────────────────────────────────────────────────────────────┘
```

## 터미널 통합

### LazyVim 내장 터미널 키맵

LazyVim은 터미널 관련 키맵을 기본 제공한다:

| 키 | 기능 |
|----|------|
| `<C-/>` | 터미널 토글 (하단) |
| `<C-_>` | 터미널 토글 (하단, 일부 터미널) |
| `<leader>ft` | 터미널 열기 (루트 디렉터리) |
| `<leader>fT` | 터미널 열기 (현재 디렉터리) |
| `<Esc><Esc>` | 터미널 Normal 모드 전환 |

터미널 모드에서 창 이동도 기본 설정되어 있다:

| 키 | 기능 |
|----|------|
| `<C-h>` | 왼쪽 창으로 이동 |
| `<C-j>` | 아래 창으로 이동 |
| `<C-k>` | 위 창으로 이동 |
| `<C-l>` | 오른쪽 창으로 이동 |

### 터미널 모드 이해

```
┌─────────────────────────────────────────────┐
│  Terminal 모드 (입력이 셸로 전달됨)           │
│                                             │
│  <Esc><Esc>  -> Normal 모드로 전환           │
│  i 또는 a    <- Normal 모드에서 Terminal로 복귀│
└─────────────────────────────────────────────┘
```

### lazygit 통합

LazyVim에는 lazygit 연동이 **기본 내장**되어 있다. 시스템에 lazygit이 설치되어 있으면 바로 사용할 수 있다.

```bash
# lazygit 설치
brew install lazygit          # macOS
sudo apt install lazygit      # Ubuntu
```

| 키 | 기능 |
|----|------|
| `<leader>gg` | lazygit 열기 (루트 디렉터리) |
| `<leader>gG` | lazygit 열기 (현재 디렉터리) |
| `<leader>gl` | lazygit 로그 |
| `<leader>gL` | lazygit 로그 (현재 파일) |

> **팁**: lazygit은 터미널 기반 Git UI로, 스테이징, 커밋, 브랜치 관리, 리베이스 등을 직관적으로 수행할 수 있다. LazyVim과 함께 사용하면 Neovim을 떠나지 않고 모든 Git 작업을 처리할 수 있다.

## Git 워크플로우

### gitsigns로 hunk 단위 작업

LazyVim에 기본 포함된 gitsigns로 변경 사항을 줄 단위로 관리한다.

| 키 | 기능 |
|----|------|
| `]h` | 다음 hunk로 이동 |
| `[h` | 이전 hunk로 이동 |
| `<leader>ghs` | hunk 스테이징 |
| `<leader>ghr` | hunk 리셋 |
| `<leader>ghS` | 버퍼 전체 스테이징 |
| `<leader>ghR` | 버퍼 전체 리셋 |
| `<leader>ghp` | hunk 미리보기 |
| `<leader>ghb` | 줄 blame |
| `<leader>ghd` | diff 보기 |
| `<leader>ub` | 인라인 blame 토글 |

hunk 단위 워크플로우:

```
1. ]h / [h       -> hunk 사이를 탐색
2. <leader>ghp   -> 변경 내용 미리보기
3. <leader>ghs   -> 원하는 hunk만 스테이지
4. <leader>ghr   -> 불필요한 변경 되돌리기
```

### Git 워크플로우 종합 (LazyVim)

일상적인 Git 작업 흐름:

```
1. 코드 작성
2. ]h / [h        -> 변경된 hunk 확인
3. <leader>ghp    -> 변경 내용 미리보기
4. <leader>ghs    -> 원하는 변경만 스테이지
5. <leader>gg     -> lazygit으로 커밋/푸시
```

lazygit을 사용하면 fugitive/neogit 같은 별도 플러그인 없이도 스테이징, 커밋, 푸시, 브랜치 관리를 모두 처리할 수 있다.

### diffview.nvim (선택)

더 상세한 diff 비교가 필요하다면 diffview.nvim을 추가할 수 있다:

```lua
-- lua/plugins/diffview.lua
return {
  {
    "sindrets/diffview.nvim",
    keys = {
      { "<leader>gd", "<cmd>DiffviewOpen<CR>", desc = "Diff 보기" },
      { "<leader>gh", "<cmd>DiffviewFileHistory %<CR>", desc = "파일 히스토리" },
      { "<leader>gH", "<cmd>DiffviewFileHistory<CR>", desc = "프로젝트 히스토리" },
    },
  },
}
```

## 프로젝트별 설정: exrc, .nvim.lua

프로젝트마다 다른 포매터, 린터, 빌드 명령이 필요할 수 있다. Neovim은 프로젝트 루트에 있는 설정 파일을 자동으로 로드하는 기능을 제공한다.

### exrc 활성화

LazyVim에서 exrc를 활성화하려면:

```lua
-- lua/config/options.lua
vim.o.exrc = true
```

이제 프로젝트 루트에 `.nvim.lua` 파일을 만들면 해당 디렉터리에서 Neovim을 열 때 자동으로 실행된다.

### .nvim.lua 활용 예시

```lua
-- 프로젝트 루트의 .nvim.lua

-- 프로젝트 전용 탭 크기
vim.bo.tabstop = 4
vim.bo.shiftwidth = 4

-- 프로젝트 전용 빌드 명령
vim.keymap.set('n', '<leader>mb', function()
  vim.cmd('split | terminal make build')
end, { desc = '프로젝트 빌드' })

-- 프로젝트 전용 테스트 명령
vim.keymap.set('n', '<leader>mt', function()
  vim.cmd('split | terminal npm test')
end, { desc = '테스트 실행' })
```

> **주의**: `exrc`를 활성화하면 신뢰할 수 없는 프로젝트의 `.nvim.lua`가 실행될 수 있다. Neovim 0.9 이상에서는 처음 실행 시 신뢰 여부를 묻는 프롬프트가 표시된다.

## 실습: 워크플로우 연습

**연습 1: 탐색 워크플로우**

1. 자신의 프로젝트를 Neovim으로 연다
2. `<leader>/`로 특정 함수 이름을 검색하여 정의를 찾는다
3. `gd`로 관련 타입의 정의로 이동한다
4. `gr`로 해당 타입이 사용되는 모든 곳을 확인한다
5. `<C-o>`로 원래 위치로 돌아온다
6. `<leader>sR`로 마지막 검색을 다시 열어본다

**연습 2: 편집 워크플로우**

1. 변수명 하나를 선택하고 `<leader>cr`로 LSP rename을 수행한다
2. `<leader>/`로 특정 문자열을 검색하고 `<C-q>`로 quickfix에 보낸다
3. `:cdo s/old/new/g | update`로 일괄 변경한다

**연습 3: Git 워크플로우**

1. 파일을 수정한 뒤 `]h`로 변경된 hunk을 탐색한다
2. `<leader>ghp`로 변경 내용을 확인한다
3. `<leader>ghs`로 원하는 hunk만 스테이지한다
4. `<leader>gg`로 lazygit을 열어 커밋한다

**연습 4: 디버깅 워크플로우**

1. `dap.core` extras를 활성화한다
2. `<leader>db`로 브레이크포인트를 설정한다
3. `<leader>dc`로 디버깅을 시작한다
4. `<leader>dO`로 한 줄씩 실행해 본다

## 핵심 정리

- **탐색**: `<leader><space>`(파일) -> `<leader>/`(텍스트) -> `gd`/`gr`(LSP) -> `<C-o>`(복귀) 흐름을 체화하라
- **편집**: LSP rename(`<leader>cr`)은 정확하고, `ciw` + `n.`은 유연하다. 상황에 맞게 선택한다
- **quickfix list**: `<leader>/` -> `<C-q>` -> `:cdo` 패턴으로 여러 파일을 일괄 변경한다
- **디버깅**: `dap.core` extras를 활성화하면 `<leader>d` 접두사로 디버깅을 제어한다
- **터미널**: `<C-/>`로 터미널 토글, `<leader>ft`로 터미널 열기
- **lazygit**: `<leader>gg`로 lazygit을 열어 모든 Git 작업을 처리한다 (LazyVim 기본 내장)
- **gitsigns**: `]h`/`[h`로 hunk 탐색, `<leader>ghs`로 스테이징, `<leader>ghp`로 미리보기
- **프로젝트별 설정**: `.nvim.lua`로 프로젝트마다 다른 설정, 키맵, 빌드 명령을 정의한다
