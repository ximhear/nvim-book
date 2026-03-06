# Chapter 17: LSP - 언어 서버 프로토콜 (LazyVim)

IDE에서 당연하게 사용하는 기능들이 있다. 함수 정의로 점프하기, 변수의 모든 참조 찾기, 코드 자동 완성, 실시간 오류 표시, 자동 포매팅. 이 모든 것을 가능하게 하는 것이 **LSP(Language Server Protocol)**다. LazyVim은 LSP를 **기본으로 내장**하고 있어서, 최소한의 설정만으로 완전한 개발 환경을 구축할 수 있다.

## LSP란

**LSP(Language Server Protocol)**는 에디터와 언어 서버(language server) 간의 **표준 통신 프로토콜**이다. Microsoft가 VSCode를 위해 개발했으며, 현재는 에디터와 언어에 관계없이 사용되는 공개 표준이다.

### LSP 이전의 세계

```
에디터 A + 언어 X -> 전용 플러그인 A-X
에디터 A + 언어 Y -> 전용 플러그인 A-Y
에디터 B + 언어 X -> 전용 플러그인 B-X
에디터 B + 언어 Y -> 전용 플러그인 B-Y
...
M개의 에디터 x N개의 언어 = MxN개의 플러그인 필요
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

## LazyVim의 LSP 구성

LazyVim은 LSP 관련 플러그인이 **이미 설정되어 있다**. 수동 설정에서 필요했던 `on_attach`, `capabilities`, `setup_handlers` 같은 보일러플레이트가 필요 없다.

### 내장된 플러그인

| 플러그인 | 역할 | 상태 |
|----------|------|------|
| **nvim-lspconfig** | 각 언어 서버의 기본 설정 제공 | 기본 포함 |
| **mason.nvim** | 언어 서버 자동 설치/관리 | 기본 포함 |
| **mason-lspconfig.nvim** | mason과 lspconfig 연동 | 기본 포함 |

> **핵심 차이**: 수동 설정에서는 이 세 플러그인을 직접 설치하고 연동해야 했지만, LazyVim에서는 모두 내장되어 있다. 사용자는 **어떤 서버를 쓸지**만 지정하면 된다.

## LSP 설정 방법

### 방법 1: LazyExtras로 언어 활성화 (권장)

가장 간단한 방법이다. LazyVim은 언어별 통합 패키지(**extras**)를 제공한다. LSP 서버 + 포매터 + 린터 + Treesitter 파서가 한 번에 설정된다.

```vim
:LazyExtras
```

이 명령을 실행하면 사용 가능한 extras 목록이 표시된다. 원하는 언어를 선택하면 된다:

- `lang.python` -- pyright + black + ruff
- `lang.typescript` -- ts_ls + prettier + eslint
- `lang.go` -- gopls + gofumpt + golangci-lint
- `lang.rust` -- rust-analyzer + rustfmt
- `lang.java` -- jdtls
- `lang.json` -- jsonls + prettier

또는 `lua/config/lazy.lua`에서 직접 import할 수 있다:

```lua
require("lazy").setup({
  spec = {
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },

    -- 언어 extras 활성화
    { import = "lazyvim.plugins.extras.lang.python" },
    { import = "lazyvim.plugins.extras.lang.typescript" },
    { import = "lazyvim.plugins.extras.lang.go" },
    { import = "lazyvim.plugins.extras.lang.rust" },

    -- 사용자 플러그인
    { import = "plugins" },
  },
})
```

> **팁**: extras를 활성화하면 해당 언어에 최적화된 설정이 자동으로 적용된다. 대부분의 경우 추가 설정이 필요 없다.

### 방법 2: 서버별 커스텀 설정

특정 서버의 설정을 직접 지정하고 싶다면, `lua/plugins/lsp.lua`를 만든다:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        -- 서버 이름을 키로, 설정을 값으로
        lua_ls = {
          settings = {
            Lua = {
              workspace = { checkThirdParty = false },
              telemetry = { enable = false },
            },
          },
        },

        pyright = {
          settings = {
            python = {
              analysis = {
                typeCheckingMode = "basic",
                autoSearchPaths = true,
                useLibraryCodeForTypes = true,
              },
            },
          },
        },

        gopls = {
          settings = {
            gopls = {
              analyses = { unusedparams = true },
              staticcheck = true,
              gofumpt = true,
            },
          },
        },

        rust_analyzer = {
          settings = {
            ["rust-analyzer"] = {
              checkOnSave = { command = "clippy" },
              cargo = { allFeatures = true },
            },
          },
        },

        ts_ls = {
          settings = {
            typescript = {
              inlayHints = {
                includeInlayParameterNameHints = "all",
                includeInlayFunctionParameterTypeHints = true,
                includeInlayVariableTypeHints = true,
              },
            },
          },
        },
      },
    },
  },
}
```

