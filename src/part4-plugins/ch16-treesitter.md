# Chapter 16: Treesitter - 구문 이해

코드를 편집할 때 에디터가 코드의 **구조**를 이해한다면 어떨까? 함수가 어디서 시작하고 끝나는지, 변수가 어떤 스코프에 속하는지, 어떤 부분이 문자열이고 어떤 부분이 주석인지를 정확히 안다면? Treesitter가 바로 이것을 가능하게 한다.

## Treesitter란

Treesitter는 **파서 생성기(parser generator)**이자 **증분 파싱 라이브러리(incremental parsing library)**다. 원래 GitHub의 Atom 에디터를 위해 개발되었으며, 현재는 Neovim에 내장되어 있다.

핵심 개념은 간단하다. 소스 코드를 읽어서 **구문 트리(Abstract Syntax Tree, AST)**를 만든다. 이 트리를 통해 에디터는 코드의 의미를 "이해"할 수 있다.

```javascript
function greet(name) {
  return "Hello, " + name;
}
```

Treesitter는 이 코드를 다음과 같은 트리로 파싱한다:

```
program
  function_declaration
    name: identifier "greet"
    parameters: formal_parameters
      identifier "name"
    body: statement_block
      return_statement
        binary_expression
          string "Hello, "
          identifier "name"
```

이 트리가 있으면 "함수 이름", "매개변수", "문자열 리터럴" 등을 정확히 구분할 수 있다.

## 기존 정규표현식 하이라이팅 vs Treesitter 하이라이팅

### 기존 방식: 정규표현식(Regex) 기반

Vim의 전통적인 구문 하이라이팅은 정규표현식으로 패턴을 매칭한다.

```vim
" 전통적인 구문 정의 (간소화)
syntax match Function /\<function\>/
syntax region String start=/"/ end=/"/
```

이 방식의 한계:

- **문맥을 모른다**: 주석 안의 `function`도 키워드로 하이라이팅될 수 있다
- **중첩 구조 처리가 어렵다**: 정규표현식은 재귀적 구조를 다루지 못한다
- **언어마다 복잡한 규칙 필요**: 각 언어에 수백 줄의 syntax 파일이 필요하다
- **느리다**: 큰 파일에서 성능 문제가 발생할 수 있다

### Treesitter 방식: 구문 트리 기반

Treesitter는 코드를 파싱하여 정확한 구문 트리를 만든다.

- **문맥을 안다**: 주석 안의 `function`은 "주석"으로 정확히 인식된다
- **중첩 구조를 완벽히 처리한다**: 트리 구조이므로 재귀적 구조가 자연스럽다
- **증분 파싱**: 코드가 변경되면 변경된 부분만 다시 파싱한다 (빠르다)
- **하나의 파서로 여러 기능 제공**: 하이라이팅, 들여쓰기, 선택, 텍스트 오브젝트 등

```
정규표현식:  "function"이라는 글자가 보이면 → 키워드로 색칠
Treesitter:  구문 트리에서 function_declaration 노드를 찾으면 → 키워드로 색칠
```

## nvim-treesitter 설치와 설정

Neovim에 Treesitter 엔진이 내장되어 있지만, 실제로 사용하려면 **nvim-treesitter** 플러그인이 필요하다. 이 플러그인이 언어별 파서 설치와 각종 모듈(하이라이팅, 들여쓰기 등)을 관리한다.

Chapter 15에서 배운 lazy.nvim으로 설치한다.

`lua/plugins/treesitter.lua`:

```lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
    event = { "BufReadPre", "BufNewFile" },
    config = function()
      require("nvim-treesitter.configs").setup({
        ensure_installed = {
          "lua",
          "vim",
          "vimdoc",
          "javascript",
          "typescript",
          "python",
          "go",
          "rust",
          "html",
          "css",
          "json",
          "yaml",
          "markdown",
          "markdown_inline",
          "bash",
          "c",
        },

        -- 자동 설치: 파일을 열 때 파서가 없으면 자동 설치
        auto_install = true,

        -- 구문 하이라이팅
        highlight = {
          enable = true,
        },

        -- 들여쓰기
        indent = {
          enable = true,
        },

        -- 점진적 선택
        incremental_selection = {
          enable = true,
          keymaps = {
            init_selection = "<C-space>",
            node_incremental = "<C-space>",
            scope_incremental = false,
            node_decremental = "<bs>",
          },
        },
      })
    end,
  },
}
```

