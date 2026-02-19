# Chapter 20: 필수 플러그인 모음

Chapter 15에서 19까지 플러그인 매니저, Treesitter, LSP, Telescope, 자동 완성을 다루었다. 이 다섯 가지가 Neovim IDE의 뼈대라면, 이 챕터에서 다루는 플러그인들은 **살과 피부**다. 파일 탐색, Git 연동, 상태 표시줄, 자동 포매팅 등 실전 개발에서 매일 사용하는 기능들을 빠르게 설정한다.

각 플러그인은 **소개 - 설치(lazy.nvim spec) - 핵심 설정 - 주요 키맵/사용법** 순서로 설명한다.

## 1. 파일 탐색기: neo-tree.nvim

### 소개

neo-tree.nvim은 Neovim 안에서 파일 시스템을 트리 형태로 탐색할 수 있는 플러그인이다. VS Code의 사이드바와 비슷한 역할을 한다.

### 설치

```lua
-- plugins/neo-tree.lua
return {
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  config = function()
    require("neo-tree").setup({
      close_if_last_window = true,
      filesystem = {
        follow_current_file = {
          enabled = true,
        },
        filtered_items = {
          visible = true,
          hide_dotfiles = false,
          hide_gitignored = false,
        },
      },
      window = {
        width = 35,
        mappings = {
          ["<space>"] = "none",  -- leader 키 충돌 방지
        },
      },
    })
  end,
}
```

### 주요 키맵

```lua
vim.keymap.set("n", "<leader>e", "<cmd>Neotree toggle<cr>", { desc = "파일 탐색기 토글" })
vim.keymap.set("n", "<leader>o", "<cmd>Neotree focus<cr>", { desc = "파일 탐색기 포커스" })
```

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

## 2. Git 연동: gitsigns.nvim

### 소개

gitsigns.nvim은 Git으로 관리되는 파일에서 **줄 단위 변경 상태**를 표시해준다. 에디터의 왼쪽 여백(sign column)에 추가/수정/삭제된 줄을 색상으로 표시하고, hunk 단위 작업(스테이징, 되돌리기, 미리보기)을 할 수 있다.

### 설치

```lua
-- plugins/gitsigns.lua
return {
  "lewis6991/gitsigns.nvim",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    require("gitsigns").setup({
      signs = {
        add          = { text = "│" },
        change       = { text = "│" },
        delete       = { text = "_" },
        topdelete    = { text = "‾" },
        changedelete = { text = "~" },
      },
      on_attach = function(bufnr)
        local gs = package.loaded.gitsigns

        local function map(mode, l, r, opts)
          opts = opts or {}
          opts.buffer = bufnr
          vim.keymap.set(mode, l, r, opts)
        end

        -- Hunk 이동
        map("n", "]h", gs.next_hunk, { desc = "다음 hunk" })
        map("n", "[h", gs.prev_hunk, { desc = "이전 hunk" })

        -- Hunk 작업
        map("n", "<leader>hs", gs.stage_hunk, { desc = "Hunk 스테이지" })
        map("n", "<leader>hr", gs.reset_hunk, { desc = "Hunk 리셋" })
        map("n", "<leader>hp", gs.preview_hunk, { desc = "Hunk 미리보기" })
        map("n", "<leader>hb", function() gs.blame_line({ full = true }) end, { desc = "줄 blame" })
        map("n", "<leader>hd", gs.diffthis, { desc = "Diff 보기" })

        -- 토글
        map("n", "<leader>tb", gs.toggle_current_line_blame, { desc = "줄 blame 토글" })
      end,
    })
  end,
}
```

### 핵심 사용법

| 키 | 동작 |
|----|------|
| `]h` / `[h` | 다음/이전 변경 블록(hunk)으로 이동 |
| `<leader>hs` | 현재 hunk를 Git 스테이징 |
| `<leader>hr` | 현재 hunk의 변경 되돌리기 |
| `<leader>hp` | 현재 hunk의 변경 내용 미리보기 |
| `<leader>hb` | 현재 줄의 Git blame 표시 |
| `<leader>hd` | 현재 파일의 diff 표시 |
| `<leader>tb` | 인라인 blame 토글 |

