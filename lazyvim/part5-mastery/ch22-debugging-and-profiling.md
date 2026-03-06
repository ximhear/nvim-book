# Chapter 22: 성능과 트러블슈팅 (LazyVim)

Neovim 설정이 복잡해질수록 문제가 생길 가능성도 높아진다. 시작이 느려지거나, LSP가 동작하지 않거나, 플러그인이 충돌하거나. 이 챕터에서는 문제를 **체계적으로 진단하고 해결하는 방법**을 다룬다.

LazyVim은 잘 구성된 배포판이므로 문제가 적지만, extras 추가나 커스터마이징 과정에서 이슈가 발생할 수 있다.

## Neovim 시작 시간 측정

Neovim이 느리게 시작된다면 가장 먼저 할 일은 **측정**이다.

### --startuptime 플래그

```bash
nvim --startuptime startup.log
```

이 명령은 Neovim이 시작되면서 각 파일을 로드하는 데 걸린 시간을 `startup.log` 파일에 기록한다. Neovim을 종료한 뒤 로그를 확인한다:

```bash
nvim startup.log
```

로그의 형식:

```
times in msec
 clock   self+sourced   self:  sourced script
 clock   elapsed:              other lines

000.006  000.006: --- NVIM STARTING ---
...
010.234  002.345  002.345: sourcing ~/.config/nvim/init.lua
015.678  003.210  003.210: sourcing .../lazy/telescope.nvim/plugin/telescope.lua
...
050.123  000.000: --- NVIM STARTED ---
```

각 열의 의미:

| 열 | 의미 |
|----|------|
| `clock` | Neovim 시작 시점부터의 누적 시간 (밀리초) |
| `self+sourced` | 해당 스크립트와 그 안에서 호출한 스크립트의 총 시간 |
| `self` | 해당 스크립트 자체에 소요된 시간 |

마지막 줄의 `--- NVIM STARTED ---` 옆의 시간이 **총 시작 시간**이다. 100ms 이하면 우수하고, 200ms 이하면 양호하며, 500ms 이상이면 최적화가 필요하다.

> **팁**: `sort -k 2 -n startup.log | tail -20` 명령으로 가장 느린 상위 20개를 확인할 수 있다.

## lazy.nvim 프로파일러

LazyVim은 lazy.nvim의 내장 프로파일러를 바로 사용할 수 있다.

```vim
:Lazy profile
```

각 플러그인의 로드 시간을 시각적으로 보여준다:

```
┌────────────────────────────────────────────────────┐
│ Profile                                            │
├────────────────────────────────────────────────────┤
│ telescope.nvim         ████████████        12.3ms  │
│ nvim-treesitter        ████████            8.5ms   │
│ nvim-lspconfig         ███████             7.2ms   │
│ nvim-cmp               ██████              6.1ms   │
│ gitsigns.nvim          ███                 3.4ms   │
│ which-key.nvim         ██                  2.1ms   │
│ ...                                                │
├────────────────────────────────────────────────────┤
│ Total: 45.2ms                                      │
└────────────────────────────────────────────────────┘
```

`:Lazy`를 실행한 뒤 상단의 `Profile` 탭을 클릭하거나 `P`를 눌러도 동일한 화면을 볼 수 있다.

## 느린 플러그인 식별과 해결

### 지연 로딩 이해

LazyVim은 이미 대부분의 플러그인에 적절한 지연 로딩이 적용되어 있다. 하지만 사용자가 추가한 플러그인에는 직접 지연 로딩을 설정해야 한다.

```lua
-- 나쁜 예: 시작 시 바로 로드
{ "some/plugin" }

-- 좋은 예: 명령어 실행 시 로드
{
  "some/plugin",
  cmd = "SomeCommand",
  keys = {
    { "<leader>xx", "<cmd>SomeCommand<CR>" },
  },
}

-- 좋은 예: 특정 파일 타입에서만 로드
{
  "some/plugin",
  ft = "java",
}

-- 좋은 예: 특정 이벤트에서 로드
{
  "some/plugin",
  event = { "BufReadPre", "BufNewFile" },
}
```

lazy.nvim의 지연 로딩 트리거 정리:

| 옵션 | 설명 | 예시 |
|------|------|------|
| `cmd` | 명령어 실행 시 | `cmd = "Telescope"` |
| `keys` | 키맵 사용 시 | `keys = { "<leader>ff" }` |
| `ft` | 파일 타입 열 때 | `ft = { "python", "lua" }` |
| `event` | 이벤트 발생 시 | `event = "BufReadPre"` |
| `lazy = true` | 수동 로드 | `require("plugin")` 호출 시 |

