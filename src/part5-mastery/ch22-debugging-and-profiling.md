# Chapter 22: 성능과 트러블슈팅

Neovim 설정이 복잡해질수록 문제가 생길 가능성도 높아진다. 시작이 느려지거나, LSP가 동작하지 않거나, 플러그인이 충돌하거나. 이 챕터에서는 문제를 **체계적으로 진단하고 해결하는 방법**을 다룬다. Neovim을 오래 사용할수록 이 챕터의 내용이 반복적으로 도움이 될 것이다.

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

로그의 형식은 다음과 같다:

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

> **팁**: 더 읽기 쉬운 결과를 원한다면 로그 파일을 정렬하여 가장 느린 항목을 찾을 수 있다. `sort -k 2 -n startup.log | tail -20` 명령으로 상위 20개를 확인할 수 있다.

## lazy.nvim 프로파일러

lazy.nvim은 내장 프로파일러(profiler)를 제공한다.

```vim
:Lazy profile
```

이 명령은 각 플러그인의 로드 시간을 시각적으로 보여준다:

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

### 지연 로딩 최적화

프로파일러에서 느린 플러그인을 발견했다면 **지연 로딩(lazy loading)**으로 개선할 수 있다.

```lua
-- 나쁜 예: 시작 시 바로 로드
{ 'nvim-telescope/telescope.nvim' }

-- 좋은 예: 명령어 실행 시 로드
{
  'nvim-telescope/telescope.nvim',
  cmd = 'Telescope',                     -- :Telescope 명령 사용 시 로드
  keys = {
    { '<leader>ff', '<cmd>Telescope find_files<CR>' },
    { '<leader>fg', '<cmd>Telescope live_grep<CR>' },
  },
}

-- 좋은 예: 특정 파일 타입에서만 로드
{
  'mfussenegger/nvim-jdtls',
  ft = 'java',                           -- Java 파일을 열 때만 로드
}

-- 좋은 예: 특정 이벤트에서 로드
{
  'lewis6991/gitsigns.nvim',
  event = { 'BufReadPre', 'BufNewFile' }, -- 파일을 열 때 로드
}
```

lazy.nvim의 지연 로딩 트리거(trigger) 정리:

| 옵션 | 설명 | 예시 |
|------|------|------|
| `cmd` | 명령어 실행 시 | `cmd = 'Telescope'` |
| `keys` | 키맵 사용 시 | `keys = { '<leader>ff' }` |
| `ft` | 파일 타입 열 때 | `ft = { 'python', 'lua' }` |
| `event` | 이벤트 발생 시 | `event = 'BufReadPre'` |
| `lazy = true` | 수동 로드 | `require('plugin')` 호출 시 |

### 불필요한 플러그인 제거

플러그인이 많아질수록 충돌과 성능 저하의 가능성이 높아진다. 정기적으로 다음을 점검한다:

- **최근 한 달간 사용하지 않은 플러그인**이 있는가?
- **기능이 겹치는 플러그인**이 있는가? (예: 여러 파일 탐색기)
- **Neovim 내장 기능으로 대체할 수 있는** 플러그인이 있는가?

```vim
:Lazy          " 플러그인 목록 확인
:Lazy clean    " 설정에서 제거된 플러그인 삭제
```

## :checkhealth 활용

`:checkhealth`는 Neovim의 환경을 진단하는 내장 도구다.

```vim
:checkhealth              " 전체 진단
:checkhealth nvim         " Neovim 코어만 진단
:checkhealth telescope    " Telescope만 진단
:checkhealth lspconfig    " LSP 설정 진단
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
| `treesitter` | 파서(parser) 설치 상태, 컴파일러 확인 |

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
:Mason            " mason.nvim 사용 시
```

**흔한 원인과 해결**:

| 원인 | 해결 |
|------|------|
| LSP 서버 미설치 | `:Mason`에서 설치 또는 시스템에 직접 설치 |
| 루트 디렉터리 인식 실패 | 프로젝트 루트에 설정 파일 확인 (`package.json`, `pyproject.toml` 등) |
| LSP 서버 버전 불일치 | `:MasonUpdate`로 업데이트 |
| filetypeq 미인식 | `:set filetype?`로 확인, 수동 설정 필요할 수 있음 |

