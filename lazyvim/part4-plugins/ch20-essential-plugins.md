# Chapter 20: 필수 플러그인 모음 (LazyVim)

Chapter 15에서 19까지 플러그인 매니저, Treesitter, LSP, Telescope, 자동 완성을 다루었다. 이 다섯 가지가 Neovim IDE의 뼈대라면, 이 챕터에서 다루는 플러그인들은 **살과 피부**다. 파일 탐색, Git 연동, 상태 표시줄, 자동 포매팅 등 실전 개발에서 매일 사용하는 기능들이다.

LazyVim에는 이 플러그인들이 **대부분 내장**되어 있다. 설치가 아니라 **사용법과 커스터마이징**에 집중한다.

## LazyVim 내장 플러그인 목록

| 플러그인 | 역할 | 수동 설정 대응 |
|----------|------|----------------|
| neo-tree.nvim | 파일 탐색기 | neo-tree.nvim |
| gitsigns.nvim | Git 변경 표시 | gitsigns.nvim |
| lualine.nvim | 상태 표시줄 | lualine.nvim |
| mini.comment | 주석 토글 | Comment.nvim |
| mini.pairs | 자동 괄호 | nvim-autopairs |
| mini.surround | 감싸기 조작 | nvim-surround |
| conform.nvim | 포매팅 | conform.nvim |
| nvim-lint | 린팅 | nvim-lint |
| which-key.nvim | 키맵 가이드 | which-key.nvim |
| indent-blankline.nvim | 들여쓰기 가이드 | indent-blankline.nvim |
| todo-comments.nvim | TODO 하이라이트 | todo-comments.nvim |
| tokyonight.nvim / catppuccin | 컬러스킴 | 동일 |

## 1. 파일 탐색기: neo-tree.nvim

LazyVim에 기본 포함되어 있다. 설치 불필요.

### 기본 키맵

| 키 | 동작 |
|----|------|
| `<leader>e` | 파일 탐색기 토글 (루트 디렉터리) |
| `<leader>E` | 파일 탐색기 토글 (현재 디렉터리) |
| `<leader>fe` | 파일 탐색기 열기 (루트 디렉터리) |
| `<leader>fE` | 파일 탐색기 열기 (현재 디렉터리) |

### Neo-tree 내 주요 키

| 키 | 동작 |
|----|------|
| `<CR>` 또는 `o` | 파일 열기 / 디렉터리 접기/펼치기 |
| `a` | 새 파일/디렉터리 생성 |
| `d` | 삭제 |
| `r` | 이름 변경 |
| `c` | 복사 |
| `m` | 이동 |
| `y` | 경로 복사 |
| `H` | 숨김 파일 토글 |
| `R` | 새로고침 |
| `/` | 검색 |

> **팁**: 파일 이름 뒤에 `/`를 붙여서 생성하면 디렉터리가 만들어진다. 예를 들어 `a`를 누르고 `components/Button.tsx`를 입력하면 디렉터리와 파일이 동시에 생성된다.

### 커스터마이징

```lua
-- lua/plugins/neo-tree.lua
return {
  {
    "nvim-neo-tree/neo-tree.nvim",
    opts = {
      filesystem = {
        filtered_items = {
          visible = true,        -- 숨김 파일 표시
          hide_dotfiles = false,
          hide_gitignored = false,
        },
      },
      window = {
        width = 35,
      },
    },
  },
}
```

## 2. Git 연동: gitsigns.nvim

LazyVim에 기본 포함. Git으로 관리되는 파일에서 **줄 단위 변경 상태**를 왼쪽 여백에 표시한다.

### 기본 키맵

