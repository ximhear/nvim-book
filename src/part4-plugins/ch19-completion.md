# Chapter 19: 자동 완성

코드를 작성할 때 변수명, 함수명, 파일 경로를 매번 처음부터 끝까지 타이핑하는 것은 비효율적이고 오타의 원인이 된다. 자동 완성(auto-completion)은 몇 글자만 입력하면 나머지를 제안해주는 기능으로, 현대 개발 환경에서 빠질 수 없는 핵심 기능이다.

Neovim에는 기본적인 완성 기능(`<C-n>`, `<C-p>`)이 내장되어 있지만, LSP 기반 완성, 스니펫, 경로 완성 등을 통합적으로 제공하려면 전용 플러그인이 필요하다.

## 자동 완성의 구조

Neovim의 자동 완성 시스템은 세 가지 요소로 구성된다:

```
┌─────────────────────────────────────────┐
│          완성 엔진 (nvim-cmp)            │
│  ┌───────────────────────────────────┐  │
│  │        소스(Sources)               │  │
│  │  ┌─────────┐ ┌────────┐ ┌──────┐ │  │
│  │  │  LSP    │ │ Buffer │ │ Path │ │  │
│  │  └─────────┘ └────────┘ └──────┘ │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │     스니펫 엔진 (LuaSnip)         │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

- **완성 엔진**: 여러 소스에서 후보를 모아 사용자에게 표시하고 선택하게 해주는 프레임워크
- **소스(Source)**: 완성 후보를 제공하는 개별 모듈 (LSP, 버퍼 단어, 파일 경로 등)
- **스니펫 엔진**: 코드 템플릿을 확장해주는 도구

## nvim-cmp 소개와 설치

**nvim-cmp**는 Neovim에서 가장 널리 사용되는 완성 엔진이다. Lua로 작성되어 빠르고, 소스를 자유롭게 추가할 수 있다.

### 기본 설치

```lua
-- plugins/completion.lua
return {
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    -- 소스
    "hrsh7th/cmp-nvim-lsp",   -- LSP 완성
    "hrsh7th/cmp-buffer",      -- 버퍼 단어 완성
    "hrsh7th/cmp-path",        -- 파일 경로 완성
    "hrsh7th/cmp-cmdline",     -- 명령줄 완성

    -- 스니펫
    "L3MON4D3/LuaSnip",
    "saadparwaiz1/cmp_luasnip",
  },
  config = function()
    -- 설정은 아래에서 단계별로 다룬다
  end,
}
```

`event = "InsertEnter"`는 Insert 모드에 진입할 때 플러그인을 로드한다는 뜻이다. 시작 시간에 영향을 주지 않으면서 필요할 때 바로 사용할 수 있다.

## 소스(Source) 설정

각 소스는 서로 다른 종류의 완성 후보를 제공한다.

### cmp-nvim-lsp: LSP 기반 완성

Chapter 17에서 설정한 LSP 서버가 제공하는 완성 후보를 가져온다. 함수 시그니처, 타입 정보, 자동 임포트 등 지능적인 완성이 가능하다.

```lua
-- LSP 서버에 cmp의 capabilities를 전달해야 한다
local capabilities = require("cmp_nvim_lsp").default_capabilities()

-- lspconfig 설정 시 capabilities 추가
require("lspconfig").lua_ls.setup({
  capabilities = capabilities,
})
```

이 설정이 없으면 LSP 서버가 클라이언트(Neovim)의 완성 기능 범위를 모르기 때문에, 일부 완성 후보가 누락될 수 있다.

### cmp-buffer: 현재 버퍼 단어 완성

현재 열려 있는 버퍼의 단어를 완성 후보로 제공한다. LSP가 지원되지 않는 파일에서도 동작하므로 범용적이다.

### cmp-path: 파일 경로 완성

파일 시스템 경로를 자동 완성한다. `./`이나 `/`를 입력하면 디렉터리와 파일 목록이 나타난다. 설정 파일이나 import 경로를 작성할 때 유용하다.

### cmp-cmdline: 명령줄 완성

Neovim의 명령줄(`:` 입력 후)과 검색(`/`, `?`)에서도 자동 완성을 사용할 수 있다.

```lua
local cmp = require("cmp")

-- `/`, `?` 검색 시 버퍼 단어 완성
cmp.setup.cmdline({ "/", "?" }, {
  mapping = cmp.mapping.preset.cmdline(),
  sources = {
    { name = "buffer" },
  },
})

