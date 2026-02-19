# Chapter 18: Telescope - 퍼지 파인더

프로젝트 안에서 원하는 파일을 찾거나, 특정 문자열이 포함된 코드를 검색하거나, 열린 버퍼를 전환하는 일은 개발 중 끊임없이 반복된다. Telescope는 이 모든 "찾기" 작업을 하나의 인터페이스로 통합하는 Neovim용 **퍼지 파인더(fuzzy finder)**다.

"퍼지"란 정확히 일치하지 않아도 된다는 뜻이다. `init`이라고 입력하면 `init.lua`, `initialize.ts`, `config_init.py` 등이 모두 매칭된다. 타이핑 몇 글자만으로 수천 개의 파일 중 원하는 것을 찾아낼 수 있다.

## Telescope란 무엇인가

Telescope는 Neovim 생태계에서 가장 인기 있는 퍼지 파인더 플러그인이다. 핵심 특징은 다음과 같다:

- **고도로 확장 가능**: Picker, Sorter, Previewer를 자유롭게 커스터마이징할 수 있다
- **통합 인터페이스**: 파일, 버퍼, Git, LSP, 도움말 등 모든 것을 같은 UI로 탐색
- **미리보기**: 선택하기 전에 파일 내용을 미리 볼 수 있다
- **Lua 기반**: Neovim의 Lua API를 활용하여 빠르고 유연하다

## 설치와 의존성

Telescope를 제대로 사용하려면 몇 가지 의존성이 필요하다:

| 의존성 | 역할 | 필수 여부 |
|--------|------|-----------|
| `plenary.nvim` | Lua 유틸리티 라이브러리 | 필수 |
| `ripgrep` (rg) | 파일 내용 검색에 사용 | 강력 권장 |
| `fd` | 파일 이름 검색에 사용 | 선택 (없으면 find 사용) |

먼저 외부 도구를 설치한다:

```bash
# macOS
brew install ripgrep fd

# Ubuntu/Debian
sudo apt install ripgrep fd-find

# Arch Linux
sudo pacman -S ripgrep fd
```

### lazy.nvim으로 설치

```lua
-- plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  tag = "0.1.8",
  dependencies = {
    "nvim-lua/plenary.nvim",
  },
  config = function()
    require("telescope").setup({})
  end,
}
```

이것만으로도 Telescope를 사용할 수 있다. 더 세부적인 설정은 아래에서 다룬다.

## 핵심 Picker

Telescope에서 **Picker**는 "무엇을 검색할 것인가"를 정의하는 단위다. 내장 Picker만으로도 대부분의 검색 작업을 처리할 수 있다.

### 파일 검색

#### find_files: 파일 이름으로 검색

프로젝트에서 파일을 찾을 때 가장 많이 사용하는 Picker다.

```vim
:Telescope find_files
```

```lua
-- 키맵 설정
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<cr>", { desc = "파일 찾기" })
```

검색창에 `init`을 입력하면 파일 이름에 `init`이 포함된 모든 파일이 나타난다. `inlu`처럼 연속되지 않은 글자를 입력해도 `init.lua`가 매칭된다 -- 이것이 퍼지 검색의 핵심이다.

#### live_grep: 파일 내용 검색

파일 내용에서 특정 텍스트를 검색한다. 내부적으로 ripgrep을 사용한다.

```vim
:Telescope live_grep
```

```lua
vim.keymap.set("n", "<leader>fg", "<cmd>Telescope live_grep<cr>", { desc = "텍스트 검색" })
```

`TODO`를 검색하면 프로젝트 전체에서 TODO가 포함된 모든 줄이 표시된다. 코드베이스에서 특정 함수가 어디서 사용되는지 찾을 때 매우 유용하다.

#### git_files: Git이 추적하는 파일만 검색

```vim
:Telescope git_files
```

`find_files`와 비슷하지만, `.gitignore`에 의해 제외된 파일은 나타나지 않는다. `node_modules`나 빌드 결과물 같은 불필요한 파일을 자연스럽게 걸러낸다.

### 버퍼와 최근 파일

#### buffers: 열린 버퍼 전환

현재 열려 있는 모든 버퍼를 표시한다.

```vim
:Telescope buffers
```

```lua
vim.keymap.set("n", "<leader>fb", "<cmd>Telescope buffers<cr>", { desc = "버퍼 목록" })
```

여러 파일을 오가며 작업할 때, `:bnext`나 `:bprev`로 하나씩 이동하는 것보다 훨씬 빠르다.