### 불필요한 플러그인 비활성화

LazyVim의 내장 플러그인 중 사용하지 않는 것이 있다면 비활성화할 수 있다:

```lua
-- lua/plugins/disabled.lua
return {
  -- 사용하지 않는 플러그인 비활성화
  { "echasnovski/mini.pairs", enabled = false },
  { "folke/flash.nvim", enabled = false },
}
```

플러그인 관리 명령:

```vim
:Lazy          " 플러그인 목록 확인
:Lazy clean    " 설정에서 제거된 플러그인 삭제
:Lazy update   " 플러그인 업데이트
:Lazy sync     " clean + update
```

## :checkhealth 활용

`:checkhealth`는 Neovim의 환경을 진단하는 내장 도구다.

```vim
:checkhealth              " 전체 진단
:checkhealth nvim         " Neovim 코어만 진단
:checkhealth telescope    " Telescope만 진단
:checkhealth lspconfig    " LSP 설정 진단
:checkhealth lazy         " lazy.nvim 진단
```

출력은 세 가지 상태로 구분된다:

```
- OK: 정상 동작
- WARNING: 동작은 하지만 개선 권장
- ERROR: 문제 있음, 해결 필요
```

주요 점검 항목:

| 항목 | 확인 내용 |
|------|----------|
| `nvim` | Neovim 버전, 클립보드, 터미널 설정 |
| `provider` | Python, Node.js, Ruby 프로바이더 상태 |
| `lspconfig` | LSP 서버 설치 여부, 설정 오류 |
| `telescope` | 외부 의존성 (ripgrep, fd) 설치 여부 |
| `treesitter` | 파서 설치 상태, 컴파일러 확인 |
| `lazy` | lazy.nvim 상태 |

> **팁**: 문제가 발생하면 가장 먼저 `:checkhealth`를 실행하라. 대부분의 환경 문제를 여기서 진단할 수 있다.

## 일반적인 문제 해결

### LSP 서버가 시작되지 않을 때

**증상**: 자동 완성, 정의 이동(`gd`), 참조 찾기(`gr`)가 동작하지 않는다.

**진단 순서**:

```vim
" 1. LSP 상태 확인
:LspInfo

" 2. LSP 로그 확인
:LspLog

" 3. LSP 서버가 설치되어 있는지 확인
:Mason

" 4. LazyVim 포매팅 정보
:LazyFormatInfo
```

**흔한 원인과 해결**:

| 원인 | 해결 |
|------|------|
| LSP 서버 미설치 | `:Mason`에서 설치 또는 extras 활성화 |
| 루트 디렉터리 인식 실패 | 프로젝트 루트에 설정 파일 확인 (`package.json` 등) |
| LSP 서버 버전 불일치 | `:MasonUpdate`로 업데이트 |
| extras 미활성화 | `:LazyExtras`에서 해당 언어 extras 활성화 |
| filetype 미인식 | `:set filetype?`로 확인 |

```lua
-- LSP 디버그 로깅 활성화
vim.lsp.set_log_level("debug")
-- 로그 확인: :LspLog
```

### Treesitter 파서 오류

**증상**: 구문 하이라이팅이 깨지거나, 텍스트 오브젝트가 동작하지 않는다.

```vim
" 파서 상태 확인
:TSInstallInfo

" 파서 재설치
:TSInstall {language}

" 모든 파서 업데이트
:TSUpdate
```

**흔한 원인**:

- C 컴파일러가 없어서 파서 빌드 실패 -> `gcc` 또는 `clang` 설치
- Neovim 업데이트 후 파서 호환성 문제 -> `:TSUpdate`로 모든 파서 재빌드
- lockfile 불일치 -> `:Lazy sync` 후 `:TSUpdate`

### 플러그인 충돌

두 플러그인이 같은 키맵을 설정하거나, 같은 이벤트에 반응하여 충돌할 수 있다.

**진단 방법**:

1. 의심되는 플러그인을 하나씩 비활성화하여 원인을 좁힌다:

```lua
{
  "some/plugin",
  enabled = false,   -- 일시 비활성화
}
```

2. LazyVim 기본 플러그인과 사용자 추가 플러그인 사이의 충돌을 확인한다

### 키맵 충돌 확인

특정 키가 의도대로 동작하지 않을 때, 어디서 매핑되었는지 확인한다:

