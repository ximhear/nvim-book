# Chapter 23: 나만의 플러그인 만들기 (LazyVim)

다른 사람이 만든 플러그인을 설치하고 설정하는 것에 익숙해졌다면, 이제 **직접 플러그인을 만들어 볼** 차례다. Neovim 플러그인은 생각보다 어렵지 않다. Lua 함수 몇 개와 Neovim API에 대한 이해만 있으면 유용한 플러그인을 만들 수 있다.

이 챕터에서는 플러그인의 구조와 규칙을 배우고, 두 개의 실전 플러그인을 처음부터 끝까지 만들어 본다. LazyVim 환경에서의 개발과 테스트 방법도 함께 다룬다.

## Neovim 플러그인의 구조

Neovim 플러그인은 정해진 디렉터리 구조를 따른다.

```
my-plugin.nvim/
├── lua/
│   └── my-plugin/
│       ├── init.lua        -- 플러그인 진입점
│       └── utils.lua       -- 유틸리티 함수
├── plugin/
│   └── my-plugin.lua       -- 자동 로드 스크립트
└── README.md
```

각 디렉터리의 역할:

| 디렉터리 | 역할 |
|----------|------|
| `lua/my-plugin/` | 플러그인의 핵심 로직. `require("my-plugin")`으로 로드된다 |
| `plugin/` | Neovim이 시작될 때 자동으로 실행되는 스크립트 |
| `README.md` | 사용법, 설치 방법 등 문서 |

### lua/ 디렉터리

`lua/` 아래의 파일은 Lua의 `require` 시스템으로 로드된다:

```lua
-- lua/my-plugin/init.lua 파일은:
require("my-plugin")          -- 이렇게 로드된다

-- lua/my-plugin/utils.lua 파일은:
require("my-plugin.utils")    -- 이렇게 로드된다
```

### plugin/ 디렉터리

`plugin/` 아래의 `.lua` 파일은 Neovim이 시작될 때 자동으로 실행된다. 플러그인의 명령어 등록, 자동 명령 설정 등 "한 번만 실행해야 하는" 초기화 코드를 여기에 둔다.

```lua
-- plugin/my-plugin.lua
-- Neovim 시작 시 자동 실행
vim.api.nvim_create_user_command("MyPluginRun", function()
  require("my-plugin").run()
end, {})
```

## setup 패턴: M.setup(opts) 컨벤션

현대 Neovim 플러그인의 사실상 표준은 `setup()` 패턴이다. 사용자가 플러그인을 설정할 때 `require("plugin").setup(opts)`를 호출하는 방식이다.

```lua
-- lua/my-plugin/init.lua

local M = {}

-- 기본 설정값
M.config = {
  greeting = "Hello",
  show_time = true,
}

function M.setup(opts)
  -- 사용자 설정을 기본값과 병합
  M.config = vim.tbl_deep_extend("force", M.config, opts or {})
end

function M.greet()
  local msg = M.config.greeting .. ", Neovim!"
  if M.config.show_time then
    msg = msg .. " (" .. os.date("%H:%M") .. ")"
  end
  vim.notify(msg)
end

return M
```

### LazyVim에서 사용

LazyVim에서는 `opts`를 전달하면 자동으로 `setup()`이 호출된다:

```lua
-- lua/plugins/my-plugin.lua
return {
  {
    "username/my-plugin.nvim",
    opts = {
      greeting = "Hi",
      show_time = false,
    },
  },
}
```

`vim.tbl_deep_extend("force", defaults, opts)`는 기본값에 사용자 설정을 덮어씌운다. 사용자가 지정하지 않은 항목은 기본값이 유지된다.

## 사용자 명령 등록

`vim.api.nvim_create_user_command`로 `:MyCommand` 형태의 사용자 명령을 등록한다.

```lua
vim.api.nvim_create_user_command("Greet", function(opts)
  local name = opts.args
  if name == "" then
    name = "World"
  end
  vim.notify("Hello, " .. name .. "!")
end, {
  nargs = "?",     -- 인자 0개 또는 1개
  desc = "인사 메시지 출력",
})
```

| nargs 값 | 의미 |
|----------|------|
| `"0"` | 인자 없음 |
| `"1"` | 정확히 1개 |
| `"?"` | 0개 또는 1개 |
| `"*"` | 0개 이상 |
| `"+"` | 1개 이상 |

자동 완성을 추가할 수도 있다:

```lua
vim.api.nvim_create_user_command("SetLang", function(opts)
  vim.bo.filetype = opts.args
end, {
  nargs = 1,
  complete = function()
    return { "lua", "python", "javascript", "typescript", "rust", "go" }
  end,
  desc = "파일 타입 설정",
})
```

