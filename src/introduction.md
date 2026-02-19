# 서문

## 이 책은 누구를 위한 것인가

이 책은 Neovim을 "가끔 사용하지만 능숙하지는 않은" 개발자를 위해 쓰였다. `i`를 눌러 입력하고, `:wq`로 저장하고 나오는 정도는 할 수 있지만, 동료가 Vim으로 코드를 마법처럼 편집하는 모습을 보며 "어떻게 저렇게 하는 거지?"라고 생각해 본 적이 있다면, 바로 당신을 위한 책이다.

이 책을 다 읽고 나면 당신은:

- Vim의 편집 언어를 **생각하듯이** 사용할 수 있다
- 복잡한 편집 작업을 **몇 번의 키 입력**으로 해결할 수 있다
- Neovim을 **본격적인 개발 환경**으로 구축할 수 있다
- 나만의 설정과 플러그인으로 **최적화된 워크플로우**를 만들 수 있다

## 학습 로드맵

이 책은 5개의 파트로 구성되어 있으며, 순서대로 읽는 것을 권장한다.

```
Part 1: 기본기 다지기          ← Vim의 "언어"를 배운다
  ↓
Part 2: 중급 기능              ← 생산성을 극대화하는 기능들
  ↓
Part 3: 설정과 커스터마이징     ← 나만의 Neovim을 만든다
  ↓
Part 4: 핵심 플러그인 생태계    ← 현대적 개발 환경을 구축한다
  ↓
Part 5: 마스터리               ← 전문가로 도약한다
```

**Part 1**은 Vim을 "언어"로 이해하는 것에 집중한다. 모션(motion), 오퍼레이터(operator), 텍스트 오브젝트(text object)가 어떻게 조합되는지 배우면, 수백 개의 명령어를 외우지 않아도 자연스럽게 편집할 수 있다.

**Part 2**에서는 레지스터, 매크로, 검색/치환 등 한 단계 더 깊은 기능을 다룬다. 이 기능들을 익히면 반복 작업에서 해방된다.

**Part 3**은 Neovim을 자신의 취향에 맞게 설정하는 방법을 다룬다. `init.lua`를 작성하고, 키맵을 설정하고, Lua 스크립팅의 기초를 배운다.

**Part 4**는 현대적인 Neovim 생태계의 핵심 플러그인들을 다룬다. LSP, Treesitter, Telescope 등을 설정하면 VSCode에 견줄 만한 개발 환경을 갖출 수 있다.

**Part 5**에서는 실전 워크플로우, 트러블슈팅, 그리고 직접 플러그인을 만드는 방법까지 다룬다.

## 실습 환경 준비

### Neovim 설치

이 책은 **Neovim 0.10 이상**을 기준으로 작성되었다.

**macOS:**
```bash
brew install neovim
```

**Ubuntu/Debian:**
```bash
# 최신 버전을 위해 PPA 또는 AppImage 사용 권장
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.appimage
chmod u+x nvim-linux-x86_64.appimage
sudo mv nvim-linux-x86_64.appimage /usr/local/bin/nvim
```

**Windows:**
```powershell
winget install Neovim.Neovim
```

설치 후 버전을 확인한다:
```bash
nvim --version
```

### 터미널 설정

Neovim을 쾌적하게 사용하려면 좋은 터미널과 폰트가 필요하다:

- **터미널**: iTerm2(macOS), Windows Terminal(Windows), Alacritty/Kitty(크로스 플랫폼)
- **폰트**: Nerd Font 계열 권장 (아이콘 표시를 위해)
  ```bash
  # macOS
  brew install --cask font-jetbrains-mono-nerd-font
  ```

### 깨끗한 상태에서 시작하기

기존 설정이 있다면 백업 후 깨끗한 상태에서 시작하는 것을 권장한다:

```bash
# 기존 설정 백업
mv ~/.config/nvim ~/.config/nvim.bak
mv ~/.local/share/nvim ~/.local/share/nvim.bak
mv ~/.local/state/nvim ~/.local/state/nvim.bak
mv ~/.cache/nvim ~/.cache/nvim.bak
```

### 이 책에서 사용하는 표기법

| 표기 | 의미 |
|------|------|
| `<CR>` | Enter 키 |
| `<Esc>` | Escape 키 |
| `<C-x>` | Ctrl + x |
| `<leader>` | Leader 키 (기본값: `\`, 이 책에서는 Space로 설정) |
| `<S-x>` | Shift + x |
| `{motion}` | 임의의 모션 명령 |

> **참고**: 이 책의 모든 예제는 직접 따라 할 수 있도록 구성되어 있다. 읽기만 하지 말고 반드시 Neovim을 열어 직접 실습하길 바란다. 근육 기억(muscle memory)이 Vim 숙달의 핵심이다.
