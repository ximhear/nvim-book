# Chapter 24: Python 개발 환경 (LazyVim)

Python은 웹 개발, 데이터 과학, 자동화, AI/ML 등 다양한 분야에서 사용되는 언어다. LazyVim은 Python 개발에 필요한 LSP, 포매팅, 린팅, 디버깅, 테스트를 **extras 하나로** 통합 설정할 수 있다.

## Python extras 활성화

LazyVim의 `lang.python` extras를 활성화하면 다음이 한 번에 설정된다:

| 구성 요소 | 도구 | 역할 |
|-----------|------|------|
| LSP | basedpyright | 타입 검사, 자동 완성, 정의 이동 |
| 린팅 | ruff | 빠른 Python 린터 (Rust 기반) |
| 포매팅 | ruff | 코드 포매팅 (Black 호환) |
| Treesitter | python 파서 | 구문 하이라이팅, 텍스트 오브젝트 |
| DAP | debugpy | 디버깅 (dap extras 활성화 시) |
| 테스트 | neotest-python | 테스트 실행 (test extras 활성화 시) |

### 활성화 방법

```vim
:LazyExtras
```

에서 `lang.python`을 선택하거나, `lua/config/lazy.lua`에서:

```lua
require("lazy").setup({
  spec = {
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },

    { import = "lazyvim.plugins.extras.lang.python" },

    { import = "plugins" },
  },
})
```

Neovim을 재시작하면 mason이 필요한 도구를 자동으로 설치한다.

## LSP: basedpyright

### basedpyright란

basedpyright는 Microsoft의 pyright를 포크한 Python LSP 서버다. pyright의 모든 기능을 포함하면서 추가 기능과 버그 수정이 더해졌다. LazyVim의 `lang.python` extras는 기본으로 basedpyright를 사용한다.

> **참고**: 기존 pyright를 사용하고 싶다면 서버 설정에서 변경할 수 있다.

### 제공하는 기능

| 기능 | 설명 |
|------|------|
| 타입 검사 | 타입 힌트 기반 정적 분석 |
| 자동 완성 | 함수, 메서드, 속성 제안 |
| 정의 이동 | `gd`로 함수/클래스 정의로 점프 |
| 참조 찾기 | `gr`로 심볼 사용 위치 검색 |
| 호버 문서 | `K`로 타입 정보, docstring 표시 |
| 자동 import | 코드 액션으로 import 자동 추가 |
| 이름 변경 | `<leader>cr`로 심볼 일괄 변경 |

### 타입 검사 수준 설정

```lua
-- lua/plugins/python.lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        basedpyright = {
          settings = {
            basedpyright = {
              analysis = {
                typeCheckingMode = "standard",  -- "off", "basic", "standard", "strict", "all"
                autoSearchPaths = true,
                useLibraryCodeForTypes = true,
                diagnosticSeverityOverrides = {
                  -- 특정 진단 심각도 조정
                  reportMissingTypeStubs = "none",
                },
              },
            },
          },
        },
      },
    },
  },
}
```

| 모드 | 설명 | 권장 대상 |
|------|------|----------|
| `off` | 타입 검사 비활성화 | 레거시 프로젝트 |
| `basic` | 기본적인 오류만 검출 | 타입 힌트를 거의 쓰지 않는 프로젝트 |
| `standard` | 균형 잡힌 검사 (기본값) | 대부분의 프로젝트 |
| `strict` | 엄격한 타입 검사 | 타입 힌트를 적극 활용하는 프로젝트 |
| `all` | 최대한 엄격 | 라이브러리 개발 |

### pyright로 변경

