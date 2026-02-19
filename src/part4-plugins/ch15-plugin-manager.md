# Chapter 15: 플러그인 매니저 - lazy.nvim

Neovim의 진정한 힘은 플러그인 생태계에서 나온다. LSP, Treesitter, 퍼지 파인더(fuzzy finder) 등 현대적인 개발 환경을 구축하려면 수십 개의 플러그인이 필요하다. 이 플러그인들을 효율적으로 설치, 업데이트, 관리하는 것이 플러그인 매니저의 역할이다.

## 왜 플러그인 매니저가 필요한가

플러그인을 수동으로 관리한다고 생각해 보자:

1. GitHub에서 플러그인 저장소를 클론한다
2. Neovim의 런타임 경로(runtime path)에 추가한다
3. 업데이트할 때마다 각 저장소에서 `git pull`을 실행한다
4. 더 이상 필요 없는 플러그인은 직접 삭제한다
5. 플러그인 간의 의존성(dependencies)을 직접 파악해야 한다

플러그인이 5개일 때는 가능하다. 하지만 20개, 30개가 되면 이 과정은 악몽이 된다. 플러그인 매니저는 이 모든 과정을 자동화하고, 추가로 **지연 로딩(lazy loading)**까지 제공하여 Neovim의 시작 시간을 최소화한다.

## lazy.nvim 소개

**lazy.nvim**은 현재 Neovim 생태계에서 가장 널리 사용되는 플러그인 매니저다. folke가 개발했으며, 다음과 같은 특징을 가진다:

- **자동 지연 로딩**: 플러그인을 필요할 때만 로드하여 시작 시간을 최소화
- **락파일(lockfile)**: `lazy-lock.json`으로 플러그인 버전을 고정
- **직관적인 UI**: 설치, 업데이트, 프로파일링을 시각적으로 확인
- **병렬 설치/업데이트**: 여러 플러그인을 동시에 처리
- **플러그인 명세(spec) 시스템**: 선언적으로 플러그인을 정의

이전에 사용되던 packer.nvim, vim-plug 등과 비교하면, lazy.nvim은 성능과 사용성 모두에서 한 단계 앞서 있다.

## 설치: Bootstrap 코드

lazy.nvim은 별도의 설치 과정 없이 `init.lua`에 부트스트랩(bootstrap) 코드를 추가하면 된다. Neovim을 처음 실행할 때 자동으로 설치된다.

`~/.config/nvim/init.lua`에 다음 코드를 추가한다:

```lua
-- lazy.nvim 부트스트랩
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

이 코드가 하는 일을 한 줄씩 살펴보자:

| 코드 | 설명 |
|------|------|
| `vim.fn.stdpath("data")` | Neovim 데이터 디렉토리 경로 반환 (`~/.local/share/nvim`) |
| `vim.loop.fs_stat(lazypath)` | 해당 경로가 존재하는지 확인 |
| `vim.fn.system({...})` | 존재하지 않으면 git clone 실행 |
| `--filter=blob:none` | 불필요한 blob 데이터 없이 클론 (빠른 다운로드) |
| `--branch=stable` | 안정 브랜치 사용 |
| `vim.opt.rtp:prepend(lazypath)` | 런타임 경로 맨 앞에 추가 |

부트스트랩 코드 다음에 `require("lazy").setup()`을 호출하여 플러그인 목록을 전달한다:

```lua
-- 기본 설정 (옵션 등은 이 앞에 배치)
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- lazy.nvim 부트스트랩 (위 코드)
-- ...

