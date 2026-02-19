# Chapter 10: 마크와 점프

코드를 편집할 때 "여기 표시해 두고 나중에 돌아와야지"라는 생각을 자주 한다. 또 "방금 전에 보던 위치로 돌아가고 싶다"는 상황도 흔하다. 마크(mark)와 점프(jump)는 이런 요구를 정확히 해결한다.

마크는 위치를 직접 지정하는 "북마크"이고, 점프 리스트와 변경 리스트는 Neovim이 자동으로 관리하는 "이동 히스토리"다. 이 둘을 결합하면 파일 안에서, 그리고 파일 사이에서 자유자재로 움직일 수 있다.

## 마크 기본

### 마크 설정과 이동

마크를 설정하려면 `m` + 문자, 마크로 이동하려면 `'` 또는 `` ` `` + 문자를 사용한다.

| 명령 | 동작 |
|------|------|
| `m{a-zA-Z}` | 현재 커서 위치에 마크 설정 |
| `'{mark}` | 마크가 설정된 **줄의 첫 비공백 문자**로 이동 |
| `` `{mark} `` | 마크가 설정된 **정확한 위치**(줄 + 열)로 이동 |

`'`(작은따옴표)와 `` ` ``(백틱)의 차이는 중요하다:

```javascript
function processData(input) {
  const result = transform(input);    // ← ma로 여기에 마크 (커서가 result의 r 위)
  validate(result);
  return result;
}
```

- `'a` → 마크 `a`가 있는 줄의 첫 비공백 문자(`c`)로 이동
- `` `a `` → 마크 `a`의 정확한 위치(`r`)로 이동

코드 편집에서는 정확한 위치로 돌아가는 `` ` ``을 더 자주 사용한다.

### 마크 목록 확인

```vim
:marks                    " 모든 마크 목록
:marks aB                 " a와 B 마크만 확인
```

출력 예시:

```
mark line  col file/text
 a     42    5 const result = transform(input);
 B     10    0 ~/project/src/main.js
 '     28    0 return processData(input);
 .     42   12 const result = transform(input);
```

### 마크 삭제

```vim
:delmarks a               " a 마크 삭제
:delmarks a-d             " a, b, c, d 마크 삭제
:delmarks!                " 소문자 마크(a-z) 전부 삭제
```

## 로컬 마크 vs 글로벌 마크

### 로컬 마크 (a-z)

소문자 마크(`a`~`z`)는 **버퍼(파일)별로 독립적**이다.

- `main.js`에서 `ma` → `main.js`의 마크 `a`
- `utils.js`에서 `ma` → `utils.js`의 마크 `a` (서로 독립)

로컬 마크는 한 파일 안에서 여러 위치를 오갈 때 사용한다:

```
ma     → 함수 시작에 마크 설정
...    → 다른 곳으로 이동해서 작업
`a     → 함수 시작으로 돌아오기
```

### 글로벌 마크 (A-Z)

대문자 마크(`A`~`Z`)는 **파일과 위치를 함께 기억**한다. 다른 파일에서도 해당 마크로 점프하면 원래 파일이 열린다.

```
" main.js에서
mA     → main.js의 현재 위치에 글로벌 마크 A 설정

" 다른 파일로 이동해서 작업한 뒤
`A     → main.js의 마크 A 위치로 돌아온다
```

글로벌 마크의 활용:

| 마크 | 용도 예시 |
|------|-----------|
| `M` | 메인 진입점 (main.js, index.ts 등) |
| `T` | 현재 작업 중인 테스트 파일 |
| `C` | 설정 파일 (config) |
| `R` | 라우터/라우팅 파일 |

자주 참고하는 파일들에 글로벌 마크를 설정해 두면, 어디서든 한 번의 키 입력으로 이동할 수 있다.

> **팁**: 글로벌 마크는 Neovim 세션 간에도 유지된다 (shada 파일에 저장). Neovim을 종료했다가 다시 열어도 마크가 남아 있다.

## 특수 마크

Neovim은 몇 가지 마크를 자동으로 설정한다. 직접 설정할 수는 없지만 이동에 사용할 수 있다.

### 자주 쓰는 특수 마크