| 옵션 | 설명 |
|------|------|
| `build = ":TSUpdate"` | 플러그인 설치/업데이트 시 파서도 업데이트 |
| `ensure_installed` | 반드시 설치할 언어 파서 목록 |
| `auto_install = true` | 파서가 없는 파일을 열면 자동 설치 |
| `highlight.enable` | Treesitter 기반 구문 하이라이팅 활성화 |
| `indent.enable` | Treesitter 기반 들여쓰기 활성화 |

## 파서 설치

### 명령으로 설치

```vim
:TSInstall python           " Python 파서 설치
:TSInstall javascript       " JavaScript 파서 설치
:TSInstall all              " 지원하는 모든 파서 설치 (시간 오래 걸림)
```

### 설치된 파서 확인

```vim
:TSInstallInfo              " 설치된 파서 목록과 상태 확인
```

### 파서 업데이트

```vim
:TSUpdate                   " 모든 파서 업데이트
:TSUpdate python            " 특정 파서만 업데이트
```

> **팁**: `ensure_installed`에 사용하는 언어를 넣어두면 Neovim을 새로 설정할 때 자동으로 필요한 파서가 설치된다. `auto_install = true`를 설정하면 목록에 없는 언어의 파일을 열 때도 자동으로 파서를 설치한다.

## 향상된 구문 하이라이팅: highlight 모듈

Treesitter 하이라이팅을 활성화하면 즉시 차이를 느낄 수 있다.

### 전후 비교

기존 정규표현식 하이라이팅에서는:
- 변수와 함수 이름이 같은 색으로 표시될 수 있다
- 문자열 안에 삽입된 표현식이 제대로 하이라이팅되지 않는다
- 복잡한 중첩 구조에서 색이 깨진다

Treesitter 하이라이팅에서는:
- 변수, 함수, 타입, 매개변수가 각각 다른 색으로 정확히 구분된다
- 템플릿 리터럴(template literal) 안의 표현식도 정확히 하이라이팅된다
- 중첩이 아무리 깊어도 정확하다

### 하이라이팅 확인

특정 토큰이 어떤 하이라이트 그룹으로 인식되는지 확인하려면:

```vim
:Inspect
```

커서 아래의 토큰에 대해 Treesitter가 인식한 노드 타입과 적용된 하이라이트 그룹을 보여준다.

## 들여쓰기 개선: indent 모듈

`indent.enable = true`를 설정하면 Treesitter가 구문 트리를 기반으로 들여쓰기를 계산한다.

```python
# Treesitter 들여쓰기가 활성화된 상태에서
# 새 줄을 추가하면 자동으로 올바른 들여쓰기가 적용된다

def process(data):
    if data is not None:
        for item in data:
            |  ← 여기서 Enter를 누르면 자동으로 이 위치에 커서가 온다
```

기존 들여쓰기 방식은 이전 줄의 들여쓰기를 참고하는 단순한 규칙을 사용하지만, Treesitter는 코드의 구조를 파악하여 정확한 들여쓰기 수준을 결정한다.

> **참고**: 일부 언어에서는 Treesitter 들여쓰기가 완벽하지 않을 수 있다. 문제가 발생하면 해당 언어에서만 비활성화할 수 있다.

## Incremental Selection: 점진적 선택 확장/축소

점진적 선택(incremental selection)은 Treesitter의 구문 트리를 활용하여 선택 범위를 **의미 단위로** 확장하거나 축소하는 기능이다.

앞서 설정한 키맵:

| 키 | 동작 |
|----|------|
| `<C-space>` (첫 번째) | 현재 노드 선택 시작 |
| `<C-space>` (반복) | 선택을 상위 노드로 확장 |
| `<BS>` (Backspace) | 선택을 하위 노드로 축소 |

