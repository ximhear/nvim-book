# Chapter 7: 검색과 치환

텍스트 편집에서 "찾기"와 "바꾸기"는 가장 기본적이면서도 강력한 기능이다. Vim의 검색과 치환은 정규표현식을 기반으로 하며, `:s`(substitute)와 `:g`(global) 명령을 결합하면 다른 에디터에서는 상상하기 어려운 수준의 텍스트 처리가 가능하다.

## 검색 기본

### 기본 검색 명령

| 키 | 동작 |
|----|------|
| `/pattern` | 앞으로(forward) 검색 |
| `?pattern` | 뒤로(backward) 검색 |
| `n` | 같은 방향으로 다음 결과 |
| `N` | 반대 방향으로 다음 결과 |
| `*` | 커서 아래 단어를 앞으로 검색 |
| `#` | 커서 아래 단어를 뒤로 검색 |

```
/hello          → "hello"를 앞으로 검색
?hello          → "hello"를 뒤로 검색
```

`/`로 검색하면 커서 위치부터 파일 끝까지 찾고, 파일 끝에 도달하면 파일 처음으로 돌아간다 (wrapscan 옵션).

### * 과 # 검색

`*`과 `#`은 현재 커서 아래의 단어를 자동으로 검색한다. 변수명이나 함수명을 추적할 때 매우 유용하다.

```javascript
const result = calculate(data);
      ↑ 커서가 여기에 있을 때 * 를 누르면
```

`*` → `/\<result\>`와 동일한 검색이 실행된다. `\<`와 `\>`는 단어 경계(word boundary)를 의미하므로, `result`만 찾고 `results`는 찾지 않는다.

> **팁**: `g*`와 `g#`은 단어 경계 없이 검색한다. `g*`로 `result`를 검색하면 `results`, `resultSet` 등도 매칭된다.

## 검색 관련 옵션

검색의 동작을 제어하는 주요 옵션들이다. `init.lua`에서 설정한다.

```lua
vim.opt.hlsearch = true       -- 검색 결과 하이라이트
vim.opt.incsearch = true      -- 입력하면서 실시간 검색 (incremental search)
vim.opt.ignorecase = true     -- 대소문자 무시
vim.opt.smartcase = true      -- 대문자가 포함되면 대소문자 구분
```

### hlsearch와 하이라이트 끄기

`hlsearch`가 켜져 있으면 검색 결과가 하이라이트되어 가독성이 떨어질 수 있다. 하이라이트를 임시로 끄려면:

```vim
:nohlsearch         " 하이라이트 끄기 (축약: :noh)
```

많이 사용하므로 키맵을 설정하는 것이 좋다:

```lua
vim.keymap.set("n", "<Esc>", "<cmd>nohlsearch<CR>")
```

### smartcase의 동작

`ignorecase`와 `smartcase`를 함께 켜면:

| 검색 패턴 | 동작 |
|-----------|------|
| `/hello` | 대소문자 무시 → `hello`, `Hello`, `HELLO` 모두 매칭 |
| `/Hello` | 대문자 포함 → `Hello`만 매칭 |
| `/HELLO` | 대문자 포함 → `HELLO`만 매칭 |

패턴에 대문자가 하나라도 있으면 자동으로 대소문자를 구분한다. 매우 직관적이고 편리한 설정이다.

### 검색 중 대소문자 강제 지정

옵션과 무관하게 검색별로 대소문자 처리를 지정할 수 있다:

```
/hello\c        → 대소문자 무시 (ignorecase 강제)
/hello\C        → 대소문자 구분 (case sensitive 강제)
```

## 정규표현식 기초

Vim의 검색은 정규표현식을 지원한다. 다만 Vim의 기본 정규표현식 문법은 다른 언어와 약간 다르다.

### Vim 기본 정규표현식

| 패턴 | 의미 | 참고 |
|------|------|------|
| `.` | 임의의 한 문자 | |
| `*` | 앞 문자가 0번 이상 반복 | |
| `\+` | 앞 문자가 1번 이상 반복 | `+`를 이스케이프해야 함 |
| `\?` | 앞 문자가 0번 또는 1번 | `?`를 이스케이프해야 함 |
| `\|` | OR | `|`를 이스케이프해야 함 |
| `\(` `\)` | 그룹 | `()`를 이스케이프해야 함 |
| `[]` | 문자 클래스 | 이스케이프 불필요 |
| `^` | 줄 시작 | |
| `$` | 줄 끝 | |
| `\<` `\>` | 단어 경계 | |