-- `:` 명령줄에서 경로와 명령 완성
cmp.setup.cmdline(":", {
  mapping = cmp.mapping.preset.cmdline(),
  sources = cmp.config.sources({
    { name = "path" },
  }, {
    { name = "cmdline" },
  }),
})
```

## 스니펫 엔진: LuaSnip

스니펫(snippet)은 자주 사용하는 코드 패턴을 축약어로 빠르게 입력할 수 있게 해준다. 예를 들어 `fn`을 입력하고 확장하면 완전한 함수 구조가 만들어진다.

### LuaSnip 설치와 cmp 연동

LuaSnip은 Neovim 생태계에서 가장 인기 있는 Lua 기반 스니펫 엔진이다.

```lua
{
  "L3MON4D3/LuaSnip",
  version = "v2.*",
  build = "make install_jsregexp",
  dependencies = {
    "rafamadriz/friendly-snippets",
  },
  config = function()
    require("luasnip.loaders.from_vscode").lazy_load()
  end,
}
```

`lazy_load()`는 현재 파일 타입에 해당하는 스니펫만 로드한다. JavaScript 파일을 열면 JavaScript 스니펫만, Python 파일을 열면 Python 스니펫만 활성화된다.

### friendly-snippets: 미리 정의된 스니펫 모음

`friendly-snippets`는 다양한 언어에 대한 VS Code 형식 스니펫 모음이다. 별도의 스니펫 작성 없이 바로 사용할 수 있다.

주요 스니펫 예시:

| 축약어 | 언어 | 확장 결과 |
|--------|------|-----------|
| `fn` | JavaScript | 함수 선언 |
| `clg` | JavaScript | `console.log()` |
| `imp` | JavaScript | `import ... from ...` |
| `def` | Python | 함수 정의 |
| `class` | Python | 클래스 정의 |
| `if` | 대부분 | if 블록 |
| `for` | 대부분 | for 루프 |

### 커스텀 스니펫 작성 기초

LuaSnip에서 직접 스니펫을 만들 수 있다:

```lua
local ls = require("luasnip")
local s = ls.snippet
local t = ls.text_node
local i = ls.insert_node

ls.add_snippets("lua", {
  -- "req"를 입력하면 require 문으로 확장
  s("req", {
    t('local '),
    i(1, "module"),
    t(' = require("'),
    i(2, "module"),
    t('")'),
  }),
})
```

위 스니펫의 동작:

1. `req`를 입력하고 스니펫 확장
2. `local ▌module = require("module")` — 첫 번째 `module`에 커서
3. 이름을 입력하고 `<Tab>` — 두 번째 `module`로 이동
4. 경로를 입력하고 `<Tab>` — 스니펫 완료

`i(1)`, `i(2)`는 탭으로 이동하는 **입력 지점(insert node)**이다. 숫자가 이동 순서를 결정한다.

## nvim-cmp 키맵 설정

완성 메뉴에서의 키맵 설정은 사용 경험에 큰 영향을 미친다.

```lua
local cmp = require("cmp")
local luasnip = require("luasnip")