`opts.servers`에 서버 이름과 설정을 넣기만 하면, LazyVim이 나머지(on_attach, capabilities, mason 연동 등)를 자동으로 처리한다.

### mason으로 도구 자동 설치

mason이 자동으로 설치할 도구를 지정할 수 있다:

```lua
return {
  {
    "williamboman/mason.nvim",
    opts = {
      ensure_installed = {
        "lua-language-server",
        "stylua",
        "prettier",
        "black",
        "ruff",
        "eslint_d",
      },
    },
  },
}
```

> **참고**: `ensure_installed`에는 mason 패키지 이름을 사용한다. lspconfig 서버 이름(`lua_ls`)과 mason 패키지 이름(`lua-language-server`)이 다를 수 있으니 주의한다. `:Mason`에서 정확한 이름을 확인할 수 있다.

## 기본 제공 키맵

LazyVim은 LSP 관련 키맵을 기본으로 제공한다. **별도의 키맵 설정이 필요 없다.**

### 이동

| 키 | 기능 | 비고 |
|----|------|------|
| `gd` | 정의로 이동 (Go to Definition) | 가장 자주 사용 |
| `gD` | 선언으로 이동 (Go to Declaration) | |
| `gr` | 참조 찾기 (Find References) | |
| `gI` | 구현으로 이동 (Go to Implementation) | 수동 설정의 `gi`와 다름 |
| `gy` | 타입 정의로 이동 (Type Definition) | 수동 설정의 `gt`와 다름 |

### 정보 표시

| 키 | 기능 |
|----|------|
| `K` | 호버 문서 (Hover) |
| `gK` | 시그니처 도움말 (Signature Help) |

### 코드 수정

| 키 | 기능 |
|----|------|
| `<leader>cr` | 이름 변경 (Rename) |
| `<leader>ca` | 코드 액션 (Code Action) |
| `<leader>cA` | 소스 코드 액션 (Source Action) |

### 진단

| 키 | 기능 |
|----|------|
| `]d` | 다음 진단으로 이동 |
| `[d` | 이전 진단으로 이동 |
| `]e` | 다음 오류(Error)로 이동 |
| `[e` | 이전 오류(Error)로 이동 |
| `]w` | 다음 경고(Warning)로 이동 |
| `[w` | 이전 경고(Warning)로 이동 |
| `<leader>cd` | 줄 진단 상세 표시 (Line Diagnostics) |

### 포매팅

| 키 | 기능 |
|----|------|
| `<leader>cf` | 포매팅 (Format) |

> **차이점 정리**: 수동 설정에서 `gi`였던 것이 LazyVim에서는 `gI`, `gt`는 `gy`, `<leader>rn`은 `<leader>cr`, `<leader>e`는 `<leader>cd`로 바뀌었다. LazyVim의 키맵은 `<leader>c`(Code) 그룹 아래로 일관되게 정리되어 있다.

## 주요 LSP 기능 활용

### gd - 정의로 이동

커서 아래의 함수나 변수가 **정의된 위치**로 점프한다. 가장 자주 사용하는 LSP 기능이다.

```python
def calculate(x, y):    # <- gd를 누르면 여기로 점프
    return x + y

result = calculate(1, 2)
            ^ 커서를 여기에 놓고 gd
```

`<C-o>`로 이전 위치로 돌아올 수 있다.

### gr - 참조 찾기

커서 아래의 심볼이 **사용된 모든 위치**를 보여준다.

```python
def calculate(x, y):
    return x + y

result = calculate(1, 2)   # <- 참조 1
total = calculate(3, 4)    # <- 참조 2
print(calculate(5, 6))     # <- 참조 3
```

`calculate` 위에서 `gr`을 누르면 세 곳의 참조가 목록으로 표시된다.

### K - 호버 문서

커서 아래 심볼의 **타입 정보와 문서**를 떠 있는 창(floating window)에 표시한다.