hunk란 Git diff에서 연속적으로 변경된 줄의 묶음이다. `]h`와 `[h`로 hunk 사이를 오가면서 `<leader>hs`로 원하는 변경만 선택적으로 스테이징할 수 있다.

## 3. 상태 표시줄: lualine.nvim

### 소개

lualine.nvim은 화면 하단의 상태 표시줄(statusline)을 아름답고 정보적으로 바꿔준다. 현재 모드, 파일 이름, Git 브랜치, 진단(diagnostics) 정보 등을 표시한다.

### 설치

```lua
-- plugins/lualine.lua
return {
  "nvim-lualine/lualine.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  config = function()
    require("lualine").setup({
      options = {
        theme = "auto",
        component_separators = { left = "", right = "" },
        section_separators = { left = "", right = "" },
      },
      sections = {
        lualine_a = { "mode" },
        lualine_b = { "branch", "diff", "diagnostics" },
        lualine_c = { { "filename", path = 1 } },
        lualine_x = { "encoding", "fileformat", "filetype" },
        lualine_y = { "progress" },
        lualine_z = { "location" },
      },
    })
  end,
}
```

### 상태 표시줄 구조

```
┌──────┬──────────────────┬──────────────────────────────────┬────────────┬──────┬──────┐
│ mode │ branch diff diag │ filename                         │ enc ft     │ prog │ loc  │
│  (a) │       (b)        │         (c)                      │    (x)     │ (y)  │ (z)  │
└──────┴──────────────────┴──────────────────────────────────┴────────────┴──────┴──────┘
```

`path = 1`로 설정하면 파일 이름 대신 상대 경로가 표시된다. 프로젝트 내에서 같은 이름의 파일이 여러 개일 때 유용하다.

## 4. 주석: Comment.nvim

### 소개

Comment.nvim은 코드 주석을 빠르게 토글하는 플러그인이다. 한 줄 주석, 블록 주석을 간단한 키 조합으로 처리한다.

### 설치

```lua
-- plugins/comment.lua
return {
  "numToStr/Comment.nvim",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    require("Comment").setup()
  end,
}
```

별도의 설정 없이 기본값만으로 충분하다.

### 주요 키맵

| 키 | 동작 |
|----|------|
| `gcc` | 현재 줄 주석 토글 |
| `gc{motion}` | 모션 범위 주석 토글 |
| `gcip` | 현재 단락 주석 토글 |
| `gc5j` | 현재 줄부터 아래 5줄 주석 토글 |
| `gbc` | 현재 줄 블록 주석 토글 |
| `gb{motion}` | 모션 범위 블록 주석 토글 |

Visual 모드에서:

| 키 | 동작 |
|----|------|
| `gc` | 선택 영역 줄 주석 토글 |
| `gb` | 선택 영역 블록 주석 토글 |

> **팁**: `gc`는 Vim의 편집 언어와 자연스럽게 결합된다. Chapter 1에서 배운 "동사 + 명사" 구조 그대로 `gc`(주석) + `ip`(단락) = `gcip`(단락 주석)처럼 사용할 수 있다.

## 5. 자동 괄호: nvim-autopairs

### 소개

nvim-autopairs는 여는 괄호를 입력하면 닫는 괄호를 자동으로 추가해준다. `(`, `{`, `[`, `"`, `'` 등을 입력하면 짝이 자동 완성된다.

### 설치

```lua
-- plugins/autopairs.lua
return {
  "windwp/nvim-autopairs",
  event = "InsertEnter",
  config = function()
    local autopairs = require("nvim-autopairs")
    autopairs.setup({
      check_ts = true,  -- Treesitter 연동
    })

    -- nvim-cmp와 연동
    local cmp_autopairs = require("nvim-autopairs.completion.cmp")
    local cmp = require("cmp")
    cmp.event:on("confirm_done", cmp_autopairs.on_confirm_done())
  end,
}
```

