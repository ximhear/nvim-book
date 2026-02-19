# Chapter 9: Command-line 모드 심화

Chapter 1에서 `:w`, `:q` 같은 기본적인 Command-line 명령을 다뤘다. 이 챕터에서는 Command-line 모드의 본질인 Ex 명령 체계를 깊이 파고들고, 범위 지정, 외부 명령 연동, 명령줄 편집 기법을 익힌다.

Ex 명령을 잘 다루면 반복적인 편집 작업을 한 줄로 처리할 수 있다.

## Ex 명령어 체계

### 역사와 기본 구조

Vim의 Command-line 명령은 1976년에 만들어진 `ex` 편집기에서 유래했다. `ex`는 화면 없이 한 줄씩 편집하는 라인 편집기(line editor)였고, `vi`는 `ex`에 시각적(visual) 인터페이스를 추가한 것이다.

그래서 `:` 뒤에 입력하는 명령을 "Ex 명령"이라 부른다. 이 유산 덕분에 Vim은 **줄 단위 일괄 처리**에 매우 강하다.

Ex 명령의 기본 구조:

```
:[범위]{명령} [인자]
```

예를 들어 `:1,5d`는 "1번째 줄부터 5번째 줄까지 삭제"를 의미한다.

## 범위 지정

범위(range)는 Ex 명령이 적용될 줄의 범위를 지정한다. 범위를 자유자재로 다루는 것이 Ex 명령의 핵심이다.

### 기본 범위 기호

| 기호 | 의미 |
|------|------|
| `.` | 현재 줄 |
| `$` | 마지막 줄 |
| `%` | 파일 전체 (1,$ 와 동일) |
| `{n}` | n번째 줄 |
| `'<` | Visual 선택의 시작 줄 |
| `'>` | Visual 선택의 끝 줄 |

### 범위 조합

| 범위 | 의미 |
|------|------|
| `:5` | 5번째 줄 |
| `:1,5` | 1~5번째 줄 |
| `:.,$` | 현재 줄부터 마지막 줄까지 |
| `:%` | 파일 전체 |
| `:.,+5` | 현재 줄부터 아래로 5줄 |
| `:.-3,.` | 현재 줄 위 3줄부터 현재 줄까지 |
| `:'<,'>` | Visual 선택 범위 |

`.+5`는 "현재 줄에서 5줄 아래"를 의미한다. 이런 상대 주소를 오프셋(offset)이라 한다.

### 패턴 기반 범위

특정 패턴이 있는 줄을 범위의 시작이나 끝으로 지정할 수 있다:

| 범위 | 의미 |
|------|------|
| `:/function/` | 다음에 나오는 "function"이 있는 줄 |
| `:?return?` | 이전에 나오는 "return"이 있는 줄 |
| `:/function/,/^}/` | "function"이 나오는 줄부터 "}"로 시작하는 줄까지 |

실전 예시:

```vim
:/function/,/^}/d          " 다음 함수 전체를 삭제
:?import?,/^$/d            " 이전 import부터 빈 줄까지 삭제
```

### Visual 선택 후 범위

Visual 모드에서 영역을 선택하고 `:`를 누르면 자동으로 `:'<,'>`가 입력된다. 이것은 선택 범위에 Ex 명령을 적용하는 가장 직관적인 방법이다.

```vim
" Visual 모드로 3줄 선택 후 :
:'<,'>d          " 선택 범위 삭제
:'<,'>s/old/new/g   " 선택 범위에서 치환
:'<,'>!sort      " 선택 범위를 sort로 정렬
```

## 유용한 Ex 명령

### :read - 파일 읽어 삽입

`:read`(줄여서 `:r`)는 파일이나 명령의 출력을 현재 버퍼에 삽입한다.

```vim
:r filename.txt           " 파일 내용을 현재 줄 아래에 삽입
:0r filename.txt          " 파일 내용을 맨 처음에 삽입
:$r filename.txt          " 파일 내용을 맨 끝에 삽입
```