| 마크 | 의미 |
|------|------|
| `` `` `` 또는 `''` | 마지막 점프 전 위치 |
| `` `. `` 또는 `'.` | 마지막 편집 위치 |
| `` `^ `` 또는 `'^` | 마지막으로 Insert 모드를 빠져나온 위치 |
| `` `[ `` | 마지막으로 변경/붙여넣기한 텍스트의 시작 |
| `` `] `` | 마지막으로 변경/붙여넣기한 텍스트의 끝 |
| `` `< `` | 마지막 Visual 선택의 시작 |
| `` `> `` | 마지막 Visual 선택의 끝 |
| `` `" `` | 마지막으로 이 버퍼를 떠난 위치 |

### 실전 활용

**`` `` ``(마지막 점프 전 위치)**: 가장 자주 사용하는 특수 마크다. `gg`로 파일 맨 위로 갔다가 `` `` ``으로 원래 위치로 돌아올 수 있다.

```
" 작업하던 중
gg         → 파일 맨 위로 점프
...        → 위쪽에서 뭔가 확인
``         → 작업하던 곳으로 복귀
```

**`` `. ``(마지막 편집 위치)**: 코드를 수정한 뒤 다른 곳을 둘러보다가, 다시 편집하던 곳으로 돌아갈 때 유용하다.

```
" 함수 안에서 코드를 수정
...        → /import로 상단 import 확인
`.         → 수정하던 곳으로 정확히 돌아오기
```

**`gi`(마지막 Insert 위치에서 Insert 모드 진입)**: `` `^ `` 위치로 이동하면서 동시에 Insert 모드에 진입한다. 입력하다가 Normal 모드로 나와서 다른 곳을 확인한 뒤, `gi`로 입력하던 곳에서 계속 입력할 수 있다.

```
" Insert 모드에서 코드 작성 중 → <Esc>
...        → 다른 곳에서 변수명 확인
gi         → 입력하던 곳으로 돌아가면서 Insert 모드 진입
```

**`` `[ ``와 `` `] ``(마지막 변경 범위)**: 붙여넣기 후 붙여넣은 영역을 선택하거나 들여쓰기를 조절할 때 유용하다.

