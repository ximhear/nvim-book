# Chapter 18: Telescope - 퍼지 파인더 (LazyVim)

프로젝트 안에서 원하는 파일을 찾거나, 특정 문자열이 포함된 코드를 검색하거나, 열린 버퍼를 전환하는 일은 개발 중 끊임없이 반복된다. Telescope는 이 모든 "찾기" 작업을 하나의 인터페이스로 통합하는 Neovim용 **퍼지 파인더(fuzzy finder)**다. LazyVim에는 Telescope가 **기본으로 포함**되어 있고, 키맵과 확장까지 설정되어 있다.

"퍼지"란 정확히 일치하지 않아도 된다는 뜻이다. `init`이라고 입력하면 `init.lua`, `initialize.ts`, `config_init.py` 등이 모두 매칭된다. 타이핑 몇 글자만으로 수천 개의 파일 중 원하는 것을 찾아낼 수 있다.

## LazyVim의 Telescope 구성

LazyVim에는 다음이 이미 설정되어 있다:

| 구성 요소 | 상태 |
|-----------|------|
| `telescope.nvim` | 기본 포함 |
| `telescope-fzf-native.nvim` | 기본 포함 (성능 향상) |
| 핵심 키맵 | 기본 설정됨 |
| LSP 연동 | 기본 설정됨 |

별도의 설치나 키맵 설정 없이 바로 사용할 수 있다.

### 외부 도구 설치

Telescope의 검색 성능을 위해 외부 도구 설치가 필요하다:

```bash
# macOS
brew install ripgrep fd

# Ubuntu/Debian
sudo apt install ripgrep fd-find

# Arch Linux
sudo pacman -S ripgrep fd
```

| 도구 | 역할 | 필수 여부 |
|------|------|-----------|
| `ripgrep` (rg) | 파일 내용 검색 | 강력 권장 |
| `fd` | 파일 이름 검색 | 선택 (없으면 find 사용) |

## 기본 제공 키맵

LazyVim은 Telescope 키맵을 체계적으로 구성해 놓았다. **별도 키맵 설정이 필요 없다.**

### 파일 검색

| 키 | Picker | 설명 |
|----|--------|------|
| `<leader><space>` | find_files | 프로젝트 파일 찾기 |
| `<leader>ff` | find_files | 프로젝트 파일 찾기 |
| `<leader>fF` | find_files (cwd) | 현재 디렉터리 기준 파일 찾기 |
| `<leader>fr` | oldfiles | 최근 파일 |
| `<leader>fR` | oldfiles (cwd) | 최근 파일 (현재 디렉터리) |
| `<leader>fg` | git_files | Git 추적 파일 검색 |

### 텍스트 검색

| 키 | Picker | 설명 |
|----|--------|------|
| `<leader>/` | live_grep | 프로젝트 전체 텍스트 검색 |
| `<leader>sg` | live_grep | 프로젝트 전체 텍스트 검색 |
| `<leader>sG` | live_grep (cwd) | 현재 디렉터리 텍스트 검색 |
| `<leader>sw` | grep_string | 커서 아래 단어 검색 |
| `<leader>sW` | grep_string (cwd) | 커서 아래 단어 검색 (현재 디렉터리) |

### 버퍼와 탐색

| 키 | Picker | 설명 |
|----|--------|------|
| `<leader>,` | buffers | 열린 버퍼 전환 |
| `<leader>fb` | buffers | 열린 버퍼 전환 |
| `<leader>:` | command_history | 명령 히스토리 |

### 검색 (Search)

| 키 | Picker | 설명 |
|----|--------|------|
| `<leader>sa` | autocommands | Autocommands 검색 |
| `<leader>sc` | command_history | 명령 히스토리 |
| `<leader>sC` | commands | 사용 가능한 명령 검색 |
| `<leader>sd` | diagnostics (buffer) | 현재 버퍼 진단 검색 |
| `<leader>sD` | diagnostics | 전체 진단 검색 |
| `<leader>sh` | help_tags | 도움말 검색 |
| `<leader>sk` | keymaps | 키맵 검색 |
| `<leader>sm` | marks | 마크 검색 |
| `<leader>sR` | resume | 마지막 검색 다시 열기 |
| `<leader>ss` | lsp_document_symbols | 문서 심볼 검색 |
| `<leader>sS` | lsp_workspace_symbols | 워크스페이스 심볼 검색 |

### Git

| 키 | Picker | 설명 |
|----|--------|------|
| `<leader>gc` | git_commits | 커밋 히스토리 |
| `<leader>gs` | git_status | 변경된 파일 목록 |

> **팁**: `<leader>sR`(resume)은 매우 유용하다. 마지막으로 사용한 Telescope 검색을 결과 그대로 다시 열어준다.

## 핵심 Picker

### find_files: 파일 이름으로 검색 (`<leader><space>` 또는 `<leader>ff`)

프로젝트에서 파일을 찾을 때 가장 많이 사용하는 Picker다.

검색창에 `init`을 입력하면 파일 이름에 `init`이 포함된 모든 파일이 나타난다. `inlu`처럼 연속되지 않은 글자를 입력해도 `init.lua`가 매칭된다 -- 이것이 퍼지 검색의 핵심이다.