### :write - 범위를 파일로 저장

`:write`(줄여서 `:w`)에 범위를 지정하면 일부분만 다른 파일로 저장할 수 있다.

```vim
:w                        " 전체 파일 저장
:10,20w part.txt          " 10~20번째 줄을 part.txt로 저장
:'<,'>w selection.txt     " Visual 선택 범위를 파일로 저장
:w >>append.txt           " 현재 파일 내용을 append.txt에 추가
```

### :move와 :copy - 줄 이동과 복사

```vim
:5m 12                    " 5번째 줄을 12번째 줄 아래로 이동
:5,10m $                  " 5~10번째 줄을 파일 끝으로 이동
:5t 12                    " 5번째 줄을 12번째 줄 아래에 복사 (copy의 약자는 t)
:5,10t 0                  " 5~10번째 줄을 파일 맨 처음에 복사
:.t.                      " 현재 줄을 바로 아래에 복사 (줄 복제)
```

> **팁**: `:t.`(현재 줄 복제)은 `yyp`와 동일하지만, 레지스터를 건드리지 않는다는 장점이 있다.

### :global - 패턴 매칭 줄에 명령 실행

`:global`(줄여서 `:g`)은 패턴에 매칭되는 모든 줄에 Ex 명령을 실행한다. 매우 강력한 명령이다.

```vim
:g/pattern/command
```

실전 예시:

```vim
:g/TODO/d                 " TODO가 포함된 줄 모두 삭제
:g/^$/d                   " 빈 줄 모두 삭제
:g/console\.log/d         " console.log가 있는 줄 모두 삭제
:g/^import/m 0            " 모든 import 줄을 파일 맨 위로 이동
:g/function/p             " function이 포함된 줄 모두 출력
```

`:g!` 또는 `:v`는 패턴에 매칭되지 **않는** 줄에 명령을 실행한다:

```vim
:v/pattern/d              " pattern이 없는 줄 모두 삭제
:g!/error/d               " error가 없는 줄 모두 삭제
```

## 외부 명령 연동

### 기본 실행

```vim
:!ls                      " ls 명령 실행, 결과를 화면에 표시
:!node %                  " 현재 파일을 node로 실행 (%는 현재 파일명)
:!mkdir -p src/components " 디렉토리 생성
```

`%`는 현재 파일 경로로 확장된다. `#`은 이전(alternate) 파일 경로로 확장된다.

### 외부 명령의 출력 삽입

```vim
:r !date                  " date 명령의 출력을 현재 줄 아래에 삽입
:r !ls src/               " ls의 출력을 삽입
:r !curl -s https://example.com/api  " API 응답을 삽입
```

### 필터링: 선택 영역을 외부 명령으로 처리

필터링(filtering)은 Ex 명령의 가장 강력한 기능 중 하나다. 범위를 지정하고 외부 명령을 실행하면, **해당 범위의 텍스트가 명령의 입력으로 전달되고, 출력으로 교체**된다.

```vim
:{range}!{command}
```

#### 실전 예시: 선택 영역 정렬

```
banana
cherry
apple
date
```

이 네 줄을 Visual 모드로 선택한 뒤 `!sort`:

```vim
:'<,'>!sort
```

결과:

```
apple
banana
cherry
date
```

#### 실전 예시: JSON 포매팅

```json
{"name":"Alice","age":30,"skills":["js","python"],"address":{"city":"Seoul"}}
```

이 줄에서 `:.!jq .` 또는 Visual 선택 후 `!jq .`:

```json
{
  "name": "Alice",
  "age": 30,
  "skills": [
    "js",
    "python"
  ],
  "address": {
    "city": "Seoul"
  }
}
```

#### 실전 예시: 숫자 열 합계

```
100
200
350
150
```

Visual 선택 후 `!paste -sd+ | bc`:

```
800
```

#### 실전 예시: 중복 제거