```vim
p                         " 텍스트 붙여넣기
`[v`]                     " 붙여넣은 텍스트를 Visual로 선택
`[v`]>                    " 붙여넣은 텍스트를 들여쓰기
```

## 점프 리스트

### 점프란 무엇인가

Neovim에서 "점프"는 일반적인 이동과 다르다. `j`, `k`, `w`, `b` 같은 **소규모 이동은 점프가 아니다**. 다음과 같은 **대규모 이동이 점프**다:

- `gg`, `G`, `{n}G` (파일 내 큰 이동)
- `/{pattern}`, `?{pattern}`, `n`, `N` (검색)
- `%` (매칭 괄호)
- `(`, `)` (문장 이동)
- `{`, `}` (단락 이동)
- `H`, `M`, `L` (화면 내 위치)
- `'{mark}`, `` `{mark} `` (마크 이동)
- `<C-]>` (태그 점프)
- `:e {file}` (파일 열기)

점프가 발생할 때마다 Neovim은 점프 전 위치를 **점프 리스트(jump list)**에 기록한다.

### 점프 리스트 이동

| 명령 | 동작 |
|------|------|
| `<C-o>` | 점프 리스트에서 이전 위치로 (older) |
| `<C-i>` | 점프 리스트에서 다음 위치로 (newer) |
| `:jumps` | 점프 리스트 확인 |

`:jumps` 출력 예시:

```
 jump line  col file/text
   4    15    0 ~/project/src/utils.js
   3    42    5 const result = transform(input);
   2     1    0 import express from 'express';
   1    28   12 return processData(input);
>  0    35    8 app.listen(PORT, () => {
   1    10    0 const app = express();
```

`>`는 현재 위치를 나타낸다. 위쪽이 이전(older), 아래쪽이 이후(newer)다.

### 점프 리스트의 실전 활용

점프 리스트는 **웹 브라우저의 뒤로 가기/앞으로 가기**와 같다.

일반적인 코드 탐색 흐름:

```
main.js의 35번째 줄에서 작업 중
  → /processData로 검색하여 28번째 줄로 점프     (점프 1)
  → gd로 정의 위치인 utils.js의 15번째 줄로 점프  (점프 2)
  → gg로 utils.js 맨 위의 import 확인              (점프 3)

이제 돌아가려면:
  <C-o>  → utils.js의 15번째 줄
  <C-o>  → main.js의 28번째 줄
  <C-o>  → main.js의 35번째 줄 (원래 위치)
```

`<C-o>`와 `<C-i>`를 자유롭게 사용하면, 여러 파일을 점프하면서 탐색한 뒤 정확하게 원래 위치로 돌아올 수 있다.

## 변경 리스트

### 변경 리스트란

변경 리스트(change list)는 **편집이 일어난 위치들의 히스토리**다. 점프 리스트가 "이동한 곳"을 기록한다면, 변경 리스트는 "수정한 곳"을 기록한다.

| 명령 | 동작 |
|------|------|
| `g;` | 변경 리스트에서 이전 변경 위치로 (older) |
| `g,` | 변경 리스트에서 다음 변경 위치로 (newer) |
| `:changes` | 변경 리스트 확인 |

`:changes` 출력 예시:

```
change line  col text
    3    42    5 const result = calculate();
    2    15    0 import { utils } from './utils';
    1    28   12 return processData(input);
>   0    28   12 return processData(input);
```

### 변경 리스트 vs 점프 리스트

| 특성 | 점프 리스트 | 변경 리스트 |
|------|-----------|-----------|
| 기록 대상 | 대규모 이동 | 편집(변경) |
| 이동 키 | `<C-o>`, `<C-i>` | `g;`, `g,` |
| 범위 | 파일 간 이동 포함 | 현재 버퍼 내 |
| 확인 명령 | `:jumps` | `:changes` |

변경 리스트는 "아까 수정한 곳이 어디였지?"라는 질문에 답한다. 파일 여러 곳을 수정하고 다니다가, `g;`를 반복하면 수정한 곳들을 역순으로 방문할 수 있다.

> **팁**: `g;`는 `` `. ``과 비슷하지만, `` `. ``은 마지막 한 곳만 기억하는 반면 `g;`는 **변경 히스토리를 따라** 계속 이전 변경 위치로 이동할 수 있다.

## 태그 점프

### 태그란

태그(tag)는 함수, 클래스, 변수 등의 **정의 위치**를 가리키는 색인이다. 전통적으로 `ctags` 같은 도구로 태그 파일을 생성했지만, 요즘은 LSP(Language Server Protocol)가 더 정확한 정의 탐색을 제공한다.

### 태그 점프 명령

| 명령 | 동작 |
|------|------|
| `<C-]>` | 커서 아래 단어의 태그(정의)로 점프 |
| `<C-t>` | 태그 스택에서 이전 위치로 돌아가기 |
| `:tag {name}` | 이름으로 태그 점프 |
| `:tn` | 같은 이름의 다음 태그 |
| `:tp` | 같은 이름의 이전 태그 |
| `:ts {name}` | 태그 목록에서 선택 |

태그 점프는 독자적인 "태그 스택(tag stack)"을 유지한다. `<C-]>`로 정의로 점프한 뒤, `<C-t>`로 돌아오는 것은 점프 리스트와 독립적이다.

```
main.js에서 processData 함수를 호출하는 곳
  → <C-]>로 processData의 정의로 점프
  → 정의를 확인
  → <C-t>로 호출 위치로 돌아오기
```

> **참고**: LSP가 설정되어 있으면 `<C-]>` 대신 LSP의 "Go to Definition" 기능을 사용하는 것이 더 정확하다. Part 4에서 LSP 설정을 다룬다.

## gd와 gD

태그 파일이나 LSP 없이도 간단한 정의 탐색이 가능하다:

| 명령 | 동작 |
|------|------|
| `gd` | 로컬 정의로 이동 (현재 함수/블록 내에서 검색) |
| `gD` | 글로벌 정의로 이동 (파일 전체에서 검색) |

`gd`는 커서 아래 단어의 **첫 번째 출현 위치**를 현재 함수 범위에서 찾는다. `gD`는 파일 전체에서 찾는다.

```javascript
const globalConfig = {};              // ← gD는 여기로 이동