## 키맵 등록

`vim.keymap.set`으로 플러그인의 기능을 키에 연결한다.

```lua
-- setup() 안에서 키맵 등록
function M.setup(opts)
  M.config = vim.tbl_deep_extend("force", M.config, opts or {})

  if M.config.keymaps then
    vim.keymap.set("n", M.config.keymaps.greet or "<leader>hh",
      M.greet, { desc = "인사 메시지" })
  end
end
```

LazyVim에서는 `keys` 옵션으로도 키맵을 등록할 수 있다 (지연 로딩과 함께):

```lua
return {
  {
    "username/my-plugin.nvim",
    keys = {
      { "<leader>hh", function() require("my-plugin").greet() end, desc = "인사 메시지" },
    },
    opts = {},
  },
}
```

## 하이라이트 그룹 설정

플러그인에서 사용하는 색상을 하이라이트 그룹으로 정의한다.

```lua
vim.api.nvim_set_hl(0, "MyPluginTitle", { fg = "#61afef", bold = true })
vim.api.nvim_set_hl(0, "MyPluginBorder", { fg = "#5c6370" })
vim.api.nvim_set_hl(0, "MyPluginNormal", { link = "Normal" })
```

| 매개변수 | 설명 |
|----------|------|
| `fg` | 전경색 (텍스트 색상) |
| `bg` | 배경색 |
| `bold` | 굵은 글씨 |
| `italic` | 기울임 |
| `underline` | 밑줄 |
| `link` | 다른 하이라이트 그룹에 연결 |

## Neovim API 활용

플러그인 개발의 핵심은 Neovim API를 이해하는 것이다.

### 버퍼 조작

```lua
-- 현재 버퍼의 모든 줄 가져오기
local lines = vim.api.nvim_buf_get_lines(0, 0, -1, false)
-- 0은 현재 버퍼, -1은 마지막 줄까지

-- 특정 범위의 줄 가져오기 (0-indexed)
local first_five = vim.api.nvim_buf_get_lines(0, 0, 5, false)

-- 버퍼에 줄 쓰기
vim.api.nvim_buf_set_lines(0, 0, 0, false, { "첫 번째 줄", "두 번째 줄" })

-- 버퍼 끝에 줄 추가
vim.api.nvim_buf_set_lines(0, -1, -1, false, { "마지막에 추가" })

-- 특정 줄 교체
vim.api.nvim_buf_set_lines(0, 2, 3, false, { "새로운 세 번째 줄" })
```

### 창 조작: floating window

floating window는 화면 위에 떠 있는 창으로, 팝업 UI를 만들 때 사용한다.

```lua
-- 빈 버퍼 생성
local buf = vim.api.nvim_create_buf(false, true)

-- 버퍼에 내용 설정
vim.api.nvim_buf_set_lines(buf, 0, -1, false, {
  "  플러그인 정보  ",
  "",
  "  버전: 1.0.0",
  "  작성자: You",
})

-- floating window 열기
local width = 30
local height = 6
local win = vim.api.nvim_open_win(buf, true, {
  relative = "editor",
  width = width,
  height = height,
  col = (vim.o.columns - width) / 2,
  row = (vim.o.lines - height) / 2,
  style = "minimal",
  border = "rounded",
})

-- q로 닫기
vim.keymap.set("n", "q", function()
  vim.api.nvim_win_close(win, true)
end, { buffer = buf })
```

`nvim_open_win`의 주요 옵션:

| 옵션 | 설명 |
|------|------|
| `relative` | `"editor"`, `"win"`, `"cursor"` 중 선택 |
| `width`, `height` | 창 크기 |
| `col`, `row` | 위치 |
| `border` | `"none"`, `"single"`, `"double"`, `"rounded"`, `"shadow"` |
| `style` | `"minimal"`로 줄 번호 등 숨김 |
| `title` | 창 제목 (Neovim 0.9 이상) |

### 네임스페이스

네임스페이스는 플러그인이 추가한 하이라이트, extmark 등을 구분하는 고유 ID다.

```lua
-- 네임스페이스 생성
local ns = vim.api.nvim_create_namespace("my-plugin")

-- 네임스페이스를 이용한 하이라이트 추가
vim.api.nvim_buf_add_highlight(buf, ns, "MyPluginTitle", 0, 0, -1)

-- 네임스페이스의 모든 하이라이트 제거
vim.api.nvim_buf_clear_namespace(buf, ns, 0, -1)
```

## 실전 플러그인 예제 1: 단어 수 표시 플러그인