```vim
:'<,'>!sort -u            " 정렬 후 중복 제거
:'<,'>!uniq               " 연속 중복만 제거 (정렬 불필요)
```

#### Normal 모드에서 필터링

`!`를 오퍼레이터로 사용하면 모션과 조합할 수 있다:

```
!ip                       " 현재 단락을 필터링 → 외부 명령 입력 대기
!!                        " 현재 줄을 필터링
!3j                       " 현재 줄부터 아래 3줄을 필터링
```

`!ip`를 누르면 하단에 `:.,.+{n}!`이 나타나고, 외부 명령을 입력할 수 있다.

## 명령줄 편집

Command-line 모드에서도 다양한 편집 키를 사용할 수 있다.

### 이동과 편집

| 키 | 동작 |
|----|------|
| `<Left>`, `<Right>` | 커서 이동 |
| `<C-b>` | 명령줄 맨 처음으로 |
| `<C-e>` | 명령줄 맨 끝으로 |
| `<C-w>` | 커서 앞 단어 삭제 |
| `<C-u>` | 커서 앞 전체 삭제 |

### 레지스터 삽입

명령줄에서 레지스터의 내용을 삽입할 수 있다:

| 키 | 동작 |
|----|------|
| `<C-r>"` | 기본 레지스터(마지막 삭제/복사) 내용 삽입 |
| `<C-r>0` | 0번 레지스터(마지막 복사) 내용 삽입 |
| `<C-r><C-w>` | 커서 아래 단어(word) 삽입 |
| `<C-r><C-a>` | 커서 아래 WORD 삽입 |
| `<C-r>/` | 마지막 검색 패턴 삽입 |
| `<C-r>%` | 현재 파일명 삽입 |

`<C-r><C-w>`는 치환 명령에서 특히 유용하다. 커서를 변경하고 싶은 단어 위에 놓고:

```vim
:%s/<C-r><C-w>/newName/g
```

이렇게 하면 커서 아래 단어를 직접 타이핑하지 않고 치환 대상으로 지정할 수 있다.

### 히스토리 탐색

| 키 | 동작 |
|----|------|
| `<Up>` | 이전 명령 (현재 입력에 매칭되는 것만) |
| `<Down>` | 다음 명령 (현재 입력에 매칭되는 것만) |
| `q:` | 명령 히스토리 창 열기 (Normal 모드에서) |
| `q/` | 검색 히스토리 창 열기 |
| `<C-f>` | 명령줄에서 히스토리 창으로 전환 |

`<Up>`과 `<Down>`은 단순한 이전/다음이 아니다. 현재 입력한 텍스트로 시작하는 명령만 순환한다. 예를 들어 `:s`를 입력한 상태에서 `<Up>`을 누르면 `:s`로 시작하는 이전 명령만 보여준다.

### 명령 히스토리 창 (q:)

`q:`를 누르면 이전에 실행한 Ex 명령들이 일반 버퍼처럼 열린다. 여기서 Normal 모드의 모든 편집 명령을 사용하여 이전 명령을 수정하고 `<Enter>`로 실행할 수 있다.

```
:g/console/d
:%s/foo/bar/g
:w
:e ~/.config/nvim/init.lua
```

이 히스토리에서 `k`로 원하는 명령으로 이동, 필요하면 편집한 뒤 `<Enter>`로 실행한다. `<C-c>`나 `:q`로 히스토리 창을 닫는다.

> **주의**: `q:`(명령 히스토리)와 `:q`(종료)를 혼동하지 않도록 주의한다. `q:`를 실수로 열었으면 `<C-c>`로 닫으면 된다.

## 유용한 Ex 명령 모음

### :earlier / :later - 시간 기반 undo

```vim
:earlier 10m              " 10분 전 상태로
:earlier 1h               " 1시간 전 상태로
:earlier 5                " 5번의 변경 전 상태로
:later 10m                " 10분 후 상태로 (earlier 후 되돌리기)
:later 5                  " 5번의 변경 후 상태로
```

