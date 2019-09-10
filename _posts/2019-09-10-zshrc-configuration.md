---
layout: post
title: "zsh 환경설정 팁"
tags: [shell, zsh]
comments: true
---

MacOS의 기본 터미널과 기본 shell은 사용하기 조금 밋밋한 점이 없지 않다. iterm2 터미널에 oh-my-zsh셸을 이용하면 셸에 예쁘게 테마를 입힐 수 있고, 특히 agnoster 테마의 경우 현재 작업중인 브랜치와 파일 변경상태 및 커밋상태가 프롬프트에 눈의 잘띄게 보이므로 상당히 편리하다. 여기에 자주쓰는 git  명령과 sublime text 실행명령을 추가하면 좀더 편리하게 사용 할 수 있다.

# shell 설정

사용중인 shell 과 터미널은 다음과 같다.

* **Terminal** : iTerm2
* **shell** : zsh ( oh-my-zsh )
* **shell theme** : agmoster

# ```.zshrc```파일 설정

MacOS 에서 사용하고 있는 zsh 환경설정 항목은 다음과 같다.

```bash
# plugins
 plugins=(
    git
    zsh-syntax-highlighting
    zsh-autosuggestions
 )

# zsh theme
ZSH_THEME="agnoster"

# git configuration
DISABLE_UNTRACKED_FILES_DIRTY="true"
alias gl="git log --graph --full-history --all --color --date=short --pretty=tformat:\"%x1b[31m%h%x08%x1b[0m%x20%ad %x1b[32m%d%x1b[0m%x2    0%s%x20%x1b[33m(%an)%x1b[0m\""

# Sublime Text
function sublime() {
   /Applications/Sublime\ Text.app/Contents/MacOS/Sublime\ Text "$@" &
}

# Simplify prompt
prompt_context() {
   if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
     prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
   fi
}
```