| 키 | 동작 |
|----|------|
| `]h` | 다음 hunk로 이동 |
| `[h` | 이전 hunk로 이동 |
| `<leader>ghs` | 현재 hunk 스테이징 |
| `<leader>ghr` | 현재 hunk 리셋 |
| `<leader>ghS` | 버퍼 전체 스테이징 |
| `<leader>ghR` | 버퍼 전체 리셋 |
| `<leader>ghp` | hunk 미리보기 |
| `<leader>ghb` | 줄 blame 표시 |
| `<leader>ghd` | diff 보기 |
| `<leader>ub` | 인라인 blame 토글 |

hunk란 Git diff에서 연속적으로 변경된 줄의 묶음이다. `]h`와 `[h`로 hunk 사이를 오가면서 `<leader>ghs`로 원하는 변경만 선택적으로 스테이징할 수 있다.

### 커스터마이징

```lua
-- lua/plugins/gitsigns.lua
return {
  {
    "lewis6991/gitsigns.nvim",
    opts = {
      signs = {
        add          = { text = "│" },
        change       = { text = "│" },
        delete       = { text = "_" },
        topdelete    = { text = "‾" },
        changedelete = { text = "~" },
      },
      current_line_blame = true,  -- 항상 인라인 blame 표시
    },
  },
}
```

## 3. 상태 표시줄: lualine.nvim

LazyVim에 기본 포함. 모드, 브랜치, diff, 진단, 파일 정보 등을 하단에 표시한다.

### 상태 표시줄 구조

```
┌──────┬──────────────────┬──────────────────────────────────┬────────────┬──────┬──────┐
│ mode │ branch diff diag │ filename                         │ enc ft     │ prog │ loc  │
│  (a) │       (b)        │         (c)                      │    (x)     │ (y)  │ (z)  │
└──────┴──────────────────┴──────────────────────────────────┴────────────┴──────┴──────┘
```

### 커스터마이징

```lua
-- lua/plugins/lualine.lua
return {
  {
    "nvim-lualine/lualine.nvim",
    opts = {
      options = {
        theme = "auto",
        component_separators = { left = "", right = "" },
        section_separators = { left = "", right = "" },
      },
      sections = {
        lualine_c = { { "filename", path = 1 } },  -- 상대 경로 표시
      },
    },
  },
}
```

`path = 1`로 설정하면 파일 이름 대신 상대 경로가 표시된다. 프로젝트 내에서 같은 이름의 파일이 여러 개일 때 유용하다.

## 4. 주석: mini.comment

LazyVim은 Comment.nvim 대신 **mini.comment**를 사용한다. 사용법은 거의 동일하다.

### 주요 키맵

| 키 | 동작 |
|----|------|
| `gcc` | 현재 줄 주석 토글 |
| `gc{motion}` | 모션 범위 주석 토글 |
| `gcip` | 현재 단락 주석 토글 |
| `gc5j` | 현재 줄부터 아래 5줄 주석 토글 |

Visual 모드에서:

| 키 | 동작 |
|----|------|
| `gc` | 선택 영역 주석 토글 |

> **팁**: `gc`는 Vim의 편집 언어와 자연스럽게 결합된다. Chapter 1에서 배운 "동사 + 명사" 구조 그대로 `gc`(주석) + `ip`(단락) = `gcip`(단락 주석)처럼 사용할 수 있다.

## 5. 자동 괄호: mini.pairs

LazyVim은 nvim-autopairs 대신 **mini.pairs**를 사용한다. 여는 괄호를 입력하면 닫는 괄호가 자동으로 추가된다.

`(`, `{`, `[`, `"`, `'` 등을 입력하면 짝이 자동 완성된다. 별도 설정 없이 바로 동작한다.

### 비활성화

자동 괄호를 끄고 싶다면:

```lua
-- lua/plugins/mini-pairs.lua
return {
  { "echasnovski/mini.pairs", enabled = false },
}
```

## 6. Surround: mini.surround

LazyVim은 nvim-surround 대신 **mini.surround**를 사용한다. 키맵이 약간 다르다.

### 주요 키맵