cmp와 연동하면 완성으로 함수를 선택했을 때 자동으로 `()`가 추가된다. 예를 들어 `print`를 완성하면 `print()`가 되고 커서가 괄호 안에 위치한다.

## 6. Surround: nvim-surround

### 소개

nvim-surround는 텍스트를 감싸는 괄호, 따옴표, 태그 등을 추가/삭제/변경하는 플러그인이다. Chapter 5에서 배운 텍스트 오브젝트의 확장이라고 생각하면 된다.

### 설치

```lua
-- plugins/surround.lua
return {
  "kylechui/nvim-surround",
  version = "*",
  event = "VeryLazy",
  config = function()
    require("nvim-surround").setup()
  end,
}
```

### 주요 키맵

세 가지 핵심 동작: **추가(ys)**, **삭제(ds)**, **변경(cs)**.

#### 추가: ys (you surround)

| 명령 | 결과 |
|------|------|
| `ysiw"` | `hello` → `"hello"` |
| `ysiw)` | `hello` → `(hello)` |
| `ysiw(` | `hello` → `( hello )` (공백 포함) |
| `ysiw}` | `hello` → `{hello}` |
| `ysiw{` | `hello` → `{ hello }` (공백 포함) |
| `yssb` | 줄 전체를 `()`로 감쌈 |
| `yss"` | 줄 전체를 `""`로 감쌈 |

#### 삭제: ds (delete surround)

| 명령 | 결과 |
|------|------|
| `ds"` | `"hello"` → `hello` |
| `ds)` | `(hello)` → `hello` |
| `dst` | `<div>hello</div>` → `hello` |

#### 변경: cs (change surround)

| 명령 | 결과 |
|------|------|
| `cs"'` | `"hello"` → `'hello'` |
| `cs)]` | `(hello)` → `[hello]` |
| `cs"t` | `"hello"` → `<tag>hello</tag>` (태그 이름 입력 프롬프트) |

> **팁**: 닫는 괄호(`)`, `]`, `}`)를 사용하면 공백 없이, 여는 괄호(`(`, `[`, `{`)를 사용하면 공백이 포함된다.

## 7. 포매팅: conform.nvim

### 소개

conform.nvim은 코드 포매터를 실행해주는 플러그인이다. Prettier, Black, stylua 등 외부 포매터와 연동하여 파일 저장 시 자동으로 코드를 정리할 수 있다.

### 설치

```lua
-- plugins/formatting.lua
return {
  "stevearc/conform.nvim",
  event = { "BufWritePre" },
  cmd = { "ConformInfo" },
  config = function()
    require("conform").setup({
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

      -- 저장 시 자동 포매팅
      format_on_save = {
        timeout_ms = 500,
        lsp_format = "fallback",
      },
    })
  end,
}
```

### 주요 키맵

```lua
vim.keymap.set({ "n", "v" }, "<leader>cf", function()
  require("conform").format({
    lsp_format = "fallback",
    async = false,
    timeout_ms = 1000,
  })
end, { desc = "코드 포매팅" })
```

`lsp_format = "fallback"`은 conform에 해당 파일 타입의 포매터가 없으면 LSP의 포매팅 기능을 사용한다는 뜻이다.

> **팁**: 포매터는 별도로 설치해야 한다. `npm install -g prettier`, `pip install black`, `cargo install stylua` 등으로 시스템에 설치한다.

## 8. 린팅: nvim-lint

### 소개

nvim-lint는 코드 린터(linter)를 실행하여 잠재적 오류와 코드 스타일 문제를 알려준다. LSP의 진단 기능과 보완적으로 사용된다.

### 설치

