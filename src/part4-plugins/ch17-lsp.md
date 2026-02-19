# Chapter 17: LSP - 언어 서버 프로토콜

IDE에서 당연하게 사용하는 기능들이 있다. 함수 정의로 점프하기, 변수의 모든 참조 찾기, 코드 자동 완성, 실시간 오류 표시, 자동 포매팅. 이 모든 것을 가능하게 하는 것이 **LSP(Language Server Protocol)**다. Neovim은 LSP 클라이언트를 내장하고 있어서, 적절한 설정만 하면 VSCode에 견줄 만한 개발 환경을 구축할 수 있다.

## LSP란

**LSP(Language Server Protocol)**는 에디터와 언어 서버(language server) 간의 **표준 통신 프로토콜**이다. Microsoft가 VSCode를 위해 개발했으며, 현재는 에디터와 언어에 관계없이 사용되는 공개 표준이다.

### LSP 이전의 세계

```
에디터 A + 언어 X → 전용 플러그인 A-X
에디터 A + 언어 Y → 전용 플러그인 A-Y
에디터 B + 언어 X → 전용 플러그인 B-X
에디터 B + 언어 Y → 전용 플러그인 B-Y
...
M개의 에디터 × N개의 언어 = M×N개의 플러그인 필요
```

### LSP 이후의 세계

```
에디터 A ─┐                ┌─ 언어 서버 X
에디터 B ─┤── LSP 프로토콜 ──┤─ 언어 서버 Y
에디터 C ─┘                └─ 언어 서버 Z

M개의 에디터 + N개의 언어 서버 = M+N개만 구현하면 됨
```

각 에디터는 LSP 클라이언트만 구현하면 되고, 각 언어는 LSP 서버만 구현하면 된다. 프로토콜이 표준이므로 모든 조합이 동작한다.

### 언어 서버가 제공하는 기능

| 기능 | 설명 |
|------|------|
| 자동 완성 (Completion) | 코드 입력 시 제안 목록 표시 |
| 정의로 이동 (Go to Definition) | 함수나 변수가 정의된 위치로 점프 |
| 참조 찾기 (Find References) | 심볼이 사용된 모든 위치 표시 |
| 호버 문서 (Hover) | 커서 아래 심볼의 타입, 문서 표시 |
| 이름 변경 (Rename) | 심볼의 모든 참조를 한 번에 변경 |
| 코드 액션 (Code Action) | 자동 수정, 리팩토링 제안 |
| 진단 (Diagnostics) | 오류, 경고, 린트 결과 실시간 표시 |
| 포매팅 (Formatting) | 코드 스타일 자동 정리 |
| 시그니처 도움말 (Signature Help) | 함수 호출 시 매개변수 정보 표시 |

## Neovim 내장 LSP 클라이언트

Neovim 0.5부터 LSP 클라이언트가 **내장**되어 있다. 별도의 플러그인 없이도 `vim.lsp` API를 사용할 수 있다. 하지만 내장 클라이언트만으로는 각 언어 서버의 설정을 직접 작성해야 하므로 번거롭다.

이를 간소화하는 플러그인들이 있다:

| 플러그인 | 역할 |
|----------|------|
| **nvim-lspconfig** | 각 언어 서버의 기본 설정을 제공 |
| **mason.nvim** | 언어 서버를 자동으로 설치하고 관리 |
| **mason-lspconfig.nvim** | mason과 lspconfig를 연동 |

## LSP 설정 단계별 가이드

### 1단계: 플러그인 설치

`lua/plugins/lsp.lua`:

```lua
return {
  -- mason.nvim: 언어 서버 설치 관리자
  {
    "williamboman/mason.nvim",
    cmd = "Mason",
    opts = {},
  },

  -- mason-lspconfig: mason과 lspconfig 연동
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = {
      "williamboman/mason.nvim",
    },
    opts = {
      ensure_installed = {
        "lua_ls",          -- Lua
        "ts_ls",           -- TypeScript/JavaScript
        "pyright",         -- Python
        "gopls",           -- Go
        "rust_analyzer",   -- Rust
      },
    },
  },

  -- nvim-lspconfig: 언어 서버 설정
  {
    "neovim/nvim-lspconfig",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = {
      "williamboman/mason.nvim",
      "williamboman/mason-lspconfig.nvim",
    },
    config = function()
      -- 아래에서 상세 설정
    end,
  },
}
```

### 2단계: on_attach 함수로 키맵 설정

언어 서버가 버퍼에 연결(attach)될 때 호출되는 함수를 정의한다. 이 함수에서 LSP 관련 키맵을 설정한다.

```lua
-- nvim-lspconfig의 config 함수 안에서:
config = function()
  local on_attach = function(client, bufnr)
    -- 키맵 설정을 위한 헬퍼 함수
    local map = function(keys, func, desc)
      vim.keymap.set("n", keys, func, { buffer = bufnr, desc = "LSP: " .. desc })
    end

    -- 이동
    map("gd", vim.lsp.buf.definition, "정의로 이동 (Go to Definition)")
    map("gD", vim.lsp.buf.declaration, "선언으로 이동 (Go to Declaration)")
    map("gr", vim.lsp.buf.references, "참조 찾기 (Find References)")
    map("gi", vim.lsp.buf.implementation, "구현으로 이동 (Go to Implementation)")
    map("gt", vim.lsp.buf.type_definition, "타입 정의로 이동 (Type Definition)")

    -- 정보 표시
    map("K", vim.lsp.buf.hover, "호버 문서 (Hover)")
    map("<C-k>", vim.lsp.buf.signature_help, "시그니처 도움말 (Signature Help)")

    -- 코드 수정
    map("<leader>rn", vim.lsp.buf.rename, "이름 변경 (Rename)")
    map("<leader>ca", vim.lsp.buf.code_action, "코드 액션 (Code Action)")

    -- 진단
    map("[d", vim.diagnostic.goto_prev, "이전 진단으로 이동")
    map("]d", vim.diagnostic.goto_next, "다음 진단으로 이동")
    map("<leader>e", vim.diagnostic.open_float, "진단 상세 표시")
    map("<leader>q", vim.diagnostic.setloclist, "진단 목록 열기")

    -- 포매팅
    map("<leader>f", function()
      vim.lsp.buf.format({ async = true })
    end, "포매팅 (Format)")
  end

  -- 언어 서버 설정 (3단계에서 계속)
end,
```

### 3단계: 언어 서버 설정

```lua
config = function()
  local on_attach = function(client, bufnr)
    -- (위의 키맵 설정)
  end

  local capabilities = vim.lsp.protocol.make_client_capabilities()

  local lspconfig = require("lspconfig")

  -- mason-lspconfig으로 자동 설정
  require("mason-lspconfig").setup_handlers({
    -- 기본 핸들러: 모든 서버에 적용
    function(server_name)
      lspconfig[server_name].setup({
        on_attach = on_attach,
        capabilities = capabilities,
      })
    end,

    -- 특정 서버의 커스텀 설정
    ["lua_ls"] = function()
      lspconfig.lua_ls.setup({
        on_attach = on_attach,
        capabilities = capabilities,
        settings = {
          Lua = {
            diagnostics = {
              globals = { "vim" },   -- vim 글로벌 변수 인식
            },
            workspace = {
              library = vim.api.nvim_get_runtime_file("", true),
              checkThirdParty = false,
            },
          },
        },
      })
    end,
  })
end,
```

> **흐름 정리**: mason.nvim이 언어 서버를 설치하고, mason-lspconfig가 설치된 서버 목록을 lspconfig에 전달하고, lspconfig가 각 서버를 적절한 설정으로 시작한다.

## 주요 LSP 기능과 키맵

### 이동 기능

**`gd` - 정의로 이동 (Go to Definition)**