cmp.setup({
  snippet = {
    expand = function(args)
      luasnip.lsp_expand(args.body)
    end,
  },

  mapping = cmp.mapping.preset.insert({
    -- 완성 목록 위/아래 이동
    ["<C-p>"] = cmp.mapping.select_prev_item(),
    ["<C-n>"] = cmp.mapping.select_next_item(),

    -- 문서 스크롤
    ["<C-b>"] = cmp.mapping.scroll_docs(-4),
    ["<C-f>"] = cmp.mapping.scroll_docs(4),

    -- 완성 메뉴 수동 호출
    ["<C-Space>"] = cmp.mapping.complete(),

    -- 완성 취소
    ["<C-e>"] = cmp.mapping.abort(),

    -- 선택 확정
    ["<CR>"] = cmp.mapping.confirm({ select = false }),

    -- Tab: 완성 선택 또는 스니펫 이동
    ["<Tab>"] = cmp.mapping(function(fallback)
      if cmp.visible() then
        cmp.select_next_item()
      elseif luasnip.expand_or_jumpable() then
        luasnip.expand_or_jump()
      else
        fallback()
      end
    end, { "i", "s" }),

    -- Shift-Tab: 역방향
    ["<S-Tab>"] = cmp.mapping(function(fallback)
      if cmp.visible() then
        cmp.select_prev_item()
      elseif luasnip.jumpable(-1) then
        luasnip.jump(-1)
      else
        fallback()
      end
    end, { "i", "s" }),
  }),
})
```

### 키맵 요약

| 키 | 동작 |
|----|------|
| `<C-n>` / `<Tab>` | 다음 완성 항목 |
| `<C-p>` / `<S-Tab>` | 이전 완성 항목 |
| `<CR>` | 선택 확정 |
| `<C-Space>` | 완성 메뉴 수동 호출 |
| `<C-e>` | 완성 취소 |
| `<C-b>` / `<C-f>` | 문서 스크롤 |
| `<Tab>` (스니펫 내) | 다음 입력 지점으로 이동 |
| `<S-Tab>` (스니펫 내) | 이전 입력 지점으로 이동 |

> **팁**: `<CR>`에서 `select = false`로 설정하면 항목을 명시적으로 선택한 경우에만 확정된다. 엔터를 눌렀을 때 의도치 않게 첫 번째 항목이 입력되는 것을 방지한다.

## 완성 UI 커스터마이징

### 소스별 표시 이름

완성 메뉴에서 각 항목이 어떤 소스에서 온 것인지 표시할 수 있다:

```lua
cmp.setup({
  formatting = {
    format = function(entry, vim_item)
      -- 소스 이름 표시
      vim_item.menu = ({
        nvim_lsp = "[LSP]",
        luasnip = "[Snippet]",
        buffer = "[Buffer]",
        path = "[Path]",
      })[entry.source.name]
      return vim_item
    end,
  },
})
```

완성 메뉴가 다음과 같이 표시된다:

```
  print          [LSP]
  println        [LSP]
  printf         [Buffer]
  fn → function  [Snippet]
  ./src/         [Path]
```

### 완성 창 스타일

```lua
cmp.setup({
  window = {
    completion = cmp.config.window.bordered(),
    documentation = cmp.config.window.bordered(),
  },
})
```

`bordered()`는 완성 메뉴와 문서 미리보기 창에 테두리를 추가한다.

### 소스 우선순위

`sources` 배열의 순서가 곧 우선순위다. 그룹으로 나눌 수도 있다:

```lua
cmp.setup({
  sources = cmp.config.sources({
    -- 첫 번째 그룹: 이 그룹에 결과가 있으면 두 번째 그룹은 표시하지 않음
    { name = "nvim_lsp" },
    { name = "luasnip" },
  }, {
    -- 두 번째 그룹: 첫 번째 그룹에 결과가 없을 때만 표시
    { name = "buffer" },
    { name = "path" },
  }),
})
```

이렇게 하면 LSP 결과가 있을 때는 LSP와 스니펫만 표시되고, LSP가 없는 파일에서는 버퍼 단어와 경로가 표시된다.

## lspkind.nvim으로 아이콘 표시

`lspkind.nvim`은 완성 항목 옆에 VS Code 스타일의 아이콘을 표시해준다. 함수인지, 변수인지, 클래스인지 한눈에 구분할 수 있다.

```lua
{
  "onsails/lspkind.nvim",
}
```

nvim-cmp의 formatting과 통합:

```lua
local lspkind = require("lspkind")

cmp.setup({
  formatting = {
    format = lspkind.cmp_format({
      mode = "symbol_text",  -- "symbol", "text", "symbol_text" 중 선택
      maxwidth = 50,
      ellipsis_char = "...",
      menu = {
        nvim_lsp = "[LSP]",
        luasnip = "[Snippet]",
        buffer = "[Buffer]",
        path = "[Path]",
      },
    }),
  },
})
```

결과:

```
   print          Function   [LSP]
   count          Variable   [LSP]
   MyClass        Class      [LSP]
   fn → function  Snippet    [Snippet]