```lua
-- plugins/linting.lua
return {
  "mfussenegger/nvim-lint",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    local lint = require("lint")

    lint.linters_by_ft = {
      javascript = { "eslint_d" },
      typescript = { "eslint_d" },
      javascriptreact = { "eslint_d" },
      typescriptreact = { "eslint_d" },
      python = { "pylint" },
    }

    -- 저장 시, 파일 읽기 시 린팅 실행
    local lint_augroup = vim.api.nvim_create_augroup("lint", { clear = true })
    vim.api.nvim_create_autocmd({ "BufEnter", "BufWritePost", "InsertLeave" }, {
      group = lint_augroup,
      callback = function()
        lint.try_lint()
      end,
    })
  end,
}
```

린팅 결과는 LSP 진단과 동일한 방식으로 표시된다. `]d`와 `[d`로 진단 항목 사이를 이동할 수 있다.

## 9. 컬러스킴: 인기 테마들

컬러스킴은 코드의 가독성과 편집 경험에 직접적인 영향을 미친다.

### tokyonight

```lua
-- plugins/colorscheme.lua
return {
  "folke/tokyonight.nvim",
  lazy = false,
  priority = 1000,  -- 다른 플러그인보다 먼저 로드
  config = function()
    require("tokyonight").setup({
      style = "night",    -- "storm", "moon", "night", "day"
      transparent = false,
    })
    vim.cmd.colorscheme("tokyonight")
  end,
}
```

### catppuccin

```lua
return {
  "catppuccin/nvim",
  name = "catppuccin",
  lazy = false,
  priority = 1000,
  config = function()
    require("catppuccin").setup({
      flavour = "mocha",  -- "latte", "frappe", "macchiato", "mocha"
      integrations = {
        cmp = true,
        gitsigns = true,
        neotree = true,
        treesitter = true,
        telescope = { enabled = true },
      },
    })
    vim.cmd.colorscheme("catppuccin")
  end,
}
```

`priority = 1000`은 이 플러그인을 다른 플러그인보다 먼저 로드한다. 컬러스킴은 UI에 영향을 미치므로 가장 먼저 적용되어야 한다.

### 기타 인기 테마

| 테마 | 특징 |
|------|------|
| `rose-pine/neovim` | 부드러운 파스텔 톤 |
| `rebelot/kanagawa.nvim` | 일본 전통 색상 기반 |
| `EdenEast/nightfox.nvim` | 다양한 변형 제공 |
| `navarasu/onedark.nvim` | One Dark Pro 기반 |
| `sainnhe/gruvbox-material` | Gruvbox의 부드러운 변형 |

## 10. which-key.nvim: 키맵 가이드

### 소개

which-key.nvim은 키를 누르고 잠시 기다리면 사용 가능한 후속 키맵을 팝업으로 표시해준다. `<leader>`를 누르면 어떤 키를 더 누를 수 있는지 안내해주므로 키맵을 외울 필요가 줄어든다.

### 설치

```lua
-- plugins/which-key.lua
return {
  "folke/which-key.nvim",
  event = "VeryLazy",
  config = function()
    local wk = require("which-key")

    wk.setup({
      delay = 300,  -- 팝업이 뜨기까지의 지연 시간 (ms)
    })

    -- 키 그룹 이름 등록
    wk.add({
      { "<leader>f", group = "찾기 (Find)" },
      { "<leader>g", group = "Git" },
      { "<leader>h", group = "Hunk" },
      { "<leader>c", group = "코드 (Code)" },
      { "<leader>d", group = "진단 (Diagnostics)" },
      { "<leader>t", group = "토글 (Toggle)" },
    })
  end,
}
```

`<leader>`를 누르고 300ms 기다리면 다음과 같은 팝업이 나타난다:

```
┌──────────────────────────────────────┐
│  f  찾기 (Find)                       │
│  g  Git                              │
│  h  Hunk                             │
│  c  코드 (Code)                       │
│  e  파일 탐색기                        │
│  ...                                 │
└──────────────────────────────────────┘
```

`f`를 누르면 `<leader>f` 아래의 키맵이 다시 표시된다. 키맵을 체계적으로 정리하는 데도 도움이 된다.

## 11. indent-blankline.nvim: 들여쓰기 가이드라인