```lua
-- LSP 디버그 로깅 활성화
vim.lsp.set_log_level('debug')
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

- C 컴파일러(compiler)가 없어서 파서 빌드 실패 → `gcc` 또는 `clang` 설치
- Neovim 업데이트 후 파서 호환성 문제 → `:TSUpdate`로 모든 파서 재빌드
- lockfile 불일치 → 플러그인과 파서를 함께 업데이트

### 플러그인 충돌

두 플러그인이 같은 키맵을 설정하거나, 같은 이벤트에 반응하여 충돌할 수 있다.

**진단 방법**:

1. 의심되는 플러그인을 하나씩 비활성화하여 원인을 좁힌다
2. lazy.nvim에서 플러그인을 일시적으로 비활성화한다:

```lua
{
  'some/plugin',
  enabled = false,   -- 일시 비활성화
}
```

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
        Last set from ~/.config/nvim/lua/plugins/telescope.lua line 15
```

모든 매핑을 검색하려면:

```vim
:Telescope keymaps     " Telescope로 키맵 검색
```

## 로그 확인

### :messages

Neovim이 표시한 메시지의 히스토리를 확인한다:

```vim
:messages              " 메시지 히스토리 보기
:messages clear        " 메시지 히스토리 지우기
```

에러 메시지가 빠르게 지나갔을 때 `:messages`로 다시 확인할 수 있다.

### vim.notify 히스토리

nvim-notify 플러그인을 사용하고 있다면 알림 히스토리를 확인할 수 있다:

```vim
:Telescope notify      " Telescope로 알림 히스토리 검색
```

```lua
-- nvim-notify 설정 예시
{
  'rcarriga/nvim-notify',
  config = function()
    vim.notify = require('notify')
  end,
}
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

-- 기본 설정만
vim.o.number = true
vim.o.termguicolors = true

-- 의심되는 플러그인만 로드
local lazypath = vim.fn.stdpath('data') .. '/lazy-minimal/lazy.nvim'
if not vim.uv.fs_stat(lazypath) then
  vim.fn.system({
    'git', 'clone', '--filter=blob:none',
    'https://github.com/folke/lazy.nvim.git',
    '--branch=stable', lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require('lazy').setup({
  -- 문제와 관련된 플러그인만 추가
  { 'neovim/nvim-lspconfig' },
}, {
  root = vim.fn.stdpath('data') .. '/lazy-minimal',
})
```

```bash
nvim -u /tmp/minimal.lua
```

이 방법으로:

1. 문제가 재현되면 → 해당 플러그인의 이슈(issue)로 보고
2. 재현되지 않으면 → 다른 플러그인과의 충돌 또는 사용자 설정이 원인

> **팁**: GitHub에 이슈를 올릴 때 최소 재현 설정을 함께 제공하면 유지 관리자(maintainer)가 빠르게 문제를 파악할 수 있다. 대부분의 플러그인 저장소는 이슈 템플릿에 최소 재현을 요구한다.

## Neovim 업데이트 전략

### stable vs nightly

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

### breaking changes 대응

Neovim 메이저 업데이트 시 설정이 깨질 수 있다. 안전한 업데이트 절차:

1. **릴리스 노트(release notes)를 읽는다**: `https://github.com/neovim/neovim/releases`
2. **설정을 백업한다**: `cp -r ~/.config/nvim ~/.config/nvim.bak`
3. **업데이트 후 `:checkhealth`를 실행한다**
4. **플러그인을 업데이트한다**: `:Lazy update`
5. **Treesitter 파서를 재빌드한다**: `:TSUpdate`
6. **문제가 생기면 백업으로 복원한다**

```lua
-- Neovim 버전에 따라 분기하는 설정 예시
if vim.fn.has('nvim-0.11') == 1 then
  -- 0.11 이상에서만 사용 가능한 기능
  vim.lsp.config('lua_ls', { ... })
else
  -- 이전 버전 호환
  require('lspconfig').lua_ls.setup({ ... })
end
```