```vim
:verbose map <leader>ff    " <leader>ff가 어디서 매핑되었는지 확인
:verbose nmap <C-p>        " Normal 모드에서 <C-p> 확인
:verbose imap <C-n>        " Insert 모드에서 <C-n> 확인
```

출력에는 매핑을 설정한 파일과 줄 번호가 표시된다:

```
n  <leader>ff    <cmd>Telescope find_files<CR>
        Last set from ~/.local/share/nvim/lazy/LazyVim/lua/lazyvim/plugins/editor.lua line 42
```

Telescope로 키맵을 검색할 수도 있다:

```
<leader>sk     " LazyVim 기본 키맵: 키맵 검색
```

## 로그 확인

### :messages

Neovim이 표시한 메시지의 히스토리를 확인한다:

```vim
:messages              " 메시지 히스토리 보기
:messages clear        " 메시지 히스토리 지우기
```

에러 메시지가 빠르게 지나갔을 때 `:messages`로 다시 확인할 수 있다.

### LazyVim 알림 히스토리

LazyVim에는 noice.nvim이 내장되어 알림 히스토리를 관리한다:

| 키 | 기능 |
|----|------|
| `<leader>sna` | 모든 알림 히스토리 |
| `<leader>snd` | 알림 해제 (Dismiss) |
| `<leader>snl` | 마지막 알림 |
| `<leader>snh` | 알림 히스토리 (Telescope) |

```vim
:Noice           " 알림 히스토리 보기
:Noice errors    " 에러만 보기
:Noice dismiss   " 모든 알림 해제
```

### Lua 에러 디버깅

Lua 설정에서 에러가 발생하면 시작 시 에러 메시지가 표시된다. 에러 메시지를 주의 깊게 읽으면 대부분 원인을 찾을 수 있다:

```
E5113: Error while calling lua chunk: ~/.config/nvim/lua/plugins/lsp.lua:25:
attempt to index a nil value
```

이 메시지는 `lsp.lua`의 25번째 줄에서 `nil` 값에 접근하려 했다는 뜻이다. 해당 줄을 확인하고, 변수가 올바르게 초기화되었는지 점검한다.

## 설정 디버깅: 최소 설정으로 문제 재현

문제가 복잡할 때는 **최소 재현(minimal reproduction)**이 가장 효과적인 진단 방법이다.

### nvim --clean

모든 사용자 설정과 플러그인 없이 Neovim을 시작한다:

```bash
nvim --clean
```

이 상태에서 문제가 재현되면 Neovim 자체의 문제이고, 재현되지 않으면 사용자 설정이 원인이다.

### nvim -u minimal.lua

문제를 재현하는 최소한의 설정 파일을 만든다:

```lua
-- /tmp/minimal.lua

vim.o.number = true
vim.o.termguicolors = true

local lazypath = vim.fn.stdpath("data") .. "/lazy-minimal/lazy.nvim"
if not vim.uv.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
  -- 문제와 관련된 플러그인만 추가
  { "LazyVim/LazyVim", import = "lazyvim.plugins" },
}, {
  root = vim.fn.stdpath("data") .. "/lazy-minimal",
})
```

```bash
nvim -u /tmp/minimal.lua
```

이 방법으로:

1. 문제가 재현되면 -> LazyVim 또는 해당 플러그인의 이슈로 보고
2. 재현되지 않으면 -> 사용자 커스터마이징이 원인

> **팁**: GitHub에 이슈를 올릴 때 최소 재현 설정을 함께 제공하면 유지 관리자가 빠르게 문제를 파악할 수 있다.

## LazyVim 업데이트 전략

### LazyVim 업데이트

```vim
:Lazy update    " 모든 플러그인 + LazyVim 업데이트
:Lazy sync      " clean + update
```

### LazyVim 버전 확인과 변경 사항

LazyVim의 변경 사항은 GitHub 릴리스에서 확인할 수 있다. 주요 변경이 있을 때는 `:LazyExtras`에서 새로운 extras가 추가되기도 한다.

### Neovim stable vs nightly

| 채널 | 특징 | 권장 대상 |
|------|------|----------|
| **stable** | 안정적, 릴리스 주기가 길다 | 대부분의 사용자 |
| **nightly** | 최신 기능, 가끔 불안정 | 최신 기능이 필요한 사용자, 플러그인 개발자 |

```bash
# stable 설치 (Homebrew)
brew install neovim

# nightly 설치 (Homebrew)
brew install --HEAD neovim

# 현재 버전 확인
nvim --version
```

### 안전한 업데이트 절차