-- 플러그인 설정
require("lazy").setup({
  -- 플러그인 명세들이 여기에 들어간다
})
```

> **중요**: `vim.g.mapleader`는 반드시 `require("lazy").setup()` **이전에** 설정해야 한다. 플러그인들이 leader 키를 참조하기 때문이다.

## 플러그인 명세(Spec) 작성법

lazy.nvim에서 플러그인은 Lua 테이블로 정의한다. 이것을 플러그인 명세(spec)라고 한다.

### 가장 단순한 형태

```lua
{ "GitHub사용자/저장소이름" }
```

예를 들어:

```lua
require("lazy").setup({
  { "tpope/vim-sleuth" },           -- 들여쓰기 자동 감지
  { "numToStr/Comment.nvim" },      -- 주석 토글
})
```

### opts: 플러그인 옵션 전달

많은 플러그인은 `setup()` 함수를 제공한다. `opts`를 지정하면 lazy.nvim이 자동으로 `require("플러그인").setup(opts)`를 호출한다.

```lua
{
  "numToStr/Comment.nvim",
  opts = {
    -- 여기에 플러그인 옵션을 넣는다
    toggler = {
      line = "gcc",    -- 줄 주석 토글
      block = "gbc",   -- 블록 주석 토글
    },
  },
}
```

`opts = {}`는 기본 옵션으로 setup을 호출하라는 의미다:

```lua
{ "numToStr/Comment.nvim", opts = {} }
-- 이것은 다음과 동일하다:
-- require("Comment").setup({})
```

### config: 직접 설정 함수 작성

`opts`만으로 부족할 때, `config` 함수를 사용하면 플러그인 로드 후 원하는 코드를 실행할 수 있다.

```lua
{
  "folke/tokyonight.nvim",
  config = function()
    require("tokyonight").setup({
      style = "night",
    })
    -- setup 이후 추가 작업
    vim.cmd.colorscheme("tokyonight")
  end,
}
```

> **팁**: `opts`와 `config`를 함께 사용할 수도 있다. 이 경우 `config` 함수의 두 번째 인자로 병합된 opts가 전달된다:
> ```lua
> {
>   "플러그인",
>   opts = { ... },
>   config = function(_, opts)
>     require("플러그인").setup(opts)
>     -- 추가 작업
>   end,
> }
> ```

### dependencies: 의존성 선언

플러그인이 다른 플러그인에 의존할 때 사용한다. 의존 플러그인이 먼저 로드된다.

```lua
{
  "nvim-telescope/telescope.nvim",
  dependencies = {
    "nvim-lua/plenary.nvim",           -- 필수 라이브러리
    "nvim-tree/nvim-web-devicons",     -- 파일 아이콘
  },
}
```

### 지연 로딩 트리거: event, ft, keys, cmd

지연 로딩은 lazy.nvim의 핵심 기능이다. 플러그인을 시작 시 로드하지 않고, 특정 조건이 충족될 때만 로드한다.

```lua
-- event: 특정 이벤트 발생 시 로드
{
  "lewis6991/gitsigns.nvim",
  event = { "BufReadPre", "BufNewFile" },   -- 파일을 열 때 로드
  opts = {},
}

-- ft: 특정 파일타입에서만 로드
{
  "ray-x/go.nvim",
  ft = { "go", "gomod" },                    -- Go 파일에서만 로드
  opts = {},
}

-- keys: 특정 키를 누를 때 로드
{
  "folke/flash.nvim",
  keys = {
    { "s", mode = { "n", "x", "o" }, function() require("flash").jump() end, desc = "Flash" },
  },
}

-- cmd: 특정 명령 실행 시 로드
{
  "stevearc/oil.nvim",
  cmd = "Oil",                               -- :Oil 명령 시 로드
  opts = {},
}
```

자주 사용하는 이벤트:

| 이벤트 | 시점 |
|--------|------|
| `BufReadPre` | 파일을 읽기 직전 |
| `BufNewFile` | 새 파일을 만들 때 |
| `VeryLazy` | Neovim UI가 완전히 로드된 후 |
| `InsertEnter` | Insert 모드 진입 시 |
| `CmdlineEnter` | 명령줄 진입 시 |

## 지연 로딩(Lazy Loading) 전략

### 왜 지연 로딩이 중요한가

30개의 플러그인을 시작 시 모두 로드하면 Neovim 시작에 200~500ms가 걸릴 수 있다. 지연 로딩을 적용하면 시작 시에는 5~10개만 로드하고, 나머지는 필요할 때 로드한다. 결과적으로 시작 시간을 50ms 이하로 줄일 수 있다.

### 전략별 가이드

**이벤트 기반(Event-based)**:
```lua
-- 파일을 열 때 로드 (가장 흔한 패턴)
event = { "BufReadPre", "BufNewFile" }

-- UI가 안정된 후 로드 (급하지 않은 플러그인)
event = "VeryLazy"