### 실전 예시

```python
result = calculate(user.get_score() + bonus)
                         ↑ 커서 위치
```

`<C-space>`를 반복해서 누르면:

1. `get_score` 선택 (식별자)
2. `get_score()` 선택 (함수 호출)
3. `user.get_score()` 선택 (멤버 접근)
4. `user.get_score() + bonus` 선택 (이항 연산)
5. `calculate(user.get_score() + bonus)` 선택 (함수 호출)
6. `result = calculate(user.get_score() + bonus)` 선택 (대입문)

`<BS>`를 누르면 반대 방향으로 축소된다.

이것이 텍스트 오브젝트와 다른 점은, **코드의 문법 구조를 따라** 선택이 확장된다는 것이다. 괄호, 따옴표 같은 물리적 구분자가 아니라 의미 단위로 선택한다.

## nvim-treesitter-textobjects: 코드 구조를 텍스트 오브젝트로

Chapter 5에서 배운 텍스트 오브젝트를 기억하는가? `iw`(단어 안쪽), `i(`(괄호 안쪽) 같은 것들이었다. **nvim-treesitter-textobjects** 플러그인은 이 개념을 코드의 문법 구조로 확장한다.

### 설치

기존 Treesitter 설정에 의존성으로 추가한다:

`lua/plugins/treesitter.lua` (확장):

```lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = {
      "nvim-treesitter/nvim-treesitter-textobjects",
    },
    config = function()
      require("nvim-treesitter.configs").setup({
        ensure_installed = {
          "lua", "vim", "vimdoc", "javascript", "typescript",
          "python", "go", "rust", "html", "css", "json",
          "yaml", "markdown", "markdown_inline", "bash", "c",
        },
        auto_install = true,
        highlight = { enable = true },
        indent = { enable = true },
        incremental_selection = {
          enable = true,
          keymaps = {
            init_selection = "<C-space>",
            node_incremental = "<C-space>",
            scope_incremental = false,
            node_decremental = "<bs>",
          },
        },

        -- 텍스트 오브젝트
        textobjects = {
          select = {
            enable = true,
            lookahead = true,   -- 커서 앞의 텍스트 오브젝트도 자동 선택
            keymaps = {
              ["af"] = "@function.outer",   -- 함수 전체
              ["if"] = "@function.inner",   -- 함수 본문
              ["ac"] = "@class.outer",      -- 클래스 전체
              ["ic"] = "@class.inner",      -- 클래스 본문
              ["aa"] = "@parameter.outer",  -- 매개변수 (구분자 포함)
              ["ia"] = "@parameter.inner",  -- 매개변수 (값만)
              ["ai"] = "@conditional.outer", -- 조건문 전체
              ["ii"] = "@conditional.inner", -- 조건문 본문
              ["al"] = "@loop.outer",       -- 반복문 전체
              ["il"] = "@loop.inner",       -- 반복문 본문
            },
          },

          -- 코드 구조 간 이동
          move = {
            enable = true,
            set_jumps = true,   -- 점프 리스트에 추가
            goto_next_start = {
              ["]m"] = "@function.outer",    -- 다음 함수 시작
              ["]c"] = "@class.outer",       -- 다음 클래스 시작
              ["]a"] = "@parameter.inner",   -- 다음 매개변수
            },
            goto_next_end = {
              ["]M"] = "@function.outer",    -- 다음 함수 끝
              ["]C"] = "@class.outer",       -- 다음 클래스 끝
            },
            goto_previous_start = {
              ["[m"] = "@function.outer",    -- 이전 함수 시작
              ["[c"] = "@class.outer",       -- 이전 클래스 시작
              ["[a"] = "@parameter.inner",   -- 이전 매개변수
            },
            goto_previous_end = {
              ["[M"] = "@function.outer",    -- 이전 함수 끝
              ["[C"] = "@class.outer",       -- 이전 클래스 끝
            },
          },

          -- 매개변수 위치 교환
          swap = {
            enable = true,
            swap_next = {
              ["<leader>a"] = "@parameter.inner",
            },
            swap_previous = {
              ["<leader>A"] = "@parameter.inner",
            },
          },
        },
      })
    end,
  },
}
```