### live_grep: 파일 내용 검색 (`<leader>/` 또는 `<leader>sg`)

파일 내용에서 특정 텍스트를 검색한다. 내부적으로 ripgrep을 사용한다.

`TODO`를 검색하면 프로젝트 전체에서 TODO가 포함된 모든 줄이 표시된다. 코드베이스에서 특정 함수가 어디서 사용되는지 찾을 때 매우 유용하다.

### buffers: 열린 버퍼 전환 (`<leader>,` 또는 `<leader>fb`)

현재 열려 있는 모든 버퍼를 표시한다. 여러 파일을 오가며 작업할 때, `:bnext`나 `:bprev`로 하나씩 이동하는 것보다 훨씬 빠르다.

### help_tags: 도움말 검색 (`<leader>sh`)

Neovim 도움말 문서를 퍼지 검색한다. `:help` 명령보다 훨씬 편리하다. `keymap`이라고 입력하면 키맵 관련 도움말 항목이 모두 나타난다.

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

### fzf 검색 구문

LazyVim에는 fzf-native가 기본 포함되어 있어 강력한 검색 구문을 사용할 수 있다:

| 구문 | 의미 | 예시 |
|------|------|------|
| `abc` | 퍼지 매칭 | `abc` |
| `'abc` | 정확한 매칭 | `'init` -> "init"만 매칭 |
| `^abc` | 접두사 매칭 | `^src` -> "src"로 시작 |
| `abc$` | 접미사 매칭 | `.lua$` -> ".lua"로 끝남 |
| `!abc` | 역 매칭 (제외) | `!test` -> "test" 제외 |

## 설정 커스터마이징

기본 설정을 변경하고 싶다면 `opts`를 오버라이드한다:

```lua
-- lua/plugins/telescope.lua
return {
  {
    "nvim-telescope/telescope.nvim",
    opts = {
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

        -- 정렬 설정
        sorting_strategy = "ascending",
      },

      pickers = {
        find_files = {
          hidden = true,  -- 숨김 파일도 표시
        },
        buffers = {
          sort_lastused = true,
          ignore_current_buffer = true,
        },
      },
    },
  },
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

## 실전 워크플로우

### 워크플로우 1: 프로젝트에서 코드 찾기

1. `<leader>/`로 live_grep 열기
2. 찾고 싶은 함수명이나 변수명 입력
3. 결과 목록에서 원하는 항목 선택
4. `<CR>`로 해당 파일의 해당 줄로 바로 이동

### 워크플로우 2: 빠른 파일 전환

1. `<leader>,`로 버퍼 목록 열기
2. 파일 이름 일부 입력
3. `<CR>`로 전환

또는 `<leader>fr`로 최근 파일을 열 수도 있다.

### 워크플로우 3: 설정 파일 찾기

LazyVim에서는 Neovim 설정 파일을 찾는 키맵이 기본 제공된다:

```
<leader>fc   -- Neovim 설정 파일 검색
```

### 워크플로우 4: 검색 후 일괄 작업

1. `<leader>/`로 live_grep 열기
2. 검색어 입력
3. `<Tab>`으로 원하는 항목 다중 선택
4. `<C-q>`로 quickfix 리스트로 전송
5. quickfix 리스트에서 `:cdo s/old/new/g`로 일괄 치환

### 워크플로우 5: 마지막 검색 이어가기

검색 결과를 닫았다가 다시 보고 싶을 때:

```
<leader>sR   -- 마지막 Telescope 검색을 결과 그대로 다시 열기
```

## 실습: Telescope 활용

1. `<leader><space>`로 프로젝트에서 `init`이 포함된 파일을 찾아보자
2. `<leader>/`로 `require`라는 텍스트가 포함된 파일을 검색해 보자
3. `<leader>,`로 열린 버퍼 목록을 확인하고, 퍼지 검색으로 전환해 보자
4. `<leader>sh`로 `telescope`를 검색하여 도움말 문서를 읽어보자
5. `<leader>sk`로 현재 설정된 키맵을 검색해 보자
6. live_grep에서 결과를 다중 선택(`<Tab>`)한 뒤 `<C-q>`로 quickfix 리스트에 보내보자
7. 결과 항목을 `<C-x>`(수평 분할)와 `<C-v>`(수직 분할)로 열어보자
8. `<leader>sR`로 마지막 검색을 다시 열어보자

## 핵심 정리

- LazyVim에는 Telescope가 **기본 포함**되어 있고, fzf-native도 함께 설정되어 있다
- **핵심 키맵**: `<leader><space>`(파일 찾기), `<leader>/`(텍스트 검색), `<leader>,`(버퍼 전환)
- **검색 키맵**: `<leader>s` 접두사 아래에 체계적으로 정리 (`sh`=도움말, `sk`=키맵, `sg`=grep 등)
- Telescope 내에서 `<C-n>`/`<C-p>`로 이동, `<CR>`로 선택, `<C-x>`/`<C-v>`/`<C-t>`로 분할/탭 열기
- `<C-q>`로 검색 결과를 **quickfix 리스트**로 보내면 일괄 작업이 가능하다
- `<leader>sR`로 **마지막 검색을 다시 열 수 있다**
- 커스터마이징은 `opts` 오버라이드로 간단히 할 수 있다