```

## 통합 설정 예제

지금까지 다룬 모든 내용을 하나의 설정 파일로 정리하면 다음과 같다:

```lua
-- plugins/completion.lua
return {
  -- 완성 엔진
  {
    "hrsh7th/nvim-cmp",
    event = "InsertEnter",
    dependencies = {
      -- 소스
      "hrsh7th/cmp-nvim-lsp",
      "hrsh7th/cmp-buffer",
      "hrsh7th/cmp-path",
      "hrsh7th/cmp-cmdline",

      -- 스니펫 엔진
      {
        "L3MON4D3/LuaSnip",
        version = "v2.*",
        build = "make install_jsregexp",
        dependencies = {
          "rafamadriz/friendly-snippets",
        },
        config = function()
          require("luasnip.loaders.from_vscode").lazy_load()
        end,
      },
      "saadparwaiz1/cmp_luasnip",

      -- 아이콘
      "onsails/lspkind.nvim",
    },
    config = function()
      local cmp = require("cmp")
      local luasnip = require("luasnip")
      local lspkind = require("lspkind")

      cmp.setup({
        snippet = {
          expand = function(args)
            luasnip.lsp_expand(args.body)
          end,
        },

        mapping = cmp.mapping.preset.insert({
          ["<C-p>"] = cmp.mapping.select_prev_item(),
          ["<C-n>"] = cmp.mapping.select_next_item(),
          ["<C-b>"] = cmp.mapping.scroll_docs(-4),
          ["<C-f>"] = cmp.mapping.scroll_docs(4),
          ["<C-Space>"] = cmp.mapping.complete(),
          ["<C-e>"] = cmp.mapping.abort(),
          ["<CR>"] = cmp.mapping.confirm({ select = false }),

          ["<Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then
              cmp.select_next_item()
            elseif luasnip.expand_or_jumpable() then
              luasnip.expand_or_jump()
            else
              fallback()
            end
          end, { "i", "s" }),

          ["<S-Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then
              cmp.select_prev_item()
            elseif luasnip.jumpable(-1) then
              luasnip.jump(-1)
            else
              fallback()
            end
          end, { "i", "s" }),
        }),

        sources = cmp.config.sources({
          { name = "nvim_lsp" },
          { name = "luasnip" },
        }, {
          { name = "buffer" },
          { name = "path" },
        }),

        formatting = {
          format = lspkind.cmp_format({
            mode = "symbol_text",
            maxwidth = 50,
            ellipsis_char = "...",
            menu = {
              nvim_lsp = "[LSP]",
              luasnip = "[Snippet]",
              buffer = "[Buffer]",
              path = "[Path]",
            },
          }),
        },

        window = {
          completion = cmp.config.window.bordered(),
          documentation = cmp.config.window.bordered(),
        },
      })

      -- 명령줄 완성
      cmp.setup.cmdline({ "/", "?" }, {
        mapping = cmp.mapping.preset.cmdline(),
        sources = {
          { name = "buffer" },
        },
      })

      cmp.setup.cmdline(":", {
        mapping = cmp.mapping.preset.cmdline(),
        sources = cmp.config.sources({
          { name = "path" },
        }, {
          { name = "cmdline" },
        }),
      })
    end,
  },
}
```

### LSP 서버에 capabilities 전달

이 설정과 함께 Chapter 17의 LSP 설정도 수정해야 한다:

```lua
-- plugins/lsp.lua에서
local capabilities = require("cmp_nvim_lsp").default_capabilities()

-- 각 LSP 서버 설정에 capabilities 추가
require("lspconfig").lua_ls.setup({
  capabilities = capabilities,
  -- 기타 설정...
})

require("lspconfig").ts_ls.setup({
  capabilities = capabilities,
})
```

## 실습: 자동 완성 체험

1. 위의 통합 설정을 적용한 뒤 Neovim을 재시작한다
2. Lua 파일을 열고 `vim.`을 입력해 보자 -- LSP 기반 완성이 나타나야 한다
3. `req`를 입력하고 스니펫이 나타나면 `<Tab>`으로 확장해 보자
4. `./`를 입력하여 파일 경로 완성을 시도해 보자
5. 완성 메뉴에서 `<C-n>`과 `<C-p>`로 항목을 오갈 수 있는지 확인하자
6. `<C-e>`로 완성을 취소하고, `<C-Space>`로 수동 호출해 보자
7. `:` 명령줄에서 `:Tele`를 입력해 명령줄 완성이 동작하는지 확인하자

## 핵심 정리

- 자동 완성 시스템은 **완성 엔진**(nvim-cmp) + **소스** + **스니펫 엔진**으로 구성된다
- 핵심 소스: **cmp-nvim-lsp**(LSP), **cmp-buffer**(버퍼 단어), **cmp-path**(경로), **cmp-cmdline**(명령줄)
- **LuaSnip** + **friendly-snippets**로 미리 정의된 코드 스니펫을 바로 사용할 수 있다
- 키맵: `<Tab>`/`<S-Tab>`으로 항목 이동, `<CR>`로 확정, `<C-Space>`로 수동 호출, `<C-e>`로 취소
- LSP 서버에 **`cmp_nvim_lsp`의 capabilities를 전달**해야 LSP 완성이 제대로 동작한다
- 소스 **그룹**을 설정하면 LSP 결과가 있을 때 버퍼 단어가 섞이지 않는다
- **lspkind.nvim**으로 완성 항목에 아이콘을 표시하면 종류를 한눈에 구분할 수 있다
