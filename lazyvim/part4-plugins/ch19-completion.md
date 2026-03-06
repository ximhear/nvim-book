# Chapter 19: 자동 완성 (LazyVim)

코드를 작성할 때 변수명, 함수명, 파일 경로를 매번 처음부터 끝까지 타이핑하는 것은 비효율적이고 오타의 원인이 된다. 자동 완성(auto-completion)은 몇 글자만 입력하면 나머지를 제안해주는 기능으로, 현대 개발 환경에서 빠질 수 없는 핵심 기능이다.

LazyVim에는 자동 완성이 **완전히 설정된 상태**로 포함되어 있다. 별도의 플러그인 설치나 키맵 설정 없이 바로 사용할 수 있다.

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

## LazyVim의 자동 완성 구성

LazyVim에는 다음이 이미 설정되어 있다:

| 구성 요소 | 플러그인 | 상태 |
|-----------|----------|------|
| 완성 엔진 | nvim-cmp | 기본 포함 |
| LSP 소스 | cmp-nvim-lsp | 기본 포함 |
| 버퍼 소스 | cmp-buffer | 기본 포함 |
| 경로 소스 | cmp-path | 기본 포함 |
| 스니펫 엔진 | LuaSnip | 기본 포함 |
| 스니펫 소스 | cmp_luasnip | 기본 포함 |
| 스니펫 모음 | friendly-snippets | 기본 포함 |
| 아이콘 | mini.icons 또는 nvim-web-devicons | 기본 포함 |

> **핵심 차이**: 수동 설정에서는 이 모든 플러그인을 하나씩 설치하고 config 함수로 연결해야 했지만, LazyVim에서는 모두 내장되어 있다.

## 기본 제공 키맵

LazyVim은 완성 관련 키맵을 기본으로 설정해 놓았다.

### 완성 메뉴 조작

| 키 | 동작 |
|----|------|
| `<C-n>` | 다음 완성 항목 |
| `<C-p>` | 이전 완성 항목 |
| `<C-b>` | 문서 위로 스크롤 |
| `<C-f>` | 문서 아래로 스크롤 |
| `<C-Space>` | 완성 메뉴 수동 호출 |
| `<C-e>` | 완성 취소 |
| `<CR>` | 선택 확정 |

### 스니펫 이동

| 키 | 동작 |
|----|------|
| `<Tab>` | 다음 스니펫 입력 지점으로 이동 |
| `<S-Tab>` | 이전 스니펫 입력 지점으로 이동 |

> **팁**: `<CR>`로 확정할 때, 항목을 명시적으로 선택한 경우에만 적용된다. 단순히 엔터를 눌렀을 때 의도치 않게 첫 번째 항목이 입력되는 것을 방지한다.

## 소스(Source) 이해

### cmp-nvim-lsp: LSP 기반 완성

Chapter 17에서 설정한 LSP 서버가 제공하는 완성 후보를 가져온다. 함수 시그니처, 타입 정보, 자동 임포트 등 지능적인 완성이 가능하다. LazyVim이 LSP 서버에 capabilities를 자동으로 전달하므로 별도 설정이 필요 없다.

### cmp-buffer: 현재 버퍼 단어 완성

현재 열려 있는 버퍼의 단어를 완성 후보로 제공한다. LSP가 지원되지 않는 파일에서도 동작하므로 범용적이다.

### cmp-path: 파일 경로 완성

파일 시스템 경로를 자동 완성한다. `./`이나 `/`를 입력하면 디렉터리와 파일 목록이 나타난다. 설정 파일이나 import 경로를 작성할 때 유용하다.

### 소스 우선순위

LazyVim은 소스 우선순위를 이미 설정해 놓았다:

1. **LSP** -- 가장 높은 우선순위
2. **스니펫** -- LSP 다음
3. **버퍼 단어** -- LSP/스니펫이 없을 때
4. **경로** -- 경로 입력 시

LSP 결과가 있을 때는 LSP와 스니펫이 표시되고, LSP가 없는 파일에서는 버퍼 단어가 표시된다.

## 스니펫

### friendly-snippets: 미리 정의된 스니펫 모음

LazyVim에는 `friendly-snippets`가 기본 포함되어 있다. 다양한 언어에 대한 VS Code 형식 스니펫을 바로 사용할 수 있다.

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

### 스니펫 사용 흐름

1. 축약어(예: `fn`)를 입력하면 완성 메뉴에 스니펫이 나타난다
2. `<CR>`로 스니펫을 선택하면 확장된다
3. `<Tab>`으로 다음 입력 지점으로 이동
4. `<S-Tab>`으로 이전 입력 지점으로 돌아가기
5. 모든 입력 지점을 채우면 스니펫 완료

### 커스텀 스니펫 작성

LuaSnip에서 직접 스니펫을 만들 수 있다:

```lua
-- lua/plugins/snippets.lua
return {
  {
    "L3MON4D3/LuaSnip",
    config = function(_, opts)
      -- LazyVim의 기본 설정을 유지하면서 스니펫 추가
      local ls = require("luasnip")
      local s = ls.snippet
      local t = ls.text_node
      local i = ls.insert_node

      ls.add_snippets("lua", {
        s("req", {
          t('local '),
          i(1, "module"),
          t(' = require("'),
          i(2, "module"),
          t('")'),
        }),
      })
    end,
  },
}
```

위 스니펫의 동작:

1. `req`를 입력하고 스니펫 확장
2. `local ▌module = require("module")` -- 첫 번째 `module`에 커서
3. 이름을 입력하고 `<Tab>` -- 두 번째 `module`로 이동
4. 경로를 입력하고 `<Tab>` -- 스니펫 완료

`i(1)`, `i(2)`는 탭으로 이동하는 **입력 지점(insert node)**이다. 숫자가 이동 순서를 결정한다.

## 커스터마이징

### 소스 추가/변경

기본 소스 구성을 변경하고 싶다면:

```lua
-- lua/plugins/completion.lua
return {
  {
    "hrsh7th/nvim-cmp",
    opts = function(_, opts)
      local cmp = require("cmp")

      -- 소스 순서 변경
      opts.sources = cmp.config.sources({
        { name = "nvim_lsp" },
        { name = "luasnip" },
        { name = "path" },
      }, {
        { name = "buffer" },
      })
    end,
  },
}
```

### 완성 창 스타일 변경

```lua
return {
  {
    "hrsh7th/nvim-cmp",
    opts = function(_, opts)
      local cmp = require("cmp")

      opts.window = {
        completion = cmp.config.window.bordered(),
        documentation = cmp.config.window.bordered(),
      }
    end,
  },
}
```

### 키맵 변경

기본 키맵을 변경하고 싶다면:

```lua
return {
  {
    "hrsh7th/nvim-cmp",
    opts = function(_, opts)
      local cmp = require("cmp")

      opts.mapping = vim.tbl_extend("force", opts.mapping, {
        -- Tab으로 완성 항목 이동 추가
        ["<Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_next_item()
          else
            fallback()
          end
        end, { "i", "s" }),
      })
    end,
  },
}
```

> **주의**: `opts`를 함수로 전달하면 LazyVim의 기본 설정 위에 병합(merge)된다. 기본 설정을 완전히 덮어쓰지 않으려면 `vim.tbl_extend`를 사용한다.

## blink.cmp (대안)

LazyVim은 nvim-cmp 외에 **blink.cmp**라는 대안도 extras로 제공한다. blink.cmp는 Rust로 작성되어 성능이 뛰어나다.

```vim
:LazyExtras
```

에서 `coding.blink` extras를 활성화하면 nvim-cmp 대신 blink.cmp를 사용할 수 있다.

또는 `lua/config/lazy.lua`에서:

```lua
{ import = "lazyvim.plugins.extras.coding.blink" },
```

blink.cmp를 사용해도 키맵과 사용법은 거의 동일하다.

## 실습: 자동 완성 체험

1. Lua 파일을 열고 `vim.`을 입력해 보자 -- LSP 기반 완성이 나타나야 한다
2. 완성 메뉴에서 `<C-n>`과 `<C-p>`로 항목을 오갈 수 있는지 확인하자
3. `<C-Space>`로 완성 메뉴를 수동으로 호출해 보자
4. `<C-e>`로 완성을 취소해 보자
5. `fn`을 입력하고 스니펫이 나타나면 `<CR>`로 확장해 보자
6. 스니펫 확장 후 `<Tab>`으로 다음 입력 지점으로 이동해 보자
7. `./`를 입력하여 파일 경로 완성을 시도해 보자
8. 완성 항목 옆의 아이콘과 소스 표시를 확인해 보자

## 핵심 정리

- LazyVim에는 자동 완성이 **완전히 설정된 상태**로 포함되어 있다 (nvim-cmp + LuaSnip + 소스들)
- **별도의 설치나 키맵 설정이 필요 없다**
- 핵심 소스: **cmp-nvim-lsp**(LSP), **cmp-buffer**(버퍼 단어), **cmp-path**(경로)
- 키맵: `<C-n>`/`<C-p>`로 항목 이동, `<CR>`로 확정, `<C-Space>`로 수동 호출, `<C-e>`로 취소
- **LuaSnip** + **friendly-snippets**가 내장되어 미리 정의된 스니펫을 바로 사용할 수 있다
- `<Tab>`/`<S-Tab>`으로 스니펫 입력 지점 사이를 이동한다
- **소스 우선순위**: LSP > 스니펫 > 버퍼 단어 > 경로 (이미 설정됨)
- 커스터마이징은 `opts` 함수로 기본 설정 위에 병합하는 방식으로 한다
- **blink.cmp** extras를 활성화하면 더 빠른 대안 엔진을 사용할 수 있다