### 주요 텍스트 오브젝트

| 텍스트 오브젝트 | 대상 | 예시 |
|-----------------|------|------|
| `@function.inner` | 함수 본문 | `dif` - 함수 본문 삭제 |
| `@function.outer` | 함수 전체 (선언 포함) | `daf` - 함수 전체 삭제 |
| `@class.inner` | 클래스 본문 | `vic` - 클래스 본문 선택 |
| `@class.outer` | 클래스 전체 | `dac` - 클래스 전체 삭제 |
| `@parameter.inner` | 매개변수 값 | `cia` - 매개변수 변경 |
| `@parameter.outer` | 매개변수 + 구분자 | `daa` - 매개변수 + 쉼표 삭제 |
| `@conditional.inner` | 조건문 본문 | `dii` - 조건문 본문 삭제 |
| `@loop.inner` | 반복문 본문 | `dil` - 반복문 본문 삭제 |

### 실전 예시: 함수 조작

```python
def calculate_total(items, tax_rate, discount):
    subtotal = sum(item.price for item in items)
    tax = subtotal * tax_rate
    total = subtotal + tax - discount
    return total
```

커서가 함수 안 어디에 있든:

| 명령 | 결과 |
|------|------|
| `daf` | 함수 전체 삭제 (def 줄부터 return까지) |
| `dif` | 함수 본문만 삭제 (def 줄은 남김) |
| `yaf` | 함수 전체 복사 |
| `vif` | 함수 본문 선택 |

### 실전 예시: 매개변수 조작

```python
def calculate_total(items, tax_rate, discount):
                           ↑ 커서 위치
```

| 명령 | 결과 |
|------|------|
| `cia` | `tax_rate`를 다른 이름으로 변경 |
| `daa` | `tax_rate,`를 삭제 (쉼표 포함) |
| `<leader>a` | `tax_rate`와 `discount`의 위치를 교환 |
| `<leader>A` | `tax_rate`와 `items`의 위치를 교환 |

### 이동 키맵

| 키 | 동작 |
|----|------|
| `]m` | 다음 함수 시작으로 이동 |
| `[m` | 이전 함수 시작으로 이동 |
| `]M` | 다음 함수 끝으로 이동 |
| `[M` | 이전 함수 끝으로 이동 |
| `]c` | 다음 클래스 시작으로 이동 |
| `[c` | 이전 클래스 시작으로 이동 |

이 이동 키맵들은 점프 리스트(`set_jumps = true`)에 기록되므로 `<C-o>`로 이전 위치로 돌아갈 수 있다.

## Treesitter Playground: 구문 트리 탐색

코드의 구문 트리를 직접 확인하고 싶을 때가 있다. Neovim 0.10 이상에서는 내장 명령으로 이를 확인할 수 있다.

```vim
:InspectTree
```

이 명령을 실행하면 현재 버퍼의 구문 트리가 새 창에 표시된다. 커서를 이동하면 대응하는 트리 노드가 하이라이팅된다.

```
(program                              | function greet(name) {
  (function_declaration               |   return "Hello, " + name;
    name: (identifier)                | }
    parameters: (formal_parameters    |
      (identifier))                   |
    body: (statement_block            |
      (return_statement               |
        (binary_expression            |
          left: (string)              |
          right: (identifier))))))    |
```

구문 트리를 탐색하면:
- 특정 노드가 어떤 타입으로 인식되는지 확인할 수 있다
- 커스텀 쿼리(query)를 작성할 때 참고할 수 있다
- 하이라이팅이 예상과 다를 때 원인을 파악할 수 있다

## 실전 설정 예제

앞서 다룬 모든 내용을 통합한 완전한 Treesitter 설정이다.

`lua/plugins/treesitter.lua`:

```lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = {
      "nvim-treesitter/nvim-treesitter-textobjects",
    },
    config = function()
      require("nvim-treesitter.configs").setup({
        ensure_installed = {
          "lua", "vim", "vimdoc",
          "javascript", "typescript", "tsx",
          "python", "go", "rust",
          "html", "css", "json", "yaml", "toml",
          "markdown", "markdown_inline",
          "bash", "c", "cpp",
          "gitcommit", "diff",
        },
        auto_install = true,
        highlight = { enable = true },
        indent = { enable = true },

        incremental_selection = {
          enable = true,
          keymaps = {
            init_selection = "<C-space>",
            node_incremental = "<C-space>",
            scope_incremental = false,
            node_decremental = "<bs>",
          },
        },

        textobjects = {
          select = {
            enable = true,
            lookahead = true,
            keymaps = {
              ["af"] = "@function.outer",
              ["if"] = "@function.inner",
              ["ac"] = "@class.outer",
              ["ic"] = "@class.inner",
              ["aa"] = "@parameter.outer",
              ["ia"] = "@parameter.inner",
            },
          },
          move = {
            enable = true,
            set_jumps = true,
            goto_next_start = {
              ["]m"] = "@function.outer",
              ["]c"] = "@class.outer",
            },
            goto_next_end = {
              ["]M"] = "@function.outer",
              ["]C"] = "@class.outer",
            },
            goto_previous_start = {
              ["[m"] = "@function.outer",
              ["[c"] = "@class.outer",
            },
            goto_previous_end = {
              ["[M"] = "@function.outer",
              ["[C"] = "@class.outer",
            },
          },
          swap = {
            enable = true,
            swap_next = {
              ["<leader>a"] = "@parameter.inner",
            },
            swap_previous = {
              ["<leader>A"] = "@parameter.inner",
            },
          },
        },
      })
    end,
  },
}
```

## 실습: Treesitter 기능 체험

다음 코드를 파일로 만들고 Treesitter 기능들을 연습해 보자:

```python
class UserManager:
    def __init__(self, database):
        self.db = database
        self.cache = {}

    def get_user(self, user_id, use_cache=True):
        if use_cache and user_id in self.cache:
            return self.cache[user_id]
        user = self.db.find_by_id(user_id)
        if user is not None:
            self.cache[user_id] = user
        return user

    def create_user(self, name, email, role="member"):
        user = {"name": name, "email": email, "role": role}
        self.db.insert(user)
        return user

    def delete_user(self, user_id):
        if user_id in self.cache:
            del self.cache[user_id]
        self.db.delete(user_id)
```

연습 과제:

1. `:InspectTree`로 구문 트리를 열고 커서를 이동하며 트리 구조를 탐색한다
2. `get_user` 함수 안에서 `vif`로 함수 본문을 선택한다
3. `get_user` 함수 안에서 `daf`로 함수 전체를 삭제한 뒤 `u`로 복원한다
4. `create_user`의 `email` 매개변수에서 `<leader>a`로 `role`과 위치를 교환한다
5. `]m`으로 다음 함수로 이동, `[m`으로 이전 함수로 이동한다
6. `<C-space>`를 반복 눌러 점진적 선택이 어떻게 확장되는지 확인한다
7. `:Inspect`로 커서 아래 토큰의 하이라이트 그룹을 확인한다

## 핵심 정리

- **Treesitter**는 소스 코드를 파싱하여 **구문 트리(AST)**를 만드는 파서 생성기다
- 기존 정규표현식 하이라이팅과 달리, 코드의 **문법 구조를 정확히** 이해한다
- **nvim-treesitter** 플러그인으로 파서 설치와 모듈(하이라이팅, 들여쓰기 등)을 관리한다
- **`:TSInstall`**로 언어별 파서를 설치하고, `ensure_installed`로 자동 설치를 설정한다
- **점진적 선택(incremental selection)**으로 구문 트리를 따라 선택 범위를 확장/축소한다
- **nvim-treesitter-textobjects**로 함수(`af`/`if`), 클래스(`ac`/`ic`), 매개변수(`aa`/`ia`) 등을 텍스트 오브젝트로 사용한다
- **`]m`/`[m`**으로 함수 간 이동, **`<leader>a`**로 매개변수 위치 교환이 가능하다
- **`:InspectTree`**로 구문 트리를 시각적으로 탐색할 수 있다