커서 아래의 함수나 변수가 **정의된 위치**로 점프한다. 가장 자주 사용하는 LSP 기능이다.

```python
def calculate(x, y):    # ← gd를 누르면 여기로 점프
    return x + y

result = calculate(1, 2)
            ↑ 커서를 여기에 놓고 gd
```

`<C-o>`로 이전 위치로 돌아올 수 있다.

**`gr` - 참조 찾기 (Find References)**

커서 아래의 심볼이 **사용된 모든 위치**를 보여준다.

```python
def calculate(x, y):
    return x + y

result = calculate(1, 2)   # ← 참조 1
total = calculate(3, 4)    # ← 참조 2
print(calculate(5, 6))     # ← 참조 3
```

`calculate` 위에서 `gr`을 누르면 세 곳의 참조가 목록으로 표시된다.

**`K` - 호버 문서 (Hover)**

커서 아래 심볼의 **타입 정보와 문서**를 떠 있는 창(floating window)에 표시한다.

```typescript
const items: string[] = ["a", "b", "c"];
items.map(...)
  ↑ 여기서 K를 누르면 map의 시그니처와 설명이 표시된다
```

### 코드 수정 기능

**`<leader>rn` - 이름 변경 (Rename)**

변수나 함수의 이름을 **프로젝트 전체에서** 한 번에 변경한다. 단순 검색/치환과 달리, LSP는 코드의 의미를 이해하므로 동명의 다른 심볼은 건드리지 않는다.

```python
# "user" 위에서 <leader>rn → "customer"로 입력

# before
def get_user(user_id):
    user = db.find(user_id)
    return user

# after
def get_customer(customer_id):
    customer = db.find(customer_id)
    return customer
```

**`<leader>ca` - 코드 액션 (Code Action)**

현재 위치에서 가능한 **자동 수정이나 리팩토링**을 제안한다. 언어 서버에 따라 다르지만, 주로 다음과 같은 액션이 제공된다:

- 빠진 import 자동 추가
- 사용하지 않는 변수 제거
- 함수 추출
- 타입 변환

### 진단 탐색

**`[d` / `]d` - 진단 간 이동**

코드의 오류나 경고 사이를 순서대로 이동한다.

```python
x = undeclared_var    # ← 오류: 선언되지 않은 변수
y = 10 / 0            # ← 경고: 0으로 나누기
z = unused_value      # ← 경고: 사용되지 않는 변수
```

`]d`로 다음 진단으로, `[d`로 이전 진단으로 이동한다.

**`<leader>e` - 진단 상세 표시**

현재 줄의 진단 메시지를 떠 있는 창에 표시한다. 줄 끝에 잘려서 보이는 긴 메시지를 전체로 확인할 때 유용하다.

## 진단(Diagnostics) 설정

진단 표시 방식을 커스터마이징할 수 있다. `init.lua` 또는 LSP 설정 파일에 추가한다:

```lua
-- 진단 표시 설정
vim.diagnostic.config({
  -- 가상 텍스트(virtual text)로 줄 끝에 메시지 표시
  virtual_text = {
    prefix = "●",       -- 메시지 앞에 표시할 기호
    spacing = 4,         -- 코드와 메시지 사이 간격
  },

  -- 징후(sign) 설정: 줄 번호 옆의 아이콘
  signs = {
    text = {
      [vim.diagnostic.severity.ERROR] = " ",
      [vim.diagnostic.severity.WARN] = " ",
      [vim.diagnostic.severity.HINT] = "󰌵 ",
      [vim.diagnostic.severity.INFO] = " ",
    },
  },

  -- 진단 메시지를 떠 있는 창에 표시
  float = {
    border = "rounded",
    source = true,      -- 진단 출처(소스) 표시
  },

  -- 심각도 정렬: 오류가 경고보다 먼저 표시
  severity_sort = true,

  -- 진단 밑줄
  underline = true,

  -- Insert 모드에서는 진단 업데이트 안 함 (타이핑 방해 방지)
  update_in_insert = false,
})
```