```typescript
const items: string[] = ["a", "b", "c"];
items.map(...)
  ^ 여기서 K를 누르면 map의 시그니처와 설명이 표시된다
```

### `<leader>cr` - 이름 변경

변수나 함수의 이름을 **프로젝트 전체에서** 한 번에 변경한다. LSP는 코드의 의미를 이해하므로 동명의 다른 심볼은 건드리지 않는다.

```python
# "user" 위에서 <leader>cr -> "customer"로 입력

# before
def get_user(user_id):
    user = db.find(user_id)
    return user

# after
def get_customer(customer_id):
    customer = db.find(customer_id)
    return customer
```

### `<leader>ca` - 코드 액션

현재 위치에서 가능한 **자동 수정이나 리팩토링**을 제안한다:

- 빠진 import 자동 추가
- 사용하지 않는 변수 제거
- 함수 추출
- 타입 변환

## 진단(Diagnostics)

LazyVim은 진단 표시를 기본으로 설정해 놓았다. 별도의 `vim.diagnostic.config()` 호출이 필요 없다.

### 진단 심각도

| 심각도 | 설명 | 기본 색상 |
|--------|------|-----------|
| ERROR | 코드가 실행되지 않는 오류 | 빨간색 |
| WARN | 잠재적 문제 (경고) | 노란색 |
| INFO | 정보성 메시지 | 파란색 |
| HINT | 개선 제안 | 초록색 |

### 진단 설정 커스터마이징

기본 진단 설정을 변경하고 싶다면:

```lua
-- lua/plugins/lsp.lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      diagnostics = {
        virtual_text = {
          prefix = "icons",   -- "icons", "●", 또는 false
        },
        severity_sort = true,
        underline = true,
        update_in_insert = false,
      },
    },
  },
}
```

## 포매팅

LazyVim은 **conform.nvim**을 포매팅에 사용한다. LSP 포매팅 대신 전용 포매터를 사용하므로 더 일관적인 결과를 얻을 수 있다.

### 기본 동작

- `<leader>cf`로 수동 포매팅
- 저장 시 자동 포매팅 (기본 활성화)

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
        python = { "black" },
        go = { "gofmt" },
        rust = { "rustfmt" },
      },
    },
  },
}
```

### 자동 포매팅 비활성화

자동 포매팅을 끄고 싶다면:

```lua
return {
  {
    "LazyVim/LazyVim",
    opts = {
      autoformat = false,
    },
  },
}
```

또는 특정 버퍼에서만 토글:

```vim
:LazyFormat         " 현재 버퍼 포매팅
:LazyFormatInfo     " 현재 버퍼의 포매터 정보 확인
```

`<leader>uf`로 현재 버퍼의 자동 포매팅을 토글할 수 있다.

## mason.nvim 사용법

mason.nvim은 LazyVim에 내장되어 있다. 별도 설치 없이 바로 사용할 수 있다.

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

## LSP 동작 확인

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

다음 Python 코드를 파일로 만들어 LSP 기능을 연습하자 (extras `lang.python`이 활성화되어 있어야 한다):

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
6. `<leader>cd`로 진단 상세 메시지를 확인한다
7. `Calculator` 위에서 `<leader>cr`로 `Calc`로 이름을 변경한다
8. `<leader>cf`로 파일 전체를 포매팅한다

## 핵심 정리

- **LSP(Language Server Protocol)**는 에디터와 언어 서버 간의 **표준 프로토콜**이다
- LazyVim은 LSP를 **기본으로 내장**하고 있어서 별도의 보일러플레이트 코드가 필요 없다
- **LazyExtras**로 언어별 통합 패키지(LSP + 포매터 + 린터)를 한 번에 활성화할 수 있다
- `opts.servers`에 서버 설정만 넣으면 LazyVim이 나머지를 자동 처리한다
- **핵심 키맵**: `gd`(정의 이동), `gr`(참조 찾기), `K`(호버), `<leader>cr`(이름 변경), `<leader>ca`(코드 액션)
- **진단**: `[d`/`]d`로 이동, `[e`/`]e`로 오류만 이동, `<leader>cd`로 상세 표시
- **포매팅**: conform.nvim이 내장, `<leader>cf`로 수동 포매팅, 저장 시 자동 포매팅
- **`:Mason`**으로 도구 관리, **`:LspInfo`**로 LSP 연결 상태 확인