-- Insert 모드 진입 시 로드 (자동 완성 등)
event = "InsertEnter"
```

**파일타입 기반(Filetype-based)**:
```lua
-- 특정 언어 전용 플러그인
ft = "python"
ft = { "javascript", "typescript", "typescriptreact" }
```

**키 기반(Key-based)**:
```lua
-- 특정 키맵으로만 사용하는 플러그인
keys = {
  { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "파일 찾기" },
  { "<leader>fg", "<cmd>Telescope live_grep<cr>", desc = "텍스트 검색" },
}
```

**명령 기반(Command-based)**:
```lua
-- 명령어로만 사용하는 플러그인
cmd = { "Telescope", "TelescopeResume" }
```

> **팁**: 확실하지 않을 때는 `event = "VeryLazy"`가 안전한 선택이다. 시작 시간에 영향을 주지 않으면서 대부분의 상황에서 잘 동작한다.

## lazy.nvim UI 사용법

lazy.nvim은 강력한 내장 UI를 제공한다.

### UI 열기

```vim
:Lazy
```

이 명령 하나로 lazy.nvim 대시보드가 열린다.

### 주요 메뉴

| 키 | 동작 |
|----|------|
| `I` | 설치 (Install) - 아직 설치되지 않은 플러그인 설치 |
| `U` | 업데이트 (Update) - 모든 플러그인을 최신 버전으로 |
| `S` | 동기화 (Sync) - 설치 + 업데이트 + 정리를 한 번에 |
| `X` | 정리 (Clean) - 명세에서 제거된 플러그인 삭제 |
| `C` | 확인 (Check) - 업데이트 가능 여부 확인 |
| `P` | 프로파일 (Profile) - 플러그인별 로딩 시간 확인 |
| `R` | 복원 (Restore) - 락파일 기준으로 버전 복원 |
| `L` | 로그 (Log) - git 변경 이력 확인 |

### 프로파일 활용

`:Lazy`에서 `P`를 누르면 각 플러그인의 로딩 시간을 확인할 수 있다. 시작이 느리다고 느껴질 때 어떤 플러그인이 병목인지 파악하는 데 유용하다.

```
플러그인 A      0.5ms   ████
플러그인 B      1.2ms   █████████
플러그인 C      0.1ms   █
총 시작 시간:   35ms
```

## 플러그인 관리 워크플로우

### 플러그인 추가

1. 사용할 플러그인의 GitHub 저장소를 찾는다
2. 플러그인 명세를 작성하여 설정 파일에 추가한다
3. Neovim을 재시작하거나 `:Lazy`에서 `I`를 누른다

### 플러그인 설정 변경

1. 해당 플러그인의 명세에서 `opts`나 `config`를 수정한다
2. Neovim을 재시작한다 (또는 `:Lazy` > `R`로 리로드)

### 플러그인 업데이트

```vim
:Lazy update
```

또는 `:Lazy` UI에서 `U`를 누른다. 업데이트 후 문제가 생기면 `lazy-lock.json`을 기반으로 `:Lazy restore`로 이전 버전으로 되돌릴 수 있다.

### 플러그인 제거

1. 설정 파일에서 해당 플러그인 명세를 삭제한다
2. `:Lazy clean` 또는 `:Lazy` UI에서 `X`를 눌러 파일을 정리한다

## 플러그인 설정 파일 구조화

플러그인이 늘어나면 `init.lua` 하나에 모든 설정을 담기 어렵다. lazy.nvim은 디렉토리 기반 자동 로드를 지원한다.

### 권장 디렉토리 구조

```
~/.config/nvim/
├── init.lua                    -- 부트스트랩 + require("lazy").setup()
└── lua/
    └── plugins/
        ├── colorscheme.lua     -- 컬러스킴 설정
        ├── treesitter.lua      -- Treesitter 설정
        ├── lsp.lua             -- LSP 설정
        ├── completion.lua      -- 자동 완성 설정
        ├── telescope.lua       -- Telescope 설정
        ├── editor.lua          -- 편집 관련 플러그인
        └── ui.lua              -- UI 관련 플러그인
