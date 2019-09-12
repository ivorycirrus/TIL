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


# .zshrc 파일 설정

로그인한 사용자의 home 폴더에 ```.zshrc``` 파일을 작성 해 두면 셸이 실행 될 때 초기화를 수행 할 수 있다.

MacOS 에서 사용하고 있는 zsh 환경설정 항목은 다음과 같다.

```bash
# plugins
 plugins=(
    git
    zsh-syntax-highlighting
    zsh-autosuggestions
 )

# zsh theme and configuration
ZSH_THEME="agnoster"
DISABLE_UNTRACKED_FILES_DIRTY="true"

# git log alias
alias gl="git log --graph --full-history --all --color --date=short --pretty=tformat:\"%x1b[31m%h%x08%x1b[0m%x20%ad %x1b[32m%d%x1b[0m%x2    0%s%x20%x1b[33m(%an)%x1b[0m\""

# Sublime Text
function sublime() {
   /Applications/Sublime\ Text.app/Contents/MacOS/Sublime\ Text "$@" &
}

{% raw  %}
# Simplify prompt
prompt_context() {
   if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
     prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
   fi
}
{% endraw %}
```

* **플러그인 설정** 
    * git : 프롬프트에서 Git 브랜치 및 커밋 상태를 표시하기 위해 사용
    * zsh-syntax-highlighting : 콘솔 입력 명령의 구문강조를 지원한다. 구문 오류가 있는 경우 빨간 글씨로도 표시해준다.
    * zsh-autosuggestions : 콘솔 입력시 자동완성 처리를 위함. 자주쓰는 명령의 경우 자동완성이 은근히 편함.

* **zsh 테마 설정**
    * ZSH_THEME : 셸 테마를 지정. 개인적으로 agnoster 테마를 선호함
    * DISABLE_UNTRACKED_FILES_DIRTY : 기본값은 false는 매 프로폼프트 표시시마다 해당 git 레파지토리의 dirty check를 수행함. 엔터 여러번 누를 경우 프롬프트 동작이 느려질 수 있다. git 명령 실행시 포롬프트가 갱신되긴 하므로, 셸 동작속도를 위해 dirty check를 끄고 사용하는 것을 추천한다.

* **git log alias**
    * 기본 ```git log``` 또는 ```git show``` 명령으로는 커밋 목록 및 병합관계등을 콘솔에서 보기 불편하다. 이걸 좀 더 예쁘게 표현하기 위한 ```git log``` 템플릿이다.

* **Sublime Text**
    * 주력으로 사용하는 에디터인 Sublime Text를 콘솔에서 실행하기 위한 함수. MacOS 네 설치위치를 확인 해서 지정해주면 편리하게 사용 가능하다.

* **Simplify prompt**
    * 프롬프트를 한줄로 간략하고 예쁘게 보이게 하기 위합 옵션