Vim의 기본 모드에서는 `+`, `?`, `|`, `()` 등에 `\`를 붙여야 한다. 이것이 번거롭다면 **very magic** 모드를 사용한다.

### very magic 모드 (\v)

`\v`를 패턴 앞에 붙이면, 거의 모든 특수 문자가 이스케이프 없이 동작한다. Perl이나 Python의 정규표현식과 비슷해진다.

```
" 기본 모드 (magic)
/\(hello\|world\)\+

" very magic 모드
/\v(hello|world)+
```

비교:

| 기본 모드 | very magic 모드 | 의미 |
|-----------|----------------|------|
| `\(abc\)` | `\v(abc)` | 그룹 |
| `a\+` | `\va+` | 1번 이상 반복 |
| `a\?` | `\va?` | 0 또는 1번 |
| `a\|b` | `\va\|b` | OR |
| `\d` | `\v\d` | 숫자 |

> **팁**: 정규표현식을 자주 사용한다면, 항상 `\v`로 시작하는 습관을 들이면 이스케이프 지옥에서 벗어날 수 있다.

### 자주 쓰는 정규표현식 패턴

```
/\v\d+              → 숫자 (1자리 이상)
/\v[A-Z][a-z]+      → 대문자로 시작하는 단어
/\v^\s*$            → 빈 줄 (공백만 있는 줄 포함)
/\v<\w+>            → 단어 경계로 감싼 단어
/\v"[^"]*"          → 큰따옴표로 감싼 문자열
/\v\s+$             → 줄 끝의 불필요한 공백 (trailing whitespace)
```

## :s 명령 (Substitute) 심화

`:s`는 Vim에서 텍스트를 치환하는 핵심 명령이다.

### 기본 문법

```vim
:[range]s/pattern/replacement/[flags]
```

### 범위 (Range)

| 범위 | 의미 |
|------|------|
| (없음) | 현재 줄만 |
| `%` | 파일 전체 |
| `1,10` | 1~10번째 줄 |
| `.,$` | 현재 줄부터 파일 끝까지 |
| `.,.+5` | 현재 줄부터 5줄 아래까지 |
| `'<,'>` | Visual 모드에서 선택한 범위 (자동 입력됨) |
| `'a,'b` | 마크 a부터 마크 b까지 |

### 플래그 (Flags)

| 플래그 | 의미 |
|--------|------|
| `g` | 줄 내 모든 매칭을 치환 (없으면 줄당 첫 번째만) |
| `c` | 각 치환 전에 확인 (confirm) |
| `i` | 대소문자 무시 |
| `I` | 대소문자 구분 (smartcase 무시) |
| `n` | 치환하지 않고 매칭 개수만 표시 |
| `e` | 매칭이 없어도 에러를 내지 않음 |

### 실전 예시

```vim
" 파일 전체에서 foo를 bar로 치환 (모든 매칭)
:%s/foo/bar/g

" 현재 줄에서만 치환
:s/foo/bar/g

" 확인하면서 치환 (y/n/a/q/l로 응답)
:%s/foo/bar/gc

" 대소문자 무시하여 치환
:%s/foo/bar/gi

" 치환하지 않고 매칭 개수만 확인
:%s/foo/bar/gn

" 10~20번째 줄에서만 치환
:10,20s/foo/bar/g

" Visual 선택 범위에서 치환
'<,'>s/foo/bar/g
```

### 확인 모드 (c 플래그)의 응답

`c` 플래그를 사용하면 각 매칭에서 다음 중 하나를 선택할 수 있다:

| 키 | 동작 |
|----|------|
| `y` | 이 매칭을 치환 |
| `n` | 이 매칭은 건너뜀 |
| `a` | 이 매칭을 포함하여 나머지 모두 치환 |
| `q` | 치환 중단 |
| `l` | 이 매칭을 치환하고 중단 (last) |

### 캡처 그룹과 역참조

`\(` `\)`로 캡처한 그룹을 `\1`, `\2`로 참조할 수 있다. very magic 모드에서는 `()`.

```vim
" 이름 순서 바꾸기: "성 이름" → "이름 성"
:%s/\v(\w+) (\w+)/\2 \1/g