### 소개

indent-blankline.nvim은 들여쓰기 수준을 세로 선으로 표시해준다. 중첩된 코드 블록의 구조를 시각적으로 파악하기 쉬워진다.

### 설치

```lua
-- plugins/indent-blankline.lua
return {
  "lukas-reineke/indent-blankline.nvim",
  main = "ibl",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    require("ibl").setup({
      indent = {
        char = "│",
      },
      scope = {
        enabled = true,
        show_start = true,
        show_end = false,
      },
    })
  end,
}
```

`scope`는 현재 커서가 위치한 코드 블록의 들여쓰기 선을 강조 표시한다. Treesitter와 연동되어 정확한 범위를 파악한다.

## 12. todo-comments.nvim: TODO/FIXME 하이라이트

### 소개

todo-comments.nvim은 코드 내의 `TODO`, `FIXME`, `HACK`, `NOTE` 등의 주석을 눈에 띄는 색상으로 하이라이트하고, 검색할 수 있게 해준다.

### 설치

```lua
-- plugins/todo-comments.lua
return {
  "folke/todo-comments.nvim",
  event = { "BufReadPre", "BufNewFile" },
  dependencies = { "nvim-lua/plenary.nvim" },
  config = function()
    require("todo-comments").setup()
  end,
}
```

기본 설정만으로 충분하다.

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

### 주요 키맵

```lua
vim.keymap.set("n", "]t", function()
  require("todo-comments").jump_next()
end, { desc = "다음 TODO" })

vim.keymap.set("n", "[t", function()
  require("todo-comments").jump_prev()
end, { desc = "이전 TODO" })
```

Telescope와 연동하여 프로젝트 전체의 TODO를 검색할 수도 있다:

```vim
:TodoTelescope
```

## 전체 플러그인 통합 설정

위에서 다룬 모든 플러그인을 하나의 lazy.nvim 설정으로 정리하면 다음과 같다. 이 구조를 참고하여 자신의 `lua/plugins/` 디렉터리에 파일을 배치하면 된다.

```
~/.config/nvim/
├── init.lua
└── lua/
    └── plugins/
        ├── neo-tree.lua
        ├── gitsigns.lua
        ├── lualine.lua
        ├── comment.lua
        ├── autopairs.lua
        ├── surround.lua
        ├── formatting.lua
        ├── linting.lua
        ├── colorscheme.lua
        ├── which-key.lua
        ├── indent-blankline.lua
        └── todo-comments.lua
```

lazy.nvim은 `lua/plugins/` 디렉터리의 모든 `.lua` 파일을 자동으로 읽어 플러그인을 로드한다(Chapter 15 참조). 각 파일이 하나의 플러그인(또는 관련 플러그인 그룹)을 담당하므로 관리가 깔끔하다.

### 단일 파일로 관리하는 경우

모든 플러그인을 한 파일에 모을 수도 있다:

```lua
-- plugins/essentials.lua
return {
  -- 파일 탐색기
  {
    "nvim-neo-tree/neo-tree.nvim",
    branch = "v3.x",
    dependencies = {
      "nvim-lua/plenary.nvim",
      "nvim-tree/nvim-web-devicons",
      "MunifTanjim/nui.nvim",
    },
    keys = {
      { "<leader>e", "<cmd>Neotree toggle<cr>", desc = "파일 탐색기" },
    },
    config = function()
      require("neo-tree").setup({
        close_if_last_window = true,
        filesystem = {
          follow_current_file = { enabled = true },
        },
      })
    end,
  },

  -- Git signs
  {
    "lewis6991/gitsigns.nvim",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- 상태 표시줄
  {
    "nvim-lualine/lualine.nvim",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    opts = {
      options = { theme = "auto" },
    },
  },

  -- 주석
  {
    "numToStr/Comment.nvim",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- 자동 괄호
  {
    "windwp/nvim-autopairs",
    event = "InsertEnter",
    opts = { check_ts = true },
  },

  -- Surround
  {
    "kylechui/nvim-surround",
    version = "*",
    event = "VeryLazy",
    opts = {},
  },

  -- 포매팅
  {
    "stevearc/conform.nvim",
    event = { "BufWritePre" },
    opts = {
      formatters_by_ft = {
        lua = { "stylua" },
        javascript = { "prettier" },
        typescript = { "prettier" },
        python = { "black" },
      },
      format_on_save = {
        timeout_ms = 500,
        lsp_format = "fallback",
      },
    },
  },

  -- 린팅
  {
    "mfussenegger/nvim-lint",
    event = { "BufReadPre", "BufNewFile" },
    config = function()
      require("lint").linters_by_ft = {
        javascript = { "eslint_d" },
        typescript = { "eslint_d" },
        python = { "pylint" },
      }
      vim.api.nvim_create_autocmd({ "BufWritePost" }, {
        callback = function()
          require("lint").try_lint()
        end,
      })
    end,
  },

  -- 컬러스킴
  {
    "folke/tokyonight.nvim",
    lazy = false,
    priority = 1000,
    config = function()
      vim.cmd.colorscheme("tokyonight")
    end,
  },

  -- 키맵 가이드
  { "folke/which-key.nvim", event = "VeryLazy", opts = {} },

  -- 들여쓰기 가이드
  {
    "lukas-reineke/indent-blankline.nvim",
    main = "ibl",
    event = { "BufReadPre", "BufNewFile" },
    opts = {},
  },

  -- TODO 하이라이트
  {
    "folke/todo-comments.nvim",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = { "nvim-lua/plenary.nvim" },
    opts = {},
  },
}
```

## 실습: 필수 플러그인 체험

1. 위의 플러그인들을 설치한 후 Neovim을 재시작한다
2. `<leader>e`로 Neo-tree를 열고, `a`로 새 파일을 만들어 보자
3. 파일을 수정한 뒤 gitsigns가 왼쪽 여백에 변경 표시를 하는지 확인하자
4. `gcc`로 코드 한 줄을 주석 처리하고, 다시 `gcc`로 해제해 보자
5. `(`, `"`, `{`를 입력하여 autopairs가 닫는 괄호를 자동 추가하는지 확인하자
6. `ysiw"`로 단어를 따옴표로 감싸고, `cs"'`로 큰따옴표를 작은따옴표로 변경해 보자
7. `<leader>`를 누르고 잠시 기다려 which-key 팝업을 확인하자
8. `TODO: 테스트`라고 주석을 작성하여 하이라이트가 적용되는지 확인하자
9. 파일을 저장하여 자동 포매팅이 동작하는지 확인하자 (포매터가 설치된 경우)

## 핵심 정리

- **Neo-tree**: 파일 탐색기, `<leader>e`로 토글, `a`(생성), `d`(삭제), `r`(이름 변경)
- **gitsigns**: 줄 단위 Git 변경 표시, `]h`/`[h`로 hunk 이동, `<leader>hs`로 스테이징
- **lualine**: 상태 표시줄, 모드/브랜치/진단 정보를 한눈에 표시
- **Comment.nvim**: `gcc`로 줄 주석 토글, `gc{motion}`으로 범위 주석
- **nvim-autopairs**: 괄호/따옴표 자동 짝 맞추기, cmp와 연동 가능
- **nvim-surround**: `ys`(추가), `ds`(삭제), `cs`(변경)로 감싸는 문자 조작
- **conform.nvim**: 저장 시 자동 포매팅, 파일 타입별 포매터 설정
- **nvim-lint**: 린터 실행, LSP 진단과 동일하게 표시
- **which-key**: `<leader>` 후 키맵 가이드 팝업, 키맵 체계 정리에 유용
- **indent-blankline**: 들여쓰기 세로 선, 현재 스코프 강조
- **todo-comments**: TODO/FIXME 하이라이트, `]t`/`[t`로 이동, Telescope 연동
- 각 플러그인은 **개별 파일로 관리**하면 추가/제거가 깔끔하다