basedpyright 대신 pyright를 사용하고 싶다면:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        basedpyright = { enabled = false },
        pyright = {
          settings = {
            python = {
              analysis = {
                typeCheckingMode = "basic",
              },
            },
          },
        },
      },
    },
  },
}
```

## 린팅과 포매팅: ruff

### ruff란

ruff는 Rust로 작성된 **초고속 Python 린터이자 포매터**다. flake8, isort, Black 등 여러 도구의 기능을 하나로 통합하면서 10~100배 빠른 성능을 제공한다. LazyVim의 `lang.python` extras에 기본 포함되어 있다.

### ruff가 대체하는 도구들

| 기존 도구 | ruff의 대응 기능 |
|-----------|-----------------|
| flake8 | 린트 규칙 |
| isort | import 정렬 |
| Black | 코드 포매팅 |
| pyflakes | 미사용 변수/import 감지 |
| pycodestyle | PEP 8 스타일 검사 |

### 기본 동작

extras를 활성화하면:

- **저장 시 자동 포매팅** -- ruff format이 적용된다
- **실시간 린팅** -- 편집 중 오류와 경고가 표시된다
- **import 정렬** -- 저장 시 import가 자동 정렬된다

### ruff 설정 커스터마이징

프로젝트 루트에 `ruff.toml` 또는 `pyproject.toml`로 ruff를 설정한다:

```toml
# ruff.toml
line-length = 88
target-version = "py311"

[lint]
select = [
    "E",    # pycodestyle errors
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
]
ignore = [
    "E501",  # line too long (포매터가 처리)
]

[lint.isort]
known-first-party = ["mypackage"]
```

또는 `pyproject.toml`에서:

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]
```

### Black을 포매터로 사용

ruff 대신 Black을 선호한다면:

```lua
-- lua/plugins/python.lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        python = { "black", "isort" },
      },
    },
  },
}
```

mason에서 설치해야 한다:

```vim
:MasonInstall black isort
```

## 가상 환경 (Virtual Environment)

Python 개발에서 가상 환경은 필수다. LazyVim/basedpyright가 올바른 가상 환경을 인식해야 정확한 자동 완성과 타입 검사가 동작한다.

### 자동 인식

basedpyright는 다음 경로에서 가상 환경을 자동으로 감지한다:

- `.venv/` (프로젝트 루트)
- `venv/` (프로젝트 루트)
- `$VIRTUAL_ENV` 환경 변수

```bash
# 프로젝트에서 가상 환경 생성 (권장 방식)
python -m venv .venv
source .venv/bin/activate

# 패키지 설치
pip install django requests numpy
```

`.venv/` 디렉터리가 프로젝트 루트에 있으면 basedpyright가 자동으로 인식한다.

### 수동 지정

자동 인식이 안 될 때 수동으로 Python 경로를 지정한다:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        basedpyright = {
          settings = {
            basedpyright = {
              analysis = {
                -- 가상 환경 경로 지정
                venvPath = ".",
                venv = ".venv",
              },
            },
          },
        },
      },
    },
  },
}
```

### venv-selector.nvim (선택)

여러 가상 환경을 전환해야 하는 경우 venv-selector.nvim을 추가할 수 있다:

```lua
-- lua/plugins/venv-selector.lua
return {
  {
    "linux-cultist/venv-selector.nvim",
    branch = "regexp",
    dependencies = {
      "neovim/nvim-lspconfig",
      "nvim-telescope/telescope.nvim",
    },
    opts = {},
    keys = {
      { "<leader>cv", "<cmd>VenvSelect<CR>", desc = "가상 환경 선택" },
    },
  },
}
```

`<leader>cv`를 누르면 Telescope에서 사용 가능한 가상 환경 목록이 표시되고, 선택하면 LSP 서버가 자동으로 재시작된다.

### conda 환경

conda를 사용하는 경우:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        basedpyright = {
          before_init = function(_, config)
            local conda_prefix = os.getenv("CONDA_PREFIX")
            if conda_prefix then
              config.settings.python = config.settings.python or {}
              config.settings.python.pythonPath = conda_prefix .. "/bin/python"
            end
          end,
        },
      },
    },
  },
}
```

## 디버깅: debugpy

Python 디버깅을 사용하려면 `dap.core` extras도 함께 활성화해야 한다.

### extras 활성화