" HTML 태그 변경: <b>text</b> → <strong>text</strong>
:%s/\v\<b\>(.*)\<\/b\>/<strong>\1<\/strong>/g
```

### 특수 치환 패턴

치환 부분(replacement)에서 사용할 수 있는 특수 패턴:

| 패턴 | 의미 |
|------|------|
| `\0` 또는 `&` | 매칭된 전체 문자열 |
| `\1` ~ `\9` | 캡처 그룹 |
| `\U` | 이후 문자를 대문자로 |
| `\L` | 이후 문자를 소문자로 |
| `\u` | 다음 한 문자를 대문자로 |
| `\l` | 다음 한 문자를 소문자로 |
| `\e` 또는 `\E` | `\U` 또는 `\L`의 효과 종료 |
| `\r` | 줄바꿈 |
| `\t` | 탭 |

```vim
" 전체 대문자로 변환
:%s/\v(\w+)/\U\1/g
" hello world → HELLO WORLD

" 첫 글자만 대문자로 (capitalize)
:%s/\v<(\w)(\w*)/\u\1\L\2/g
" hello world → Hello World

" 매칭된 문자열을 괄호로 감싸기
:%s/\v\d+/(&)/g
" age: 30 → age: (30)

" snake_case를 camelCase로 변환
:%s/\v_(\w)/\u\1/g
" user_name → userName
```

## :g (Global) 명령

`:g` 명령은 **패턴이 매칭되는 모든 줄에 명령을 실행**한다. `:s`와는 다르다 -- `:s`는 텍스트를 치환하지만, `:g`는 매칭된 줄에 임의의 Ex 명령을 실행할 수 있다.

### 기본 문법

```vim
:[range]g/pattern/command          " 패턴이 매칭되는 줄에 명령 실행
:[range]g!/pattern/command         " 패턴이 매칭되지 않는 줄에 명령 실행
:[range]v/pattern/command          " g!와 동일 (v = inverse)
```

### :g 실전 예시

```vim
" 패턴이 있는 줄 삭제
:g/console\.log/d

" 빈 줄 모두 삭제
:g/^\s*$/d

" 패턴이 없는 줄 삭제 (= 패턴이 있는 줄만 남기기)
:v/ERROR/d

" 패턴이 있는 줄을 파일 끝으로 이동
:g/TODO/m$

" 패턴이 있는 줄을 복사하여 파일 끝에 추가
:g/FIXME/t$

" 패턴이 있는 줄에서 치환 수행
:g/class/s/old/new/g

" 패턴이 있는 줄에 들여쓰기 추가
:g/return/normal >>

" 패턴이 있는 줄을 줄 번호와 함께 표시
:g/TODO/p
```

### :g와 :s의 결합

`:g`의 진가는 다른 명령과 결합할 때 발휘된다:

```vim
" 주석이 아닌 줄에서만 치환
:g/^\s*[^/]/s/foo/bar/g

" 특정 함수 안에서만 치환 (function과 } 사이)
:/function processData/,/^}/s/data/input/g

" TODO 주석을 DONE으로 변경하고 날짜 추가
:g/TODO/s/TODO/DONE 2024-01-15/
```

### :g로 줄 정리

```vim
" 연속된 빈 줄을 하나로 줄이기 (3개 이상 연속 개행 → 2개로)
:%s/\v\n{3,}/\r\r/

" 줄 끝 공백 제거
:%s/\s\+$//e

" 특정 패턴이 있는 줄만 남기기 (나머지 삭제)
:v/pattern/d

" 줄 순서 뒤집기
:g/^/m0
```

## vimgrep과 quickfix list

여러 파일에 걸쳐 검색할 때는 `:vimgrep`을 사용한다. 검색 결과는 quickfix list에 저장된다.

### :vimgrep 기본

```vim
:vimgrep /pattern/ **/*.js          " 모든 js 파일에서 검색
:vimgrep /TODO/ %                   " 현재 파일에서 검색
:vimgrep /\vfunction\s+\w+/ **/*.ts " 정규식으로 검색
```

`**`는 하위 디렉토리를 재귀적으로 포함한다는 의미다.

### quickfix list 탐색

`:vimgrep` 결과는 quickfix list에 저장된다:

| 명령 | 동작 |
|------|------|
| `:copen` | quickfix 창 열기 |
| `:cclose` | quickfix 창 닫기 |
| `:cnext` 또는 `:cn` | 다음 결과로 이동 |
| `:cprev` 또는 `:cp` | 이전 결과로 이동 |
| `:cfirst` | 첫 번째 결과로 이동 |
| `:clast` | 마지막 결과로 이동 |
| `:cc N` | N번째 결과로 이동 |

```vim
" 실전 워크플로우
:vimgrep /TODO/ **/*.lua        " 모든 lua 파일에서 TODO 검색
:copen                          " quickfix 창 열기
                                " Enter로 해당 위치로 이동
:cnext                          " 다음 결과로
:cprev                          " 이전 결과로
```

quickfix 창에서는 `j`/`k`로 이동하고, `<Enter>`로 해당 파일의 해당 위치로 점프한다.

> **팁**: quickfix list 탐색에 키맵을 설정하면 편리하다:
> ```lua
> vim.keymap.set("n", "]q", "<cmd>cnext<CR>")
> vim.keymap.set("n", "[q", "<cmd>cprev<CR>")
> ```

### :grep과 외부 프로그램

`:vimgrep`은 Vim 내장 기능이라 느릴 수 있다. 대규모 프로젝트에서는 외부 프로그램을 활용하는 `:grep`이 더 빠르다:

```lua
-- ripgrep을 grepprg로 설정
vim.opt.grepprg = "rg --vimgrep --smart-case"
vim.opt.grepformat = "%f:%l:%c:%m"
```

설정 후:

```vim
:grep "pattern" **/*.js         " ripgrep으로 검색
:copen                          " 결과를 quickfix list로
```

## 검색과 치환 실전 워크플로우

### 워크플로우 1: 변수명 일괄 변경

프로젝트 전체에서 `userName`을 `username`으로 변경:

```vim
" 1. 먼저 검색으로 확인
/userName