### 진단 심각도

| 심각도 | 설명 | 기본 색상 |
|--------|------|-----------|
| ERROR | 코드가 실행되지 않는 오류 | 빨간색 |
| WARN | 잠재적 문제 (경고) | 노란색 |
| INFO | 정보성 메시지 | 파란색 |
| HINT | 개선 제안 | 초록색 |

## 포매팅

### 수동 포매팅

```vim
" 현재 버퍼 전체 포매팅
<leader>f

" 또는 명령줄에서
:lua vim.lsp.buf.format()
```

### 저장 시 자동 포매팅

`on_attach` 함수에 자동 명령(autocommand)을 추가한다:

```lua
local on_attach = function(client, bufnr)
  -- (기존 키맵 설정)

  -- 저장 시 자동 포매팅
  if client.supports_method("textDocument/formatting") then
    vim.api.nvim_create_autocmd("BufWritePre", {
      buffer = bufnr,
      callback = function()
        vim.lsp.buf.format({ bufnr = bufnr })
      end,
    })
  end
end
```

> **팁**: 포매터가 여러 개일 때 특정 포매터를 선택하고 싶다면, `vim.lsp.buf.format()`에 `filter` 옵션을 전달할 수 있다.

## 언어별 서버 설정 예시

각 언어 서버는 고유한 설정 옵션을 가지고 있다. 자주 사용하는 서버들의 설정 예시를 살펴보자.

### lua_ls (Lua)

Neovim 설정을 작성할 때 필수적이다. `vim` 글로벌 변수를 인식하도록 설정한다.

```lua
["lua_ls"] = function()
  lspconfig.lua_ls.setup({
    on_attach = on_attach,
    capabilities = capabilities,
    settings = {
      Lua = {
        runtime = { version = "LuaJIT" },
        diagnostics = {
          globals = { "vim" },
        },
        workspace = {
          library = vim.api.nvim_get_runtime_file("", true),
          checkThirdParty = false,
        },
        telemetry = { enable = false },
      },
    },
  })
end,
```

### ts_ls (TypeScript / JavaScript)

```lua
["ts_ls"] = function()
  lspconfig.ts_ls.setup({
    on_attach = on_attach,
    capabilities = capabilities,
    settings = {
      typescript = {
        inlayHints = {
          includeInlayParameterNameHints = "all",
          includeInlayFunctionParameterTypeHints = true,
          includeInlayVariableTypeHints = true,
        },
      },
    },
  })
end,
```

### pyright (Python)

```lua
["pyright"] = function()
  lspconfig.pyright.setup({
    on_attach = on_attach,
    capabilities = capabilities,
    settings = {
      python = {
        analysis = {
          typeCheckingMode = "basic",    -- "off", "basic", "strict"
          autoSearchPaths = true,
          useLibraryCodeForTypes = true,
        },
      },
    },
  })
end,
```

### gopls (Go)

```lua
["gopls"] = function()
  lspconfig.gopls.setup({
    on_attach = on_attach,
    capabilities = capabilities,
    settings = {
      gopls = {
        analyses = {
          unusedparams = true,
        },
        staticcheck = true,
        gofumpt = true,
      },
    },
  })
end,
```

### rust_analyzer (Rust)

```lua
["rust_analyzer"] = function()
  lspconfig.rust_analyzer.setup({
    on_attach = on_attach,
    capabilities = capabilities,
    settings = {
      ["rust-analyzer"] = {
        checkOnSave = {
          command = "clippy",      -- cargo check 대신 clippy 사용
        },
        cargo = {
          allFeatures = true,
        },
      },
    },
  })
end,
```

## mason.nvim 사용법

mason.nvim은 언어 서버뿐 아니라 린터(linter), 포매터(formatter) 등도 관리할 수 있는 도구 설치 관리자다.

### UI 열기

```vim
:Mason
```

직관적인 UI에서 사용 가능한 도구 목록을 확인하고 설치/삭제할 수 있다.