## 설정 백업과 버전 관리: dotfiles 관리

Neovim 설정은 코드와 마찬가지로 **버전 관리(version control)**해야 한다. 설정을 잃어버리거나, 변경 이력을 추적하고 싶을 때 Git이 가장 좋은 도구다.

### dotfiles 저장소 구성

```bash
# dotfiles 저장소 생성
mkdir ~/dotfiles
cd ~/dotfiles
git init

# Neovim 설정을 심볼릭 링크로 연결
ln -s ~/dotfiles/nvim ~/.config/nvim
```

디렉터리 구조 예시:

```
~/dotfiles/
├── nvim/
│   ├── init.lua
│   ├── lua/
│   │   ├── config/
│   │   │   ├── options.lua
│   │   │   ├── keymaps.lua
│   │   │   └── autocmds.lua
│   │   └── plugins/
│   │       ├── lsp.lua
│   │       ├── telescope.lua
│   │       └── treesitter.lua
│   └── lazy-lock.json
├── .gitignore
└── README.md
```

### lazy-lock.json의 역할

`lazy-lock.json`은 모든 플러그인의 정확한 커밋 해시(commit hash)를 기록한다. 이 파일을 Git에 포함하면:

- 다른 컴퓨터에서 **동일한 플러그인 버전**을 재현할 수 있다
- 플러그인 업데이트 후 문제가 생기면 이전 상태로 **롤백(rollback)**할 수 있다

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

# Neovim 실행 시 lazy.nvim이 자동으로 플러그인 설치
nvim
```

## 실습: 트러블슈팅 연습

다음 진단 도구를 직접 실행해 보자:

**연습 1: 시작 시간 측정**

1. `nvim --startuptime /tmp/startup.log`으로 Neovim을 실행한다
2. Neovim을 종료하고 `/tmp/startup.log`를 열어 총 시작 시간을 확인한다
3. 가장 시간이 오래 걸리는 항목 3개를 찾는다
4. `:Lazy profile`로 플러그인별 로드 시간을 확인한다

**연습 2: 환경 점검**

1. `:checkhealth`를 실행한다
2. WARNING이나 ERROR가 있는지 확인한다
3. 발견된 문제를 해결한다 (대부분 안내 메시지가 해결 방법을 포함한다)

**연습 3: 키맵 진단**

1. `:verbose map <leader>`로 leader 키맵 목록을 확인한다
2. `:Telescope keymaps`로 전체 키맵을 검색한다
3. 사용하지 않는 키맵이 있는지, 충돌하는 키맵이 있는지 점검한다

**연습 4: 최소 설정으로 시작**

1. `nvim --clean`으로 깨끗한 상태에서 Neovim을 시작한다
2. 평소와 다른 동작이 있는지 비교한다
3. `/tmp/minimal.lua`를 작성하고 `nvim -u /tmp/minimal.lua`로 시작해 본다

## 핵심 정리

- **시작 시간 측정**: `nvim --startuptime`과 `:Lazy profile`로 병목 지점을 찾는다
- **지연 로딩**: `cmd`, `keys`, `ft`, `event` 트리거로 플러그인 로드를 지연시켜 시작 시간을 단축한다
- **`:checkhealth`**: 환경 문제의 80%를 진단하는 첫 번째 도구
- **LSP 문제**: `:LspInfo` → `:LspLog` → `:Mason` 순으로 진단한다
- **키맵 충돌**: `:verbose map {key}`로 매핑 출처를 확인한다
- **최소 재현**: `nvim --clean`과 `nvim -u minimal.lua`로 문제 원인을 격리한다
- **업데이트 전략**: stable 채널을 사용하고, 업데이트 전에 설정을 백업하며, 릴리스 노트를 읽는다
- **dotfiles**: Neovim 설정을 Git으로 관리하고, `lazy-lock.json`을 포함하여 플러그인 버전을 고정한다