```lua
-- lua/config/lazy.lua
{ import = "lazyvim.plugins.extras.lang.python" },
{ import = "lazyvim.plugins.extras.dap.core" },
```

### debugpy 설치

mason이 자동으로 설치하지만, 수동으로도 가능하다:

```vim
:MasonInstall debugpy
```

또는 가상 환경에 직접 설치:

```bash
pip install debugpy
```

### 기본 키맵

DAP 관련 키맵은 ch21에서 다룬 것과 동일하다:

| 키 | 기능 |
|----|------|
| `<leader>db` | 브레이크포인트 토글 |
| `<leader>dB` | 조건부 브레이크포인트 |
| `<leader>dc` | 디버그 시작/계속 |
| `<leader>dO` | 스텝 오버 |
| `<leader>di` | 스텝 인투 |
| `<leader>do` | 스텝 아웃 |
| `<leader>dt` | 디버그 종료 |
| `<leader>dr` | REPL 토글 |

### 디버깅 실전

```python
# calculator.py
def add(a, b):
    result = a + b      # <- 여기에 브레이크포인트
    return result

def main():
    x = add(10, 20)
    y = add(x, 30)
    print(f"Result: {y}")

if __name__ == "__main__":
    main()
```

1. `result = a + b` 줄에서 `<leader>db`로 브레이크포인트 설정
2. `<leader>dc`로 디버그 시작 -- "Launch file" 선택
3. 브레이크포인트에서 멈추면 변수 값을 확인 (dap-ui에 표시)
4. `<leader>dO`로 한 줄씩 실행
5. `<leader>dc`로 계속 실행
6. `<leader>dt`로 종료

### 디버그 설정 커스터마이징

Django, Flask 등 특정 프레임워크에 맞는 디버그 설정을 추가할 수 있다:

```lua
-- lua/plugins/python-dap.lua
return {
  {
    "mfussenegger/nvim-dap",
    optional = true,
    opts = function()
      local dap = require("dap")

      -- Django 디버그 설정 추가
      table.insert(dap.configurations.python, {
        type = "python",
        request = "launch",
        name = "Django",
        program = "${workspaceFolder}/manage.py",
        args = { "runserver", "--noreload" },
        django = true,
        justMyCode = true,
      })

      -- Flask 디버그 설정 추가
      table.insert(dap.configurations.python, {
        type = "python",
        request = "launch",
        name = "Flask",
        module = "flask",
        args = { "run", "--no-debugger", "--no-reload" },
        env = { FLASK_APP = "app.py" },
        justMyCode = true,
      })

      -- pytest 디버그 설정 추가
      table.insert(dap.configurations.python, {
        type = "python",
        request = "launch",
        name = "pytest (current file)",
        module = "pytest",
        args = { "${file}", "-v" },
        justMyCode = true,
      })
    end,
  },
}
```

## 테스트: neotest

LazyVim은 neotest를 extras로 제공한다. `lang.python` extras와 함께 `test.core` extras를 활성화하면 Python 테스트를 Neovim 안에서 실행할 수 있다.

### extras 활성화

```lua
-- lua/config/lazy.lua
{ import = "lazyvim.plugins.extras.lang.python" },
{ import = "lazyvim.plugins.extras.test.core" },
```

### 기본 키맵

| 키 | 기능 |
|----|------|
| `<leader>tt` | 가장 가까운 테스트 실행 |
| `<leader>tT` | 현재 파일의 모든 테스트 실행 |
| `<leader>tr` | 마지막 테스트 다시 실행 |
| `<leader>ts` | 테스트 요약 토글 |
| `<leader>to` | 테스트 출력 표시 |
| `<leader>tS` | 테스트 중지 |

### pytest 사용

neotest-python은 기본으로 pytest를 사용한다. 프로젝트에 pytest가 설치되어 있어야 한다:

```bash
pip install pytest
```

테스트 파일 예시:

```python
# tests/test_calculator.py
from calculator import add, subtract

def test_add():
    assert add(1, 2) == 3

def test_add_negative():
    assert add(-1, -2) == -3

def test_subtract():
    assert subtract(5, 3) == 2

class TestCalculator:
    def test_add_zero(self):
        assert add(0, 0) == 0

    def test_subtract_same(self):
        assert subtract(5, 5) == 0
```

테스트 파일을 열고:

1. `<leader>tt`로 커서 위치의 테스트 하나를 실행
2. `<leader>tT`로 파일의 모든 테스트를 실행
3. `<leader>ts`로 테스트 요약 패널 표시
4. `<leader>to`로 실패한 테스트의 출력 확인

### pytest 설정

```lua
-- lua/plugins/neotest.lua
return {
  {
    "nvim-neotest/neotest",
    optional = true,
    opts = {
      adapters = {
        ["neotest-python"] = {
          runner = "pytest",
          python = ".venv/bin/python",
          args = { "-v", "--tb=short" },
        },
      },
    },
  },
}
```

## Python 전용 키맵 요약

LazyVim extras 활성화 후 Python 파일에서 사용할 수 있는 주요 키맵:

### LSP (기본)

| 키 | 기능 |
|----|------|
| `gd` | 정의로 이동 |
| `gr` | 참조 찾기 |
| `K` | 호버 문서 (docstring 표시) |
| `<leader>cr` | 이름 변경 |
| `<leader>ca` | 코드 액션 (import 추가 등) |
| `<leader>cf` | 포매팅 |
| `]d` / `[d` | 다음/이전 진단 |

### 디버깅 (dap.core extras)

| 키 | 기능 |
|----|------|
| `<leader>db` | 브레이크포인트 |
| `<leader>dc` | 디버그 시작/계속 |
| `<leader>dO` | 스텝 오버 |
| `<leader>di` | 스텝 인투 |
| `<leader>do` | 스텝 아웃 |
| `<leader>dt` | 종료 |

### 테스트 (test.core extras)

| 키 | 기능 |
|----|------|
| `<leader>tt` | 테스트 실행 (커서 위치) |
| `<leader>tT` | 파일 전체 테스트 |
| `<leader>tr` | 마지막 테스트 반복 |
| `<leader>ts` | 테스트 요약 |

## 프로젝트 구조 예시

전형적인 Python 프로젝트에서의 설정 파일:

```
my-project/
├── .venv/                    # 가상 환경
├── pyproject.toml            # 프로젝트 설정 + ruff 설정
├── ruff.toml                 # (선택) ruff 별도 설정
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
└── .nvim.lua                 # (선택) 프로젝트별 Neovim 설정
```

### pyproject.toml 예시

```toml
[project]
name = "mypackage"
version = "0.1.0"
requires-python = ">=3.11"

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
```

### .nvim.lua (프로젝트별 설정)

```lua
-- 프로젝트 전용 실행 명령
vim.keymap.set("n", "<leader>mr", function()
  vim.cmd("split | terminal python -m mypackage.main")
end, { desc = "프로젝트 실행" })

vim.keymap.set("n", "<leader>mt", function()
  vim.cmd("split | terminal pytest -v")
end, { desc = "전체 테스트" })
```

## 실전 워크플로우

### 새 Python 프로젝트 시작

```bash
# 1. 프로젝트 디렉터리 생성
mkdir my-project && cd my-project

# 2. 가상 환경 생성 및 활성화
python -m venv .venv
source .venv/bin/activate

# 3. 기본 패키지 설치
pip install pytest

# 4. Neovim 실행
nvim .
```

Neovim을 열면 basedpyright가 `.venv`를 자동 인식하고, ruff가 린팅/포매팅을 처리한다.

### 일상적인 개발 흐름

```
1. <leader><space>  -> 파일 찾기
2. 코드 작성
   - 자동 완성으로 함수/메서드 입력
   - K로 docstring 확인
   - <leader>ca로 import 자동 추가
3. 저장 -> ruff가 자동 포매팅 + import 정렬
4. ]d / [d -> 진단 확인 및 수정
5. <leader>tt -> 테스트 실행
6. <leader>db + <leader>dc -> 필요 시 디버깅
7. <leader>gg -> lazygit으로 커밋
```