### 주요 명령

| 명령 | 설명 |
|------|------|
| `:Mason` | mason UI 열기 |
| `:MasonInstall <서버>` | 특정 서버 설치 |
| `:MasonUninstall <서버>` | 특정 서버 삭제 |
| `:MasonUpdate` | 설치된 도구 업데이트 |
| `:LspInfo` | 현재 버퍼에 연결된 LSP 서버 정보 |

> **팁**: `mason-lspconfig`의 `ensure_installed`에 서버를 추가하면, Neovim을 시작할 때 자동으로 설치된다. 새 컴퓨터에서도 설정 파일만 복사하면 필요한 서버가 자동 설치되므로 편리하다.

## 실전 통합 설정 예제

지금까지 다룬 모든 내용을 하나의 파일로 통합한 완전한 LSP 설정이다.

`lua/plugins/lsp.lua`:

```lua
return {
  -- mason: 도구 설치 관리
  {
    "williamboman/mason.nvim",
    cmd = "Mason",
    opts = {},
  },

  -- mason-lspconfig: mason과 lspconfig 연동
  {
    "williamboman/mason-lspconfig.nvim",
    dependencies = { "williamboman/mason.nvim" },
    opts = {
      ensure_installed = {
        "lua_ls",
        "ts_ls",
        "pyright",
        "gopls",
        "rust_analyzer",
      },
    },
  },

  -- lspconfig: 언어 서버 설정
  {
    "neovim/nvim-lspconfig",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = {
      "williamboman/mason.nvim",
      "williamboman/mason-lspconfig.nvim",
    },
    config = function()
      -- 진단 설정
      vim.diagnostic.config({
        virtual_text = { prefix = "●", spacing = 4 },
        signs = {
          text = {
            [vim.diagnostic.severity.ERROR] = " ",
            [vim.diagnostic.severity.WARN] = " ",
            [vim.diagnostic.severity.HINT] = "󰌵 ",
            [vim.diagnostic.severity.INFO] = " ",
          },
        },
        float = { border = "rounded", source = true },
        severity_sort = true,
        underline = true,
        update_in_insert = false,
      })

      -- 키맵 설정
      local on_attach = function(client, bufnr)
        local map = function(keys, func, desc)
          vim.keymap.set("n", keys, func, {
            buffer = bufnr,
            desc = "LSP: " .. desc,
          })
        end

        -- 이동
        map("gd", vim.lsp.buf.definition, "정의로 이동")
        map("gD", vim.lsp.buf.declaration, "선언으로 이동")
        map("gr", vim.lsp.buf.references, "참조 찾기")
        map("gi", vim.lsp.buf.implementation, "구현으로 이동")
        map("gt", vim.lsp.buf.type_definition, "타입 정의로 이동")

        -- 정보
        map("K", vim.lsp.buf.hover, "호버 문서")
        map("<C-k>", vim.lsp.buf.signature_help, "시그니처 도움말")

        -- 수정
        map("<leader>rn", vim.lsp.buf.rename, "이름 변경")
        map("<leader>ca", vim.lsp.buf.code_action, "코드 액션")

        -- 진단
        map("[d", vim.diagnostic.goto_prev, "이전 진단")
        map("]d", vim.diagnostic.goto_next, "다음 진단")
        map("<leader>e", vim.diagnostic.open_float, "진단 상세")
        map("<leader>q", vim.diagnostic.setloclist, "진단 목록")

        -- 포매팅
        map("<leader>f", function()
          vim.lsp.buf.format({ async = true })
        end, "포매팅")
      end

      -- 서버 능력(capabilities) 설정
      local capabilities = vim.lsp.protocol.make_client_capabilities()

      local lspconfig = require("lspconfig")

      -- mason-lspconfig 핸들러
      require("mason-lspconfig").setup_handlers({
        -- 기본 핸들러
        function(server_name)
          lspconfig[server_name].setup({
            on_attach = on_attach,
            capabilities = capabilities,
          })
        end,

        -- lua_ls 커스텀 설정
        ["lua_ls"] = function()
          lspconfig.lua_ls.setup({
            on_attach = on_attach,
            capabilities = capabilities,
            settings = {
              Lua = {
                runtime = { version = "LuaJIT" },
                diagnostics = { globals = { "vim" } },
                workspace = {
                  library = vim.api.nvim_get_runtime_file("", true),
                  checkThirdParty = false,
                },
                telemetry = { enable = false },
              },
            },
          })
        end,
      })
    end,
  },
}
```