현재 버퍼의 통계(줄 수, 단어 수, 문자 수)를 floating window에 표시하는 플러그인을 만들어 보자.

### 디렉터리 구조

```
wordcount.nvim/
├── lua/
│   └── wordcount/
│       └── init.lua
└── plugin/
    └── wordcount.lua
```

### 핵심 코드: lua/wordcount/init.lua

```lua
local M = {}

M.config = {
  border = "rounded",
  width = 40,
}

function M.setup(opts)
  M.config = vim.tbl_deep_extend("force", M.config, opts or {})
end

-- 버퍼 통계 계산
local function get_stats()
  local lines = vim.api.nvim_buf_get_lines(0, 0, -1, false)
  local line_count = #lines
  local word_count = 0
  local char_count = 0

  for _, line in ipairs(lines) do
    for _ in line:gmatch("%S+") do
      word_count = word_count + 1
    end
    char_count = char_count + #line
  end

  return {
    lines = line_count,
    words = word_count,
    chars = char_count,
    filetype = vim.bo.filetype ~= "" and vim.bo.filetype or "plain text",
    filename = vim.fn.expand("%:t"),
  }
end

function M.show()
  local stats = get_stats()
  local filename = stats.filename ~= "" and stats.filename or "[No Name]"

  local content = {
    "  " .. filename,
    "  " .. string.rep("-", M.config.width - 4),
    "",
    "    줄 수 (Lines):     " .. stats.lines,
    "    단어 수 (Words):   " .. stats.words,
    "    문자 수 (Chars):   " .. stats.chars,
    "    파일 타입:          " .. stats.filetype,
    "",
    "  [q] 닫기",
  }

  local buf = vim.api.nvim_create_buf(false, true)
  vim.api.nvim_buf_set_lines(buf, 0, -1, false, content)
  vim.bo[buf].modifiable = false

  local height = #content
  local width = M.config.width

  local win = vim.api.nvim_open_win(buf, true, {
    relative = "editor",
    width = width,
    height = height,
    col = (vim.o.columns - width) / 2,
    row = (vim.o.lines - height) / 2,
    style = "minimal",
    border = M.config.border,
    title = " Word Count ",
    title_pos = "center",
  })

  local ns = vim.api.nvim_create_namespace("wordcount")
  vim.api.nvim_buf_add_highlight(buf, ns, "Title", 0, 0, -1)
  vim.api.nvim_buf_add_highlight(buf, ns, "Comment", #content - 1, 0, -1)

  local function close()
    if vim.api.nvim_win_is_valid(win) then
      vim.api.nvim_win_close(win, true)
    end
  end

  vim.keymap.set("n", "q", close, { buffer = buf })
  vim.keymap.set("n", "<Esc>", close, { buffer = buf })
end

return M
```

### 자동 로드 스크립트: plugin/wordcount.lua

```lua
vim.api.nvim_create_user_command("WordCount", function()
  require("wordcount").show()
end, { desc = "단어 수 표시" })
```

### LazyVim에서 사용

```lua
-- lua/plugins/wordcount.lua
return {
  {
    dir = "~/projects/wordcount.nvim",   -- 로컬 개발 시
    opts = {
      border = "double",
      width = 45,
    },
    keys = {
      { "<leader>wc", "<cmd>WordCount<CR>", desc = "단어 수 표시" },
    },
  },
}
```

## 실전 플러그인 예제 2: 빠른 메모 플러그인

프로젝트별로 메모 파일을 토글하는 플러그인이다. 키 하나로 메모 파일을 열고 닫을 수 있다.

### 디렉터리 구조

```
quicknote.nvim/
├── lua/
│   └── quicknote/
│       └── init.lua
└── plugin/
    └── quicknote.lua
```

### 핵심 코드: lua/quicknote/init.lua