### 리팩토링 흐름

```
1. <leader>cr  -> 변수/함수 이름 변경 (프로젝트 전체)
2. <leader>ca  -> 코드 액션 (사용하지 않는 import 제거 등)
3. <leader>/   -> 프로젝트 전체에서 텍스트 검색
4. gr          -> 심볼 참조 찾기
5. <leader>tT  -> 리팩토링 후 테스트 확인
```

## 트러블슈팅

### basedpyright가 패키지를 인식하지 못할 때

```vim
" 1. LSP 상태 확인
:LspInfo

" 2. Python 경로 확인 -- basedpyright가 어떤 Python을 사용하는지
" K를 눌러 호버에서 확인하거나:
:lua print(vim.lsp.get_clients()[1].config.settings)
```

**해결**: 가상 환경이 활성화된 상태에서 Neovim을 실행하거나, venv-selector로 가상 환경을 선택한다.

### ruff가 동작하지 않을 때

```vim
:ConformInfo     " 포매터 상태 확인
:LspInfo         " ruff LSP 확인
```

**해결**: `:MasonInstall ruff`로 재설치하거나, `ruff.toml` 문법 오류를 확인한다.

### 디버깅이 시작되지 않을 때

```vim
:lua require("dap").configurations.python   " DAP 설정 확인
```

**해결**: debugpy가 설치되어 있는지 확인 (`:MasonInstall debugpy`), 가상 환경의 Python 경로가 올바른지 확인한다.

## 실습: Python 개발 환경 체험

**연습 1: 기본 설정**

1. `lang.python` extras를 활성화한다
2. Python 프로젝트를 열고 `.py` 파일을 만든다
3. `import os`를 입력하고 `os.path.` 뒤에서 자동 완성이 동작하는지 확인한다
4. 의도적으로 타입 오류를 만들어 진단이 표시되는지 확인한다

**연습 2: 포매팅과 린팅**

1. 일부러 들여쓰기가 엉망인 코드를 작성한다
2. 저장하여 ruff가 자동 포매팅하는지 확인한다
3. 사용하지 않는 import를 추가하고 `<leader>ca`로 코드 액션을 확인한다

**연습 3: 디버깅**

1. `dap.core` extras를 활성화한다
2. 간단한 Python 스크립트를 작성한다
3. `<leader>db`로 브레이크포인트를 설정한다
4. `<leader>dc`로 디버깅을 시작하고 변수를 확인한다

**연습 4: 테스트**

1. `test.core` extras를 활성화한다
2. `tests/test_example.py` 파일을 만들고 테스트를 작성한다
3. `<leader>tt`로 개별 테스트를 실행한다
4. `<leader>ts`로 테스트 요약 패널을 확인한다

## 핵심 정리

- **`lang.python` extras** 하나로 LSP + 린팅 + 포매팅 + Treesitter가 설정된다
- **basedpyright**: 기본 LSP 서버, `typeCheckingMode`로 검사 수준 조절
- **ruff**: 초고속 린터/포매터, `ruff.toml` 또는 `pyproject.toml`로 프로젝트별 설정
- **가상 환경**: `.venv/` 디렉터리를 자동 인식, venv-selector로 전환 가능
- **디버깅**: `dap.core` extras + debugpy, Django/Flask/pytest 설정 추가 가능
- **테스트**: `test.core` extras + neotest-python, `<leader>tt`로 실행
- **키맵**: LSP(`gd`, `gr`, `K`, `<leader>cr`, `<leader>ca`), DAP(`<leader>d*`), Test(`<leader>t*`)
- **프로젝트 설정**: `pyproject.toml`에 ruff + pytest 설정을 통합 관리
- 가상 환경을 활성화한 상태에서 Neovim을 실행하면 가장 확실하게 동작한다