function setup() {
  const localVar = 42;                // ← gd는 여기로 이동
  console.log(localVar);              //   커서가 여기에 있을 때
  return globalConfig;                //   gD를 누르면 맨 위로
}
```

이 기능은 언어 서버 없이도 작동하므로, 간단한 탐색에 유용하다. 다만 정확도는 LSP보다 떨어진다.

## 실전 활용: 코드 탐색 워크플로우

### 워크플로우 1: 글로벌 마크로 핵심 파일 즐겨찾기

프로젝트를 시작할 때 핵심 파일에 글로벌 마크를 설정한다:

```
" 각 파일을 열고 마크 설정
mM     → main entry point (main.js)
mR     → router (routes.js)
mC     → config (config.js)
mT     → 현재 작업 중인 테스트 파일
```

이후 어디서든 `` `M ``으로 메인 파일로, `` `R ``로 라우터로 즉시 이동한다.

### 워크플로우 2: 점프 리스트로 탐색 후 복귀

```
1. 코드 작성 중 (35번째 줄)
2. 다른 함수의 동작을 확인해야 함
3. /functionName으로 검색하여 정의로 이동
4. 그 함수가 호출하는 다른 함수도 확인 → <C-]>로 점프
5. 파악 완료, 원래 작업으로 복귀
6. <C-o> <C-o> <C-o> → 원래 35번째 줄로 돌아옴
```

### 워크플로우 3: 로컬 마크로 편집 지점 표시

하나의 파일에서 여러 곳을 수정해야 할 때:

```
1. 첫 번째 수정 지점에 ma 설정
2. 두 번째 수정 지점에 mb 설정
3. 세 번째 수정 지점에 mc 설정
4. `a → 수정 → `b → 수정 → `c → 수정
5. `a로 돌아가서 결과 확인
```

### 워크플로우 4: 변경 리스트로 수정 지점 순회

리팩토링 후 수정한 곳들을 검토할 때:

```
1. 여러 곳에서 코드를 수정
2. g; g; g; 으로 수정한 곳들을 역순으로 방문
3. 각 수정이 올바른지 확인
4. g, g, g, 으로 앞으로 이동하며 다시 확인
```

## 실습: 마크와 점프 연습

다음 파일을 만들고 연습해 보자:

```javascript
// navigation.js
import { Router } from 'express';
import { authenticate } from './middleware/auth';
import { validate } from './middleware/validate';
import { UserService } from './services/user';

const router = Router();
const userService = new UserService();

router.get('/users', authenticate, async (req, res) => {
  const users = await userService.findAll();
  res.json(users);
});

router.get('/users/:id', authenticate, async (req, res) => {
  const user = await userService.findById(req.params.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
});

router.post('/users', authenticate, validate, async (req, res) => {
  const newUser = await userService.create(req.body);
  res.status(201).json(newUser);
});

router.put('/users/:id', authenticate, validate, async (req, res) => {
  const updated = await userService.update(req.params.id, req.body);
  res.json(updated);
});

router.delete('/users/:id', authenticate, async (req, res) => {
  await userService.delete(req.params.id);
  res.status(204).send();
});

export default router;
```

연습 과제:

1. 첫 번째 `router.get`에 `ma`, `router.post`에 `mb`, `router.delete`에 `mc` 마크 설정
2. `` `a ``로 이동 → `` `c ``로 이동 → `` `b ``로 이동하며 마크 점프 확인
3. `gg`로 파일 맨 위 이동 → `` `` ``로 이전 위치로 복귀
4. `/findAll`로 검색 점프 → `<C-o>`로 이전 위치로 → `<C-i>`로 다시 앞으로
5. `gd`를 `userService` 위에서 눌러 로컬 정의로 이동
6. 아무 곳에서나 텍스트 수정 (예: `ciw`로 단어 변경) → 다른 곳으로 이동 → `` `. ``으로 편집 위치로 복귀
7. 여러 곳에서 수정 → `g;`로 변경 지점들을 역순 방문
8. `:jumps`로 점프 리스트 확인 → `:changes`로 변경 리스트 확인
9. `mA`로 글로벌 마크 설정 → 다른 파일을 열고 → `` `A ``로 원래 파일의 마크 위치로 복귀

## 핵심 정리

- **`m{a-z}`**: 로컬 마크 설정 (파일별 독립), **`m{A-Z}`**: 글로벌 마크 설정 (파일 + 위치 기억)
- **`'{mark}`**: 마크 줄의 첫 비공백 문자로, **`` `{mark} ``**: 정확한 위치(줄 + 열)로 이동
- 특수 마크: **`` `` ``** (직전 점프 위치), **`` `. ``** (마지막 편집 위치), **`gi`** (마지막 Insert 위치에서 입력 재개)
- **점프 리스트**: `<C-o>` (이전), `<C-i>` (다음) -- 브라우저의 뒤로/앞으로 가기
- **변경 리스트**: `g;` (이전 변경), `g,` (다음 변경) -- 수정한 곳들을 순회
- **`<C-]>`**: 태그(정의) 점프, **`<C-t>`**: 태그 스택에서 복귀
- **`gd`**: 로컬 정의로 이동, **`gD`**: 글로벌 정의로 이동
- 글로벌 마크를 핵심 파일에 설정해 두면 프로젝트 탐색이 획기적으로 빨라진다