```lua
local M = {}

M.config = {
  note_file = ".notes.md",
  width = 0.6,
  height = 0.6,
  border = "rounded",
}

local state = {
  win = nil,
  buf = nil,
}

function M.setup(opts)
  M.config = vim.tbl_deep_extend("force", M.config, opts or {})
end

local function find_project_root()
  local markers = { ".git", "package.json", "Cargo.toml", "go.mod", "pyproject.toml" }
  local current = vim.fn.expand("%:p:h")

  for _, marker in ipairs(markers) do
    local root = vim.fs.find(marker, {
      path = current,
      upward = true,
      stop = vim.env.HOME,
    })
    if #root > 0 then
      return vim.fn.fnamemodify(root[1], ":h")
    end
  end

  return vim.fn.getcwd()
end

local function get_note_path()
  local root = find_project_root()
  return root .. "/" .. M.config.note_file
end

local function is_open()
  return state.win and vim.api.nvim_win_is_valid(state.win)
end

local function close()
  if is_open() then
    if state.buf and vim.api.nvim_buf_is_valid(state.buf) then
      vim.api.nvim_buf_call(state.buf, function()
        if vim.bo.modified then
          vim.cmd("silent write")
        end
      end)
    end
    vim.api.nvim_win_close(state.win, true)
    state.win = nil
  end
end

local function open()
  local note_path = get_note_path()

  if state.buf and vim.api.nvim_buf_is_valid(state.buf) then
    -- 기존 버퍼 사용
  else
    state.buf = vim.fn.bufadd(note_path)
    vim.fn.bufload(state.buf)
    vim.bo[state.buf].buflisted = false
  end

  local width = math.floor(vim.o.columns * M.config.width)
  local height = math.floor(vim.o.lines * M.config.height)

  state.win = vim.api.nvim_open_win(state.buf, true, {
    relative = "editor",
    width = width,
    height = height,
    col = math.floor((vim.o.columns - width) / 2),
    row = math.floor((vim.o.lines - height) / 2),
    border = M.config.border,
    title = " Quick Note: " .. vim.fn.fnamemodify(note_path, ":t") .. " ",
    title_pos = "center",
  })

  vim.wo[state.win].wrap = true
  vim.wo[state.win].linebreak = true

  vim.keymap.set("n", "q", close, { buffer = state.buf, desc = "메모 닫기" })
end

function M.toggle()
  if is_open() then
    close()
  else
    open()
  end
end

return M
```

### 자동 로드 스크립트: plugin/quicknote.lua

```lua
vim.api.nvim_create_user_command("QuickNote", function()
  require("quicknote").toggle()
end, { desc = "빠른 메모 토글" })
```

### LazyVim에서 사용

```lua
-- lua/plugins/quicknote.lua
return {
  {
    dir = "~/projects/quicknote.nvim",
    opts = {
      note_file = ".notes.md",
      width = 0.7,
      height = 0.5,
    },
    keys = {
      { "<leader>nn", "<cmd>QuickNote<CR>", desc = "빠른 메모 토글" },
    },
  },
}
```

`<leader>nn`을 누르면 프로젝트 루트에 `.notes.md` 파일이 floating window로 열린다. 메모를 작성하고 다시 `<leader>nn`을 누르면 자동 저장 후 닫힌다.

## 테스트: plenary.nvim을 이용한 테스트

plenary.nvim은 Neovim 플러그인 테스트를 위한 프레임워크를 제공한다.

### 테스트 파일 구조

```
my-plugin.nvim/
├── lua/
│   └── my-plugin/
│       └── init.lua
└── tests/
    └── my-plugin_spec.lua    -- _spec.lua로 끝나야 한다
```

### 테스트 작성

```lua
-- tests/my-plugin_spec.lua

describe("my-plugin", function()
  local plugin = require("my-plugin")

  before_each(function()
    plugin.setup({})
  end)

  it("기본 설정이 올바르게 적용된다", function()
    assert.equals("Hello", plugin.config.greeting)
    assert.is_true(plugin.config.show_time)
  end)

  it("사용자 설정으로 기본값을 덮어쓴다", function()
    plugin.setup({ greeting = "Hi" })
    assert.equals("Hi", plugin.config.greeting)
    assert.is_true(plugin.config.show_time)  -- 지정하지 않은 값은 유지
  end)

  describe("get_stats", function()
    it("빈 버퍼의 통계를 올바르게 계산한다", function()
      local buf = vim.api.nvim_create_buf(false, true)
      vim.api.nvim_set_current_buf(buf)
      vim.api.nvim_buf_set_lines(buf, 0, -1, false, {
        "Hello World",
        "This is a test",
      })

      local stats = plugin.get_stats()
      assert.equals(2, stats.lines)
      assert.equals(6, stats.words)

      vim.api.nvim_buf_delete(buf, { force = true })
    end)
  end)
end)
```

### 테스트 실행

```bash
# 단일 파일 테스트
nvim --headless -c "PlenaryBustedFile tests/my-plugin_spec.lua"

# 디렉터리의 모든 테스트 실행
nvim --headless -c "PlenaryBustedDirectory tests/ {minimal_init = 'tests/minimal_init.lua'}"
```

```lua
-- tests/minimal_init.lua
vim.opt.rtp:append(".")
vim.opt.rtp:append("~/.local/share/nvim/lazy/plenary.nvim")
```

## 배포: GitHub에 올리고 설치하기

### 저장소 준비

```bash
cd ~/projects/my-plugin.nvim
git init
git add .
git commit -m "Initial commit"
```