## LSP 동작 확인

설정을 마쳤다면 실제로 동작하는지 확인해 보자.

### 1. LSP 서버 상태 확인

```vim
:LspInfo
```

현재 버퍼에 연결된 언어 서버 정보를 표시한다. 서버가 정상적으로 연결되었다면 "attached"라고 표시된다.

### 2. 첫 번째 테스트

Lua 파일을 열고 다음을 시도한다:

```lua
-- test.lua
local function greet(name)
  return "Hello, " .. name
end

local result = greet("World")
print(resutl)   -- 의도적 오타
```

- `resutl` 아래에 진단(빨간 밑줄)이 표시되어야 한다
- `greet` 위에서 `K`를 누르면 함수 시그니처가 표시되어야 한다
- `greet` 위에서 `gd`를 누르면 함수 정의로 이동해야 한다
- `greet` 위에서 `gr`을 누르면 참조 위치가 표시되어야 한다

## 실습: LSP 기능 체험

다음 Python 코드를 파일로 만들어 LSP 기능을 연습하자 (`pyright`가 설치되어 있어야 한다):

```python
class Calculator:
    def __init__(self):
        self.history = []

    def add(self, a, b):
        result = a + b
        self.history.append(("add", a, b, result))
        return result

    def subtract(self, a, b):
        result = a - b
        self.history.append(("subtract", a, b, result))
        return result

    def get_history(self):
        return self.history


calc = Calculator()
sum_result = calc.add(10, 20)
diff_result = calc.subtract(30, 15)
print(calc.get_histry())   # 의도적 오타
```

연습 과제:

1. `Calculator` 위에서 `K`를 눌러 호버 문서를 확인한다
2. `calc.add` 에서 `add` 위에 커서를 놓고 `gd`로 정의로 이동한다
3. `<C-o>`로 이전 위치로 돌아온다
4. `add` 위에서 `gr`로 참조를 찾는다
5. `]d`로 오류가 있는 줄(`get_histry`)로 이동한다
6. `<leader>e`로 진단 상세 메시지를 확인한다
7. `Calculator` 위에서 `<leader>rn`으로 `Calc`로 이름을 변경한다
8. `<leader>f`로 파일 전체를 포매팅한다

## 핵심 정리

- **LSP(Language Server Protocol)**는 에디터와 언어 서버 간의 **표준 프로토콜**이다
- Neovim은 **내장 LSP 클라이언트**를 가지고 있다
- **nvim-lspconfig**는 각 언어 서버의 설정을 간소화하고, **mason.nvim**은 서버 설치를 자동화한다
- **mason-lspconfig**가 mason과 lspconfig를 연동하여 설치부터 설정까지 자동화한다
- **핵심 키맵**: `gd`(정의 이동), `gr`(참조 찾기), `K`(호버), `<leader>rn`(이름 변경), `<leader>ca`(코드 액션)
- **진단(diagnostics)**으로 오류와 경고를 실시간으로 확인하고, `[d`/`]d`로 이동한다
- **`vim.lsp.buf.format()`**으로 코드를 포매팅하며, 저장 시 자동 포매팅도 설정할 수 있다
- 각 언어 서버마다 **고유한 설정**(settings)이 있으며, `setup_handlers`에서 서버별로 커스터마이징한다
- **`:LspInfo`**로 현재 버퍼의 LSP 연결 상태를, **`:Mason`**으로 설치된 도구를 확인한다