" 2. 매칭 개수 확인
:%s/userName/username/gn

" 3. 확인하면서 치환
:%s/userName/username/gc
```

### 워크플로우 2: 로그 정리

```vim
" console.log 줄 모두 삭제
:g/console\.log/d

" 또는 주석 처리
:g/console\.log/normal I//
```

### 워크플로우 3: 코드 포맷 변환

```vim
" 큰따옴표를 작은따옴표로 변환
:%s/"/'/g

" 화살표 함수를 일반 함수로
:%s/\v(\w+)\s*\=\s*\(([^)]*)\)\s*\=\>\s*\{/function \1(\2) {/g
```

## 실습: 검색과 치환 연습

다음 코드를 파일로 만들고 연습하자:

```javascript
var firstName = "John";
var lastName = "Doe";
var email = "john@example.com";
var age = 30;
var isActive = true;

function getUserName(first_name, last_name) {
  console.log("Getting user name");
  var full_name = first_name + " " + last_name;
  console.log("Full name: " + full_name);
  return full_name;
}

function getEmail(user_id) {
  console.log("Fetching email for user: " + user_id);
  var email = "user@example.com";
  return email;
}

// TODO: Add validation
// TODO: Add error handling
// FIXME: Memory leak in getUserName
```

연습 과제:

1. `/var`로 검색한 뒤 `n`/`N`으로 이동해 보라
2. `*`로 `email` 단어를 검색해 보라
3. `:%s/var/const/g`로 모든 `var`를 `const`로 치환하라
4. `:%s/\v_(\w)/\u\1/g`로 snake_case를 camelCase로 변환하라
5. `:g/console\.log/d`로 모든 `console.log` 줄을 삭제하라
6. `:g/TODO/p`로 TODO가 있는 줄을 확인하라
7. `:%s/\v\/\/ (TODO|FIXME): (.+)/\/\/ DONE: \2/g`로 TODO와 FIXME를 DONE으로 변경하라
8. `:%s/\v"([^"]+)"/'\1'/g`로 큰따옴표를 작은따옴표로 변환하라

## 핵심 정리

- **`/`**(앞으로), **`?`**(뒤로) 검색, **`n`**/**`N`**으로 탐색, **`*`**/**`#`**으로 커서 단어 검색
- **`ignorecase`** + **`smartcase`** 조합이 가장 편리한 대소문자 처리
- **`\v`** (very magic)로 시작하면 정규표현식 이스케이프 부담이 줄어든다
- **`:%s/old/new/g`**가 기본 치환, **`c`** 플래그로 확인하면서 치환
- **캡처 그룹** `\1`, `\2`와 **대소문자 변환** `\U`, `\L`, `\u`, `\l`로 고급 치환 가능
- **`:g/pattern/command`**는 매칭되는 줄에 임의의 명령 실행 -- `:s`보다 범용적
- **`:v/pattern/command`**는 매칭되지 않는 줄에 명령 실행 (`:g!`과 동일)
- **`:vimgrep`**으로 여러 파일 검색, 결과는 **quickfix list**에서 `:cnext`/`:cprev`로 탐색
- 대규모 프로젝트에서는 `grepprg`을 **ripgrep** 등 외부 도구로 설정하면 빠르다