```

### init.lua에서 디렉토리 지정

```lua
require("lazy").setup("plugins")
```

이렇게 하면 `lua/plugins/` 디렉토리 안의 모든 Lua 파일을 자동으로 로드한다. 각 파일은 플러그인 명세 테이블을 반환해야 한다.

### 개별 플러그인 파일 예시

`lua/plugins/colorscheme.lua`:

```lua
return {
  {
    "folke/tokyonight.nvim",
    lazy = false,       -- 컬러스킴은 즉시 로드
    priority = 1000,    -- 다른 플러그인보다 먼저 로드
    config = function()
      require("tokyonight").setup({
        style = "night",
      })
      vim.cmd.colorscheme("tokyonight")
    end,
  },
}
```

`lua/plugins/editor.lua`:

```lua
return {
  -- 자동 들여쓰기 감지
  { "tpope/vim-sleuth" },

  -- 주석 토글
  {
    "numToStr/Comment.nvim",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- surround 편집
  {
    "kylechui/nvim-surround",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },
}
```

> **팁**: 파일 하나에 관련 플러그인 여러 개를 묶어도 되고, 복잡한 플러그인은 파일 하나에 하나만 넣어도 된다. 자신이 찾기 편한 방식을 선택한다.

## 실전 예제: 기본 플러그인 세트 설정

처음 시작하는 사람을 위한 기본 설정을 구성해 보자. 전체 디렉토리 구조부터 시작한다.

### init.lua

```lua
-- leader 키 설정
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- lazy.nvim 부트스트랩
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- 플러그인 로드
require("lazy").setup("plugins")
```

### lua/plugins/colorscheme.lua

```lua
return {
  {
    "folke/tokyonight.nvim",
    lazy = false,
    priority = 1000,
    config = function()
      require("tokyonight").setup({ style = "night" })
      vim.cmd.colorscheme("tokyonight")
    end,
  },
}
```

### lua/plugins/editor.lua

```lua
return {
  -- 들여쓰기 자동 감지
  { "tpope/vim-sleuth" },

  -- 주석 토글 (gcc, gbc)
  {
    "numToStr/Comment.nvim",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- surround 조작 (ys, ds, cs)
  {
    "kylechui/nvim-surround",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- 자동 괄호 쌍
  {
    "windwp/nvim-autopairs",
    event = "InsertEnter",
    opts = {},
  },
}
```

### lua/plugins/ui.lua

```lua
return {
  -- 상태줄
  {
    "nvim-lualine/lualine.nvim",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    event = "VeryLazy",
    opts = {
      options = {
        theme = "tokyonight",
      },
    },
  },

  -- 들여쓰기 가이드라인
  {
    "lukas-reineke/indent-blankline.nvim",
    main = "ibl",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- Git 표시
  {
    "lewis6991/gitsigns.nvim",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },
}
```

이 설정으로 Neovim을 시작하면 lazy.nvim이 자동으로 모든 플러그인을 설치한다. `:Lazy`를 실행하여 상태를 확인할 수 있다.

## 실습: lazy.nvim 설정하기

1. `~/.config/nvim/init.lua`에 부트스트랩 코드를 추가하고 Neovim을 재시작한다
2. `require("lazy").setup({ "folke/tokyonight.nvim" })`으로 첫 번째 플러그인을 추가한다
3. `:Lazy`를 실행하여 UI를 탐색한다
4. `lua/plugins/` 디렉토리를 만들고, 파일별로 플러그인을 분리한다
5. `:Lazy profile`에서 시작 시간을 확인한다
6. `event = "VeryLazy"`를 추가하기 전후의 시작 시간을 비교한다

## 핵심 정리

- **플러그인 매니저**는 플러그인의 설치, 업데이트, 삭제, 의존성 관리를 자동화한다
- **lazy.nvim**은 현대적이고 빠른 Neovim 플러그인 매니저다
- **부트스트랩 코드**로 lazy.nvim 자체를 자동 설치할 수 있다
- **플러그인 명세(spec)**는 Lua 테이블로 작성하며, `opts`, `config`, `dependencies` 등을 지정한다
- **지연 로딩**으로 시작 시간을 극적으로 줄인다: `event`, `ft`, `keys`, `cmd`
- **`:Lazy`** UI로 설치, 업데이트, 정리, 프로파일링을 시각적으로 수행한다
- **`lua/plugins/`** 디렉토리에 파일별로 분리하면 설정을 체계적으로 관리할 수 있다
- `vim.g.mapleader`는 반드시 `require("lazy").setup()` **이전에** 설정한다