"30분 전에 지운 코드가 필요해"라는 상황에서 `:earlier 30m`은 생명줄이 된다. `u`를 반복하는 것보다 훨씬 정확하다.

### :changes - 변경 위치 목록

```vim
:changes                  " 변경이 일어난 위치들의 목록
```

출력 예시:

```
change line  col text
    3    42    5 const result = calculate();
    2    15    0 import { utils } from './utils';
    1    28   12 return processData(input);
>   0    28   12 return processData(input);
```

`g;`와 `g,`로 변경 위치를 이동할 수 있다 (Chapter 10에서 자세히 다룬다).

### :jumps - 점프 위치 목록

```vim
:jumps                    " 점프 히스토리
```

`<C-o>`와 `<C-i>`로 점프 위치를 이동한다 (역시 Chapter 10에서 자세히 다룬다).

### :registers - 레지스터 내용 확인

```vim
:registers                " 모든 레지스터의 내용 확인
:reg a b c                " a, b, c 레지스터만 확인
```

### :marks - 마크 목록

```vim
:marks                    " 설정된 마크 목록
```

## 실습: Command-line 모드 연습

다음 파일을 만들고 연습해 보자:

```javascript
// app.js
import express from 'express';
import cors from 'cors';
import { logger } from './utils/logger';

const app = express();
const PORT = 3000;

// TODO: Add authentication middleware
app.use(cors());
app.use(express.json());

app.get('/users', (req, res) => {
  console.log('GET /users');
  const users = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 },
    { name: 'Charlie', age: 35 },
    { name: 'Diana', age: 28 },
    { name: 'Eve', age: 22 },
  ];
  res.json(users);
});

app.get('/health', (req, res) => {
  console.log('Health check');
  res.json({ status: 'ok' });
});

// TODO: Add error handling
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

연습 과제:

1. `:5m 0`으로 5번째 줄(`const app`)을 파일 맨 위로 이동 → `u`로 되돌리기
2. `:%s/console\.log/logger.info/g`로 모든 `console.log`를 `logger.info`로 치환
3. `:g/TODO/d`로 모든 TODO 주석 삭제
4. users 배열의 내용을 Visual 선택 → `:'<,'>!sort`로 이름순 정렬
5. `{"status":"ok"}` 부분을 Visual 선택 → `:'<,'>!jq .`로 포매팅 (jq가 설치된 경우)
6. `:10,20t $`로 10~20번째 줄을 파일 끝에 복사
7. `q:`로 명령 히스토리 창 열기 → 이전 명령 확인 후 `<C-c>`로 닫기
8. `/users`를 검색한 뒤, `:` → `<C-r>/`로 검색 패턴을 명령줄에 삽입 확인
9. `:earlier 2`로 2번의 변경 전 상태로 돌아가기 → `:later 2`로 복구

## 핵심 정리

- Ex 명령의 구조: **`:[범위]{명령} [인자]`**
- 범위 기호: **`.`** (현재 줄), **`$`** (마지막 줄), **`%`** (전체), **`'<,'>`** (Visual 선택)
- **`:g/pattern/command`**: 패턴 매칭 줄에 일괄 명령 실행 -- 가장 강력한 Ex 명령
- **`:{range}!cmd`**: 선택 범위를 외부 명령으로 필터링 (정렬, 포매팅 등)
- **`:r !cmd`**: 외부 명령 출력을 버퍼에 삽입
- **`<C-r><C-w>`**: 명령줄에서 커서 아래 단어를 삽입 -- 치환에 필수
- **`q:`**: 명령 히스토리 창 -- 이전 명령을 편집하고 재실행
- **`:earlier`/`:later`**: 시간 기반 undo -- "n분 전 상태로" 되돌리기
- **`:t.`**: 현재 줄 복제 (레지스터를 건드리지 않는 `yyp`)