| 명령 | 동작 | nvim-surround 대응 |
|------|------|---------------------|
| `gsa{char}` | 감싸기 추가 (Add) | `ys{motion}{char}` |
| `gsd{char}` | 감싸기 삭제 (Delete) | `ds{char}` |
| `gsr{old}{new}` | 감싸기 변경 (Replace) | `cs{old}{new}` |
| `gsf` | 커서 주변 감싸기 찾기 (Find right) | -- |
| `gsF` | 커서 주변 감싸기 찾기 (Find left) | -- |
| `gsh` | 감싸기 하이라이트 | -- |

### 사용 예시

| 명령 | 결과 |
|------|------|
| `gsaiw"` | `hello` -> `"hello"` |
| `gsaiw)` | `hello` -> `(hello)` |
| `gsd"` | `"hello"` -> `hello` |
| `gsr"'` | `"hello"` -> `'hello'` |

> **팁**: mini.surround는 `gs` 접두사를 사용한다. nvim-surround의 `ys`/`ds`/`cs`와 다르니 주의한다.

### nvim-surround로 교체

nvim-surround의 키맵(`ys`, `ds`, `cs`)이 익숙하다면 교체할 수 있다:

```lua
-- lua/plugins/surround.lua
return {
  -- mini.surround 비활성화
  { "echasnovski/mini.surround", enabled = false },

  -- nvim-surround 활성화
  {
    "kylechui/nvim-surround",
    version = "*",
    event = "VeryLazy",
    opts = {},
  },
}
```

## 7. 포매팅: conform.nvim

LazyVim에 기본 포함. 저장 시 자동 포매팅이 기본 활성화되어 있다.

### 기본 키맵

| 키 | 동작 |
|----|------|
| `<leader>cf` | 포매팅 |
| `<leader>uf` | 자동 포매팅 토글 (현재 버퍼) |
| `<leader>uF` | 자동 포매팅 토글 (전역) |

### 포매터 설정

```lua
-- lua/plugins/formatting.lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        lua = { "stylua" },
        javascript = { "prettier" },
        typescript = { "prettier" },
        javascriptreact = { "prettier" },
        typescriptreact = { "prettier" },
        css = { "prettier" },
        html = { "prettier" },
        json = { "prettier" },
        yaml = { "prettier" },
        markdown = { "prettier" },
        python = { "black" },
        go = { "gofmt" },
        rust = { "rustfmt" },
      },
    },
  },
}
```

> **팁**: 포매터는 별도로 설치해야 한다. `npm install -g prettier`, `pip install black`, `cargo install stylua` 등으로 시스템에 설치하거나, mason의 `ensure_installed`에 추가한다.

### 포매터 정보 확인

```vim
:ConformInfo     " 현재 버퍼에 적용되는 포매터 확인
:LazyFormatInfo  " LazyVim의 포매팅 설정 확인
```

## 8. 린팅: nvim-lint

LazyVim에 기본 포함. 코드 린터를 실행하여 잠재적 오류와 스타일 문제를 알려준다.

### 린터 설정

```lua
-- lua/plugins/linting.lua
return {
  {
    "mfussenegger/nvim-lint",
    opts = {
      linters_by_ft = {
        javascript = { "eslint_d" },
        typescript = { "eslint_d" },
        javascriptreact = { "eslint_d" },
        typescriptreact = { "eslint_d" },
        python = { "pylint" },
      },
    },
  },
}
```

린팅 결과는 LSP 진단과 동일한 방식으로 표시된다. `]d`와 `[d`로 진단 항목 사이를 이동할 수 있다.

## 9. 컬러스킴

LazyVim은 **tokyonight**를 기본 컬러스킴으로 사용한다. catppuccin도 extras로 제공된다.

### 기본 테마 변경

```lua
-- lua/plugins/colorscheme.lua
return {
  -- 다른 테마 설치
  {
    "catppuccin/nvim",
    name = "catppuccin",
    lazy = true,
    opts = {
      flavour = "mocha",  -- "latte", "frappe", "macchiato", "mocha"
    },
  },

  -- LazyVim 기본 컬러스킴 변경
  {
    "LazyVim/LazyVim",
    opts = {
      colorscheme = "catppuccin",
    },
  },
}
```