#### oldfiles: 최근 열었던 파일

Neovim이 기억하는 최근 파일 목록을 보여준다.

```vim
:Telescope oldfiles
```

```lua
vim.keymap.set("n", "<leader>fo", "<cmd>Telescope oldfiles<cr>", { desc = "최근 파일" })
```

### 도움말과 정보 검색

#### help_tags: 도움말 검색

Neovim 도움말 문서를 퍼지 검색한다.

```vim
:Telescope help_tags
```

```lua
vim.keymap.set("n", "<leader>fh", "<cmd>Telescope help_tags<cr>", { desc = "도움말 검색" })
```

`:help` 명령보다 훨씬 편리하다. `keymap`이라고 입력하면 키맵 관련 도움말 항목이 모두 나타난다.

#### keymaps: 키맵 검색

현재 설정된 모든 키맵을 검색한다.

```vim
:Telescope keymaps
```

"이 키가 뭐였지?" 싶을 때 바로 찾을 수 있다.

#### commands: 명령 검색

사용 가능한 모든 Ex 명령과 사용자 정의 명령을 검색한다.

```vim
:Telescope commands
```

### Git 연동

Telescope는 Git 관련 Picker도 제공한다:

```vim
:Telescope git_commits     " 커밋 히스토리 탐색
:Telescope git_branches    " 브랜치 전환
:Telescope git_status      " 변경된 파일 목록
:Telescope git_stash       " stash 목록
```

```lua
vim.keymap.set("n", "<leader>gc", "<cmd>Telescope git_commits<cr>", { desc = "Git 커밋" })
vim.keymap.set("n", "<leader>gb", "<cmd>Telescope git_branches<cr>", { desc = "Git 브랜치" })
vim.keymap.set("n", "<leader>gs", "<cmd>Telescope git_status<cr>", { desc = "Git 상태" })
```

## Telescope 내 키맵

Telescope 창이 열리면 두 가지 모드가 있다: **Insert 모드**(검색 입력)와 **Normal 모드**(목록 탐색).

### Insert 모드 (검색 입력 중)

| 키 | 동작 |
|----|------|
| `<C-n>` / `<C-j>` | 다음 항목으로 이동 |
| `<C-p>` / `<C-k>` | 이전 항목으로 이동 |
| `<CR>` | 선택한 항목 열기 |
| `<C-x>` | 수평 분할(split)로 열기 |
| `<C-v>` | 수직 분할(vsplit)로 열기 |
| `<C-t>` | 새 탭(tab)으로 열기 |
| `<C-u>` | 미리보기 위로 스크롤 |
| `<C-d>` | 미리보기 아래로 스크롤 |
| `<C-q>` | 결과를 quickfix 리스트로 전송 |
| `<Esc>` | Telescope 닫기 |

### Normal 모드 (목록 탐색 중)

Insert 모드에서 `<Esc>`를 누르면 Normal 모드로 전환된다:

| 키 | 동작 |
|----|------|
| `j` / `k` | 위/아래 이동 |
| `<CR>` | 선택한 항목 열기 |
| `H` / `M` / `L` | 목록의 맨 위/중간/맨 아래 |
| `gg` / `G` | 목록의 처음/끝 |
| `?` | 키맵 도움말 표시 |
| `q` | Telescope 닫기 |

> **팁**: Telescope 안에서도 Vim의 모달 편집 개념이 그대로 적용된다. Insert 모드에서는 검색어를 입력하고, Normal 모드에서는 결과 목록을 탐색한다.

## 설정 커스터마이징

Telescope의 기본 설정을 프로젝트에 맞게 조정할 수 있다.

### 기본 설정 예제