1. **LazyVim 릴리스 노트를 확인한다**
2. **설정을 백업한다**: `cp -r ~/.config/nvim ~/.config/nvim.bak`
3. **`:Lazy update`를 실행한다**
4. **`:checkhealth`를 실행한다**
5. **Treesitter 파서를 재빌드한다**: `:TSUpdate`
6. **문제가 생기면 롤백한다**: `:Lazy restore`

## 설정 백업과 버전 관리: dotfiles 관리

Neovim 설정은 코드와 마찬가지로 **버전 관리**해야 한다.

### LazyVim dotfiles 구조

```
~/dotfiles/
├── nvim/
│   ├── init.lua
│   ├── lua/
│   │   ├── config/
│   │   │   ├── lazy.lua         -- lazy.nvim 설정, extras import
│   │   │   ├── options.lua      -- 사용자 옵션
│   │   │   ├── keymaps.lua      -- 사용자 키맵
│   │   │   └── autocmds.lua     -- 사용자 자동 명령
│   │   └── plugins/
│   │       ├── colorscheme.lua  -- 컬러스킴
│   │       ├── lsp.lua          -- LSP 커스터마이징
│   │       ├── formatting.lua   -- 포매터 설정
│   │       └── ...
│   └── lazy-lock.json
├── .gitignore
└── README.md
```

### lazy-lock.json의 역할

`lazy-lock.json`은 모든 플러그인의 정확한 커밋 해시를 기록한다. 이 파일을 Git에 포함하면:

- 다른 컴퓨터에서 **동일한 플러그인 버전**을 재현할 수 있다
- 플러그인 업데이트 후 문제가 생기면 이전 상태로 **롤백**할 수 있다

```bash
# lazy-lock.json도 커밋에 포함
git add nvim/lazy-lock.json
git commit -m "update plugin versions"
```

```vim
" lock 파일 기준으로 플러그인 복원
:Lazy restore
```

### 여러 컴퓨터에서 동기화

```bash
# 새 컴퓨터에서 설정 복원
git clone https://github.com/username/dotfiles.git ~/dotfiles
ln -s ~/dotfiles/nvim ~/.config/nvim

# Neovim 실행 시 LazyVim이 자동으로 플러그인 설치
nvim
```

## 실습: 트러블슈팅 연습

**연습 1: 시작 시간 측정**

1. `nvim --startuptime /tmp/startup.log`으로 Neovim을 실행한다
2. Neovim을 종료하고 `/tmp/startup.log`를 열어 총 시작 시간을 확인한다
3. 가장 시간이 오래 걸리는 항목 3개를 찾는다
4. `:Lazy profile`로 플러그인별 로드 시간을 확인한다

**연습 2: 환경 점검**

1. `:checkhealth`를 실행한다
2. WARNING이나 ERROR가 있는지 확인한다
3. 발견된 문제를 해결한다

**연습 3: 키맵 진단**

1. `<leader>sk`로 전체 키맵을 검색한다
2. `:verbose map <leader>cr`로 rename 키맵이 어디서 설정되었는지 확인한다
3. 충돌하는 키맵이 있는지 점검한다

**연습 4: 최소 설정으로 시작**

1. `nvim --clean`으로 깨끗한 상태에서 Neovim을 시작한다
2. 평소와 다른 동작이 있는지 비교한다

## 핵심 정리

- **시작 시간 측정**: `nvim --startuptime`과 `:Lazy profile`로 병목 지점을 찾는다
- **지연 로딩**: 사용자 추가 플러그인에 `cmd`, `keys`, `ft`, `event` 트리거를 설정한다 (LazyVim 내장 플러그인은 이미 최적화되어 있다)
- **플러그인 비활성화**: `enabled = false`로 LazyVim 내장 플러그인도 비활성화할 수 있다
- **`:checkhealth`**: 환경 문제의 80%를 진단하는 첫 번째 도구
- **LSP 문제**: `:LspInfo` -> `:LspLog` -> `:Mason` -> `:LazyExtras` 순으로 진단한다
- **키맵 충돌**: `<leader>sk` 또는 `:verbose map {key}`로 매핑 출처를 확인한다
- **알림 히스토리**: `<leader>sna` 또는 `:Noice`로 지나간 알림을 다시 확인한다
- **최소 재현**: `nvim --clean`과 `nvim -u minimal.lua`로 문제 원인을 격리한다
- **업데이트**: `:Lazy update` 후 문제 시 `:Lazy restore`로 롤백한다
- **dotfiles**: Git으로 관리하고, `lazy-lock.json`을 포함하여 플러그인 버전을 고정한다