### tokyonight 커스터마이징

```lua
return {
  {
    "folke/tokyonight.nvim",
    opts = {
      style = "night",        -- "storm", "moon", "night", "day"
      transparent = false,
    },
  },
}
```

### 기타 인기 테마

| 테마 | 특징 |
|------|------|
| `rose-pine/neovim` | 부드러운 파스텔 톤 |
| `rebelot/kanagawa.nvim` | 일본 전통 색상 기반 |
| `EdenEast/nightfox.nvim` | 다양한 변형 제공 |
| `navarasu/onedark.nvim` | One Dark Pro 기반 |
| `sainnhe/gruvbox-material` | Gruvbox의 부드러운 변형 |

## 10. which-key.nvim: 키맵 가이드

LazyVim에 기본 포함. `<leader>`를 누르고 잠시 기다리면 사용 가능한 후속 키맵이 팝업으로 표시된다.

LazyVim은 키 그룹을 이미 체계적으로 등록해 놓았다:

```
┌──────────────────────────────────────┐
│  f  Find/File                        │
│  g  Git                              │
│  c  Code                             │
│  s  Search                           │
│  u  UI/Toggle                        │
│  b  Buffer                           │
│  w  Windows                          │
│  x  Diagnostics/Quickfix             │
│  ...                                 │
└──────────────────────────────────────┘
```

### 추가 그룹 등록

```lua
-- lua/plugins/which-key.lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      spec = {
        { "<leader>m", group = "내 커스텀 그룹" },
      },
    },
  },
}
```

## 11. indent-blankline.nvim: 들여쓰기 가이드라인

LazyVim에 기본 포함. 들여쓰기 수준을 세로 선으로 표시한다.

### 커스터마이징

```lua
-- lua/plugins/indent-blankline.lua
return {
  {
    "lukas-reineke/indent-blankline.nvim",
    opts = {
      indent = {
        char = "│",
      },
      scope = {
        enabled = true,
        show_start = true,
        show_end = false,
      },
    },
  },
}
```

`scope`는 현재 커서가 위치한 코드 블록의 들여쓰기 선을 강조 표시한다. Treesitter와 연동되어 정확한 범위를 파악한다.

## 12. todo-comments.nvim: TODO/FIXME 하이라이트

LazyVim에 기본 포함. 코드 내의 `TODO`, `FIXME`, `HACK`, `NOTE` 등을 하이라이트한다.

### 지원하는 키워드

| 키워드 | 색상 | 용도 |
|--------|------|------|
| `TODO` | 파랑 | 해야 할 작업 |
| `FIXME` | 빨강 | 수정이 필요한 버그 |
| `HACK` | 주황 | 임시 해결책 |
| `WARN` / `WARNING` | 노랑 | 주의 사항 |
| `NOTE` / `INFO` | 초록 | 참고 정보 |
| `PERF` / `OPTIM` | 보라 | 성능 관련 |
| `TEST` | 보라 | 테스트 관련 |

### 기본 키맵

| 키 | 동작 |
|----|------|
| `]t` | 다음 TODO로 이동 |
| `[t` | 이전 TODO로 이동 |
| `<leader>st` | Telescope로 TODO 검색 |
| `<leader>sT` | Telescope로 TODO/FIXME/HACK 검색 |
| `<leader>xt` | TODO 목록 (Trouble) |
| `<leader>xT` | TODO/FIXME/HACK 목록 (Trouble) |

## 13. LazyVim UI 토글 키맵

LazyVim은 `<leader>u` 접두사로 다양한 UI 토글 키맵을 제공한다:

| 키 | 토글 대상 |
|----|-----------|
| `<leader>uf` | 자동 포매팅 (버퍼) |
| `<leader>uF` | 자동 포매팅 (전역) |
| `<leader>us` | 맞춤법 검사 |
| `<leader>uw` | 줄 바꿈 (wrap) |
| `<leader>uL` | 상대 줄 번호 |
| `<leader>ul` | 줄 번호 |
| `<leader>ud` | 진단 표시 |
| `<leader>uc` | Conceal |
| `<leader>ub` | 인라인 blame |
| `<leader>ui` | Treesitter 하이라이트 |

## 설정 파일 구조

LazyVim에서 플러그인을 커스터마이징할 때의 파일 구조:

```
~/.config/nvim/
├── init.lua
└── lua/
    ├── config/
    │   ├── lazy.lua        -- lazy.nvim 설정, extras import
    │   ├── keymaps.lua     -- 사용자 키맵
    │   └── options.lua     -- 사용자 옵션
    └── plugins/
        ├── colorscheme.lua -- 컬러스킴 커스터마이징
        ├── lsp.lua         -- LSP 서버 추가/설정
        ├── formatting.lua  -- 포매터 설정
        ├── neo-tree.lua    -- Neo-tree 커스터마이징
        └── ...             -- 추가 플러그인/설정
```

`lua/plugins/` 디렉터리의 모든 `.lua` 파일이 자동으로 로드된다. 각 파일에서 기존 플러그인의 `opts`를 오버라이드하거나 새 플러그인을 추가할 수 있다.

## 실습: 플러그인 활용

1. `<leader>e`로 Neo-tree를 열고, `a`로 새 파일을 만들어 보자
2. 파일을 수정한 뒤 gitsigns가 왼쪽 여백에 변경 표시를 하는지 확인하자
3. `gcc`로 코드 한 줄을 주석 처리하고, 다시 `gcc`로 해제해 보자
4. `(`, `"`, `{`를 입력하여 mini.pairs가 닫는 괄호를 자동 추가하는지 확인하자
5. `gsaiw"`로 단어를 따옴표로 감싸고, `gsr"'`로 큰따옴표를 작은따옴표로 변경해 보자
6. `<leader>`를 누르고 잠시 기다려 which-key 팝업을 확인하자
7. `TODO: 테스트`라고 주석을 작성하여 하이라이트가 적용되는지 확인하자
8. `<leader>st`로 프로젝트 전체의 TODO를 검색해 보자
9. 파일을 저장하여 자동 포매팅이 동작하는지 확인하자
10. `<leader>uf`로 자동 포매팅을 토글해 보자

## 핵심 정리

- LazyVim에는 실전 개발에 필요한 플러그인이 **대부분 내장**되어 있다
- **Neo-tree**: `<leader>e`로 토글, `a`(생성), `d`(삭제), `r`(이름 변경)
- **gitsigns**: `]h`/`[h`로 hunk 이동, `<leader>ghs`로 스테이징, `<leader>ub`로 blame 토글
- **lualine**: 상태 표시줄, 모드/브랜치/진단 정보를 한눈에 표시
- **mini.comment**: `gcc`로 줄 주석 토글, `gc{motion}`으로 범위 주석
- **mini.pairs**: 괄호/따옴표 자동 짝 맞추기 (기본 활성화)
- **mini.surround**: `gsa`(추가), `gsd`(삭제), `gsr`(변경) -- `gs` 접두사 사용
- **conform.nvim**: `<leader>cf`로 포매팅, `<leader>uf`로 자동 포매팅 토글
- **nvim-lint**: 린터 실행, 진단과 동일하게 표시
- **which-key**: `<leader>` 후 키맵 가이드, 그룹이 체계적으로 정리됨
- **todo-comments**: `]t`/`[t`로 이동, `<leader>st`로 Telescope 검색
- **`<leader>u` 접두사**로 다양한 UI 토글 사용 가능
- 커스터마이징은 `lua/plugins/` 디렉터리에 파일을 만들고 **`opts` 오버라이드**로 한다
