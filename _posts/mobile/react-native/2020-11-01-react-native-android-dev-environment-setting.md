---
title: 리눅스 우분투에서 리액트 네이티브 안드로이드 개발환경 세팅
author: mirukman
date: 2020-11-01 17:00:00 +0800
categories: [mobile, react-native]
tags: [react-native,리액트,네이티브,우분투,안드로이드,개발환경,세팅,설정]
---

처음에 expo를 사용해서 리액트를 개발하다보니 여러가지 단점이 눈에 보였다.

1. 초기 생성한 프로젝트의 크기부터가 너무 크다.(~30MB)
2. 빌드타임이 너무 오래걸린다.
3. 코드 좀 수정하고 나면 불러올 수 없어 앱을 재시작 해야한다거나 문법 에러가 없는데도 최근 코드를 불러오지 못해 에러를 발생시키는 문제가 너무 빈번해서 짜증난다.
4. expo에서 아직도 지원하지 않는 네이티브 api들이 많이 존재하며 자바나 코틀린을 사용해 네이티브 코드를 작성할 수 없다. 이것은 아직 겪어보지 않은 문제이지만 추후 커스터마이징의 자유로움을 제한할 수도 있다는 생각에 찜찜함을 지울 수 없다.

어차피 react-native-cli로 넘어가야 할 것이므로 지금부터 시작해보려 했는데 초기 세팅 문제가 만만치 않아 이를 정리한다.

<br>

## 1. nvm, node 설치 ##
---

+ nvm 설치

0.36.0버전 설치

~~~ bash
sudo curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.36.0/install.sh | bash
~~~

<br>
이제 ~/.bashrc 파일에 nvm 변수가 추가되었을 것이다. source 명령어로 bash shell에서 nvm 변수를 업데이트 시켜주어야한다.
~~~ bash
source ~/.bashrc
~~~

버전을 확인해보면 

~~~ bash
nvm --version
> 0.36.0
~~~

nvm이 잘 설치되었다.

<br>

+ nvm으로 node 설치

아래 명령어를 입력하면

~~~ bash
nvm ls
~~~

설치되어있거나 설치할 수 있는 node 버전들이 출력된다.

최신 안정화버전을 설치하기 위해 아래 명령을 입력한다.

~~~ bash
nvm install --lts
~~~

혹은 특정 버전을 설치하고 싶다면 아래와 같이 입력하면 된다.

~~~ bash
nvm install <버전>
~~~

이후 설치가 잘 되었는지 확인

~~~ bash
node --version
> 12.19.0
~~~

~~~ bash
npm --version
> 6.14.8
~~~

여러 버전의 노드가 설치되어있을 때 특정 노드를 사용하는 방법

~~~ bash
nvm use <버전>
~~~

<br>

## 2. react-native-cli 설치 ##
---

~~~ bash
$ npm install -g react-native-cli
~~~

-g 옵션은 전역 설치를 의미한다. npm으로 설치한 모듈은 특정 프로젝트에 국한되는데 이를 전역(global) 범위로 설치해주기 위함이다.

<br>

이제 특정 폴더 혹은 워크스페이스로 이동해서 아래 명령어로 react-native 앱을 생성할 수 있다.

~~~ bash
react-native init SampleProject
~~~

<br>

## 3. 안드로이드 스튜디오 설치 ##
---

JDK는 설치되었다고 가정한다. 설치되지 않았다면 jdk를 설치하고 /etc/bash.bashrc에 JAVA_HOME 환경변수를 추가해주면 된다.

+ 안드로이드 설치

리눅스 버전 안드로이드 스튜디오 다운로드 링크

[https://developer.android.com/studio?hl=ko](https://developer.android.com/studio?hl=ko)

<br>

+ ANDROID_HOME 환경변수 설정

~~~ bash
sudo vim /etc/
~~~

아래 코드를 마지막 줄에 붙여넣는다.

~~~ bash
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
~~~

<br>

+ 안드로이드 에뮬레이터 실행을 위한 SDK 설치

여기부터가 문제이다. 에뮬레이터가 실행되지 않는 문제로 시간을 좀 잡아먹었는데 아래가 그 해결 방법이다.

기본적으로는 리액트 네이티브를 실행하기 위한 안드로이드 버전(현재 기준 안드로이드10-Q SDK가 설치되어있어야한다.)

<br>


아래 문서에 매우 친절하게 안드로이드 에뮬레이팅을 위한 설정 방법이 나와있다.

[https://reactnative.dev/docs/environment-setup](https://reactnative.dev/docs/environment-setup)


보기 힘들다면 간단하게 아래에 정리된 내용을 따라하면된다.

먼저 안드로이드 스튜디오를 실행한다.

+ File → Settings → Appearance & Behavior → System Settings → Android SDK
	+ SDK Platforms 탭
		+ 오른쪽 하단 Show Package Details 체크박스를 체크
		+ Android 10 (Q) 항목에서 아래 항목들 체크
			+ Android SDK Platform 29
			+ Intel x86 Atom_64 System Image or Google APIs Intel x86 Atom System Image
	+ SDK Tools 탭
		+ 오른쪽 하단 Show Package Details 체크박스를 체크
		+ Android SDK Build Tools 항목에서 아래 항목 체크
			+ 29.0.2
	+ 최종적으로 apply 눌러 적용

<br>

+ 리액트 네이티브 앱 실행을 위한 새로운 에뮬레이터 생성
	+ Tools - AVD Manager
	+ Create Virtual Device(이미 다른 에뮬레이터가 있다면 삭제)
	+ Select Hardware는 디폴트로 두고 Next
	+ System Image 에서는 Android 10 (Q) 선택 후 Next
	+ AVD Name 마음대로 작성 후 Finish

리눅스에서 에뮬레이터 생성시 일어날 수 있는 문제가 있다. 'kvm permission denied'와 같은 경고 메시지가 뜬다면 사용자 등록이 되지 않아서 생기는 것이다. /dev/kvm에 사용자를 추가해주면 되는데, 명령어들을 차례대로 입력한다.

~~~ bash
sudo apt install qemu-kvm
sudo adduser $USER kvm
sudo chown $USER /dev/kvm
~~~

<br>

## 4. Watchman 설치 ##
---

watchman은 페이스북에서 만든 툴인데 코드 수정 시 곧바로 화면에 변경사항을 적용시켜준다.

아래 링크는 페이스북 watchman 깃허브 저장소이다.

[https://github.com/facebook/watchman/releases/](https://github.com/facebook/watchman/releases/)

최신 버전을 받으면 된다.(리눅스라면 watchman-v0000.00.00.00-linux.zip 파일)

<br>
이후 다운받은 압축파일이 있는 경로(~/Downloads)로 이동하여 아래 명령어 차례로 입력

~~~ bash
unzip watchman-\*-linux.zip
sudo mkdir -p /usr/local/{bin,lib} /usr/local/var/run/watchman
sudo cp bin/\* /usr/local/bin
sudo cp lib/\* /usr/local/lib
sudo chmod 755 /usr/local/bin/watchman
sudo chmod 2777 /usr/local/var/run/watchman
~~~

이제 react-native 앱에서는 watchman을 사용할 수 있는 상태가 되었다.

## 5. 리액트 네이티브 앱 실행 ##
---

아까 생성해둔 'SampleProject' 디렉토리로 이동

~~~ bash
react-native start
~~~

~~~ bash
react-native run-android
~~~

에뮬레이터가 실행되면서 화면이 나타난다.
