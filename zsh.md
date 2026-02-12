
# Zsh 세팅 가이드

## 1. Oh My Zsh 설치

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 2. Powerlevel10k 테마 설치

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

`~/.zshrc`에서 테마 변경:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Nerd Font 설치 (아이콘 표시용):

```bash
brew install --cask font-meslo-lg-nerd-font
```

터미널 앱 설정에서 폰트를 `MesloLGS NF`로 변경 후 `p10k configure`로 스타일 설정.

## 3. zsh-autosuggestions (자동완성 제안)

이전 명령어를 회색으로 자동 제안, `→` 방향키로 채택.

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

## 4. zsh-syntax-highlighting (명령어 색상 구분)

명령어가 유효하면 초록, 틀리면 빨간색으로 표시.

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## 5. 플러그인 등록

`~/.zshrc`에서 `plugins=` 줄 수정 (쉼표 없이 공백으로 구분):

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

## 6. 적용

```bash
source ~/.zshrc
```