GitHub에 저장소를 만들고 푸시한다:

```bash
git remote add origin https://github.com/username/my-plugin.nvim.git
git push -u origin main
```

### 다른 사람이 LazyVim에서 설치하는 방법

```lua
-- lua/plugins/my-plugin.lua
return {
  {
    "username/my-plugin.nvim",
    opts = {
      -- 사용자 설정
    },
  },
}
```

### 좋은 플러그인이 갖추어야 할 것들

| 항목 | 설명 |
|------|------|
| **README.md** | 설치 방법, 설정 옵션, 사용 예시 |
| **setup() 패턴** | 사용자가 옵션을 전달할 수 있는 표준 인터페이스 |
| **기본값** | 설정 없이도 동작하는 합리적인 기본값 |
| **타입 어노테이션** | LuaLS 타입 주석으로 자동 완성 지원 |
| **테스트** | 핵심 기능의 정상 동작을 보장 |
| **LICENSE** | MIT 등 오픈 소스 라이선스 |

### 로컬 개발 시 팁

플러그인을 개발하는 동안에는 `dir` 옵션으로 로컬 경로를 지정한다:

```lua
{
  dir = "~/projects/my-plugin.nvim",
  opts = {},
}
```

코드를 수정한 뒤 변경 사항을 바로 반영하려면:

```vim
:Lazy reload my-plugin.nvim
```

## 커뮤니티 참여

### awesome-neovim

[awesome-neovim](https://github.com/rockerBOO/awesome-neovim)은 Neovim 플러그인을 카테고리별로 정리한 목록이다. 플러그인을 만들기 전에 이미 비슷한 것이 있는지 확인하고, 만든 플러그인을 홍보하는 창구로도 활용할 수 있다.

### 플러그인 홍보

1. **awesome-neovim에 PR 보내기**: 카테고리에 맞는 섹션에 플러그인을 추가한다
2. **Reddit r/neovim**: 플러그인을 소개하는 글을 올린다
3. **GitHub 태그**: `neovim`, `neovim-plugin`, `lua` 태그를 추가한다
4. **스크린샷/GIF**: 시각적으로 동작을 보여주는 것이 효과적이다

## 실습: 플러그인 만들기

**연습 1: 단어 수 플러그인 구현**

1. `~/projects/wordcount.nvim/` 디렉터리에 단어 수 플러그인을 구현한다
2. LazyVim의 `lua/plugins/` 에 `dir` 옵션으로 등록한다
3. `:WordCount` 명령으로 floating window에 통계를 표시한다
4. `q`로 창을 닫는 기능을 추가한다

**연습 2: 빠른 메모 플러그인 구현**

1. `~/projects/quicknote.nvim/` 디렉터리에 빠른 메모 플러그인을 구현한다
2. `<leader>nn`으로 프로젝트별 메모 파일을 토글한다
3. 닫을 때 자동 저장이 동작하는지 확인한다
4. `:Lazy reload quicknote.nvim`으로 코드 변경 후 바로 테스트한다

**연습 3: 나만의 플러그인 아이디어**

다음 중 하나를 직접 구현해 보자:

- **TODO 하이라이터**: 주석의 `TODO`, `FIXME`, `HACK`에 자동으로 색상을 입힌다
- **빠른 터미널**: 자주 쓰는 셸 명령을 floating window에서 실행한다
- **세션 북마크**: 현재 열린 파일 목록을 저장하고 복원한다

## 핵심 정리

- 플러그인 구조: `lua/` (핵심 로직) + `plugin/` (자동 로드) 디렉터리
- **`M.setup(opts)`** 패턴이 현대 Neovim 플러그인의 표준이다. LazyVim에서는 `opts`를 전달하면 자동으로 호출된다
- **`nvim_create_user_command`**로 `:명령어`를 등록하고, **`vim.keymap.set`** 또는 lazy.nvim의 `keys`로 키맵을 연결한다
- **버퍼 API**: `nvim_buf_get_lines`/`nvim_buf_set_lines`로 버퍼 내용을 읽고 쓴다
- **Floating window**: `nvim_open_win`으로 팝업 UI를 만든다
- **네임스페이스**: `nvim_create_namespace`로 플러그인의 하이라이트를 분리한다
- **로컬 개발**: `dir` 옵션으로 로컬 경로를 지정하고, `:Lazy reload`로 변경 사항을 반영한다
- **테스트**: plenary.nvim의 `describe`/`it` 구문으로 테스트를 작성한다
- **배포**: GitHub에 올리면 `"username/plugin.nvim"`으로 누구나 설치할 수 있다