```lua
return {
  "nvim-telescope/telescope.nvim",
  tag = "0.1.8",
  dependencies = {
    "nvim-lua/plenary.nvim",
  },
  config = function()
    local telescope = require("telescope")
    local actions = require("telescope.actions")

    telescope.setup({
      defaults = {
        -- 검색 결과에서 제외할 패턴
        file_ignore_patterns = {
          "node_modules",
          ".git/",
          "dist/",
          "build/",
        },

        -- 레이아웃 설정
        layout_strategy = "horizontal",
        layout_config = {
          horizontal = {
            preview_cutoff = 120,
            preview_width = 0.5,
          },
          width = 0.87,
          height = 0.80,
        },

        -- 커스텀 키맵
        mappings = {
          i = {
            ["<C-j>"] = actions.move_selection_next,
            ["<C-k>"] = actions.move_selection_previous,
            ["<C-q>"] = actions.send_selected_to_qflist + actions.open_qflist,
            ["<Esc>"] = actions.close,
          },
          n = {
            ["q"] = actions.close,
          },
        },

        -- 정렬 설정
        sorting_strategy = "ascending",

        -- 프롬프트 위치
        prompt_prefix = "   ",
        selection_caret = "  ",
      },

      pickers = {
        find_files = {
          -- 숨김 파일도 표시
          hidden = true,
        },
        buffers = {
          -- 최근 사용 순으로 정렬
          sort_lastused = true,
          -- 현재 버퍼 제외
          ignore_current_buffer = true,
        },
      },
    })
  end,
}
```

### layout_strategy 옵션

| 전략 | 설명 |
|------|------|
| `horizontal` | 미리보기가 오른쪽 (기본값) |
| `vertical` | 미리보기가 위쪽 |
| `center` | 결과만 화면 중앙에 표시 |
| `flex` | 화면 크기에 따라 자동 전환 |

좁은 화면에서는 `vertical`, 넓은 화면에서는 `horizontal`이 편리하다. `flex`를 사용하면 자동으로 전환된다.

## 확장(Extensions)

Telescope의 기능을 더욱 강화하는 확장 플러그인들이 있다.

### fzf-native: 검색 성능 향상

기본 정렬 알고리즘 대신 C로 컴파일된 fzf 알고리즘을 사용한다. 대규모 프로젝트에서 체감 차이가 크다.

```lua
return {
  "nvim-telescope/telescope.nvim",
  tag = "0.1.8",
  dependencies = {
    "nvim-lua/plenary.nvim",
    {
      "nvim-telescope/telescope-fzf-native.nvim",
      build = "make",
    },
  },
  config = function()
    local telescope = require("telescope")

    telescope.setup({
      extensions = {
        fzf = {
          fuzzy = true,
          override_generic_sorter = true,
          override_file_sorter = true,
          case_mode = "smart_case",
        },
      },
    })

    -- 확장 로드
    telescope.load_extension("fzf")
  end,
}
```

fzf-native를 설치하면 검색 구문도 더 강력해진다:

| 구문 | 의미 | 예시 |
|------|------|------|
| `abc` | 퍼지 매칭 | `abc` |
| `'abc` | 정확한 매칭 | `'init` → "init"만 매칭 |
| `^abc` | 접두사 매칭 | `^src` → "src"로 시작 |
| `abc$` | 접미사 매칭 | `.lua$` → ".lua"로 끝남 |
| `!abc` | 역 매칭 (제외) | `!test` → "test" 제외 |

### ui-select: Neovim 선택 UI 대체

`vim.ui.select`를 Telescope UI로 대체한다. 코드 액션(code action) 선택 등에서 Telescope의 인터페이스를 사용할 수 있다.

```lua
{
  "nvim-telescope/telescope-ui-select.nvim",
  config = function()
    require("telescope").load_extension("ui-select")
  end,
}
```

### file_browser: 파일 탐색

Telescope 안에서 파일 시스템을 탐색하고, 파일을 생성/삭제/이동할 수 있다.

```lua
{
  "nvim-telescope/telescope-file-browser.nvim",
  dependencies = { "nvim-telescope/telescope.nvim", "nvim-lua/plenary.nvim" },
  config = function()
    require("telescope").load_extension("file_browser")
  end,
}
```

## 실전 워크플로우

### 워크플로우 1: 프로젝트에서 코드 찾기

1. `<leader>fg`로 live_grep 열기
2. 찾고 싶은 함수명이나 변수명 입력
3. 결과 목록에서 원하는 항목 선택
4. `<CR>`로 해당 파일의 해당 줄로 바로 이동

### 워크플로우 2: 빠른 파일 전환

1. `<leader>fb`로 버퍼 목록 열기
2. 파일 이름 일부 입력
3. `<CR>`로 전환

또는 `<leader>fo`로 최근 파일을 열 수도 있다.

### 워크플로우 3: 설정 파일 찾기

Neovim 설정을 수정하고 싶을 때:

```lua
vim.keymap.set("n", "<leader>fn", function()
  require("telescope.builtin").find_files({
    cwd = vim.fn.stdpath("config"),
  })
end, { desc = "Neovim 설정 파일 검색" })
```

이렇게 하면 `<leader>fn`으로 Neovim 설정 디렉터리에서만 파일을 검색할 수 있다.

### 워크플로우 4: 검색 후 일괄 작업

1. `<leader>fg`로 live_grep 열기
2. 검색어 입력
3. `<Tab>`으로 원하는 항목 다중 선택
4. `<C-q>`로 quickfix 리스트로 전송
5. quickfix 리스트에서 `:cdo s/old/new/g`로 일괄 치환

## 추천 키맵 설정

```lua
-- 파일 검색
vim.keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<cr>", { desc = "파일 찾기" })
vim.keymap.set("n", "<leader>fg", "<cmd>Telescope live_grep<cr>", { desc = "텍스트 검색" })
vim.keymap.set("n", "<leader>fb", "<cmd>Telescope buffers<cr>", { desc = "버퍼 목록" })
vim.keymap.set("n", "<leader>fo", "<cmd>Telescope oldfiles<cr>", { desc = "최근 파일" })
vim.keymap.set("n", "<leader>fh", "<cmd>Telescope help_tags<cr>", { desc = "도움말 검색" })

-- 현재 버퍼 내 검색
vim.keymap.set("n", "<leader>/", "<cmd>Telescope current_buffer_fuzzy_find<cr>", { desc = "버퍼 내 검색" })

-- Git
vim.keymap.set("n", "<leader>gc", "<cmd>Telescope git_commits<cr>", { desc = "Git 커밋" })
vim.keymap.set("n", "<leader>gb", "<cmd>Telescope git_branches<cr>", { desc = "Git 브랜치" })
vim.keymap.set("n", "<leader>gs", "<cmd>Telescope git_status<cr>", { desc = "Git 상태" })

-- LSP 연동 (LSP가 설정된 경우)
vim.keymap.set("n", "gd", "<cmd>Telescope lsp_definitions<cr>", { desc = "정의로 이동" })
vim.keymap.set("n", "gr", "<cmd>Telescope lsp_references<cr>", { desc = "참조 찾기" })
vim.keymap.set("n", "<leader>ds", "<cmd>Telescope lsp_document_symbols<cr>", { desc = "문서 심볼" })

-- 커서 아래 단어로 검색
vim.keymap.set("n", "<leader>fw", "<cmd>Telescope grep_string<cr>", { desc = "커서 단어 검색" })

-- Neovim 설정 파일 검색
vim.keymap.set("n", "<leader>fn", function()
  require("telescope.builtin").find_files({ cwd = vim.fn.stdpath("config") })
end, { desc = "설정 파일 검색" })
```

## 실습: Telescope 활용

1. Telescope를 설치한 후 `:Telescope`를 입력하고 `<Tab>`을 눌러 사용 가능한 Picker 목록을 확인하자
2. `<leader>ff`로 프로젝트에서 `init`이 포함된 파일을 찾아보자
3. `<leader>fg`로 `require`라는 텍스트가 포함된 파일을 검색해 보자
4. `<leader>fb`로 열린 버퍼 목록을 확인하고, 퍼지 검색으로 전환해 보자
5. `<leader>fh`로 `telescope`를 검색하여 도움말 문서를 읽어보자
6. live_grep에서 결과를 다중 선택(`<Tab>`)한 뒤 `<C-q>`로 quickfix 리스트에 보내보자
7. 결과 항목을 `<C-x>`(수평 분할)와 `<C-v>`(수직 분할)로 열어보자

## 핵심 정리

- Telescope는 Neovim의 **통합 퍼지 파인더**로, 파일, 텍스트, 버퍼, Git, LSP 등 모든 것을 검색한다
- 핵심 Picker: **`find_files`**(파일 이름), **`live_grep`**(파일 내용), **`buffers`**(버퍼 전환)
- Telescope 내에서 `<C-n>`/`<C-p>`로 이동, `<CR>`로 선택, `<C-x>`/`<C-v>`/`<C-t>`로 분할/탭 열기
- **fzf-native** 확장을 설치하면 검색 성능이 크게 향상된다
- `<C-q>`로 검색 결과를 **quickfix 리스트**로 보내면 일괄 작업이 가능하다
- Telescope는 **LSP와 연동**하여 정의 이동, 참조 찾기 등에도 사용할 수 있다
- 자주 쓰는 Picker에 키맵을 설정하면 **프로젝트 내비게이션이 극적으로 빨라진다**
