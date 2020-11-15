---
title: 우분투 18.04 LTS에서 윈도우10 부팅 USB 만들기(가상머신 사용)
author: mirukman
date: 2020-11-15 17:00:00 +0800
categories: [IT]
tags: [우분투,리눅스,윈도우,부팅,usb,ubuntu,linux,windows,usb]
---

우분투의 WoeUSB라는 툴을 사용하다가 원인모를 버그 때문에 실패했다.

이후 가상머신을 설치하고 Windows 가상머신을 만들어 부팅 USB를 만들려 했으나 USB가 인식되지 않는 문제가 있어 꽤나 삽질을 하였다.

<br>

### 1. Windows10 ISO 파일 다운로드 ###
---

VirtualBox에서 가상머신에 Windows 10 설치를 위해서 Windows 10 ISO 파일을 다운로드한다.

아래 링크에서 Windows 10 ISO 이미지 파일 다운로드

[https://www.microsoft.com/ko-kr/software-download/windows10ISO](https://www.microsoft.com/ko-kr/software-download/windows10ISO)

<br>

### 2. virtualbox 다운로드 ###
---

[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) 링크에서 수동으로 다운받아도 된다.

아래 스크립트는 우분투 리눅스 18.04 LTS 버전 기준으로 확실히 동작한다. 혹시 버전이나 다른 문제로 VirtualBox 6.0이 설치가 되지 않거나 혹은 찜찜하면 수동으로 받는걸 추천한다.

~~~ bash
$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add

$ echo "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian bionic contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list

$ sudo apt-get update

$ sudo apt-get -y install virtualbox-6.0

$ systemctl status vboxdrv

$ virtualbox
~~~

<br>

### 3. USB 컨트롤러 인식을 위한 확장팩 설치 ###
---

VirtualBox 관리자 창에서

**\[도움말\] - \[VirtualBox 정보\]** 화면에서 VirtualBox 버전을 확인한다.

이후 [https://www.virtualbox.org/wiki/Download_Old_Builds](https://www.virtualbox.org/wiki/Download_Old_Builds) 에서 자신의 VirtualBox 버전에 맞는 링크를 클릭한다.

예를 들어 6.0 버전 링크를 클릭하면 아래 사진과 같이 세부적인 버전으로 또 나뉜다.

![virtualbox 다운로드 페이지](https://mirukman.github.io/images/it/make-windows10-booting-usb-on-ubuntu/virtualbox-old-versions.png){: width="100%" height="100%"}

여기서 자신의 VirtualBox 버전에 맞는 버전의 Extension Pack을 클릭한다.

그럼 자동으로 다운로드가 받아지면서 virtualbox 관리자 화면이 뜨고 추가할 것인지 묻는 메시지창이 나타난다. 확인을 누르고 설치를 하면 된다.

만약 자동으로 확장팩 추가 안내창이 뜨지 않으면 VirtualBox 관리자 창에서 **\[파일\] - \[환경설정\] - \[확장팩(Extension Pack)\]** 메뉴에 들어가 수동으로 추가해주면 된다.

<br>

### 4. Windows 10 가상머신 생성 ###
---

Windows10 가상머신을 생성한다.

그냥 기본 세팅으로 계속 확인을 누르면 생성된다.

이후 생성된 머신을 시작시키면 과정1에서 다운받은 Windows10 ISO 이미지를 선택해서 Windows 10 운영체제를 가상머신에 설치할 수 있다.

일단 가상머신에 Windows10을 설치했으면 가상머신을 종료한다.

<br>

### 5. vboxusers group에 사용자 추가 ###
---

~~~ bash
$ sudo usermod -aG vboxusers <username>
~~~

\<username\>에는 ubuntu 사용자 계정이름을 넣어주면 된다.

<br>

### 6. 재부팅 ###
---

난 위 과정까지는 문제없이 했는데, 잠시 후 설명할 7 과정에서 실패했었다. 도저히 USB 컨트롤러 설정 화면에서 설치가능한 컨트롤러 목록이 나타나지 않아서 고생했다.

그런데 vboxusers group에 사용자 추가를 하고나서 재부팅을 해주니 된다.

재부팅을 해주어야 시스템이 재시작되면서 vboxusers group에 새로 추가된 사용자가 인식 되는듯 하다. 사실 재부팅까지는 하지않아도 virtualbox나 vboxusers 사용자 인식과 관련된 프로세스를 재시작 해주면 될 듯 하나 재부팅이 가장 깔끔한듯 싶다.

<br>

### 7. USB 컨트롤러 추가 ###
---

설치한 가상머신에 마우스 우측클릭 후 설정에 진입한다. 그러면 USB 탭이 있는데 거기에서 자신의 USB 타입(1.0/2.0/3.0)에 맞는 컨트롤러를 선택 후 새로운 USB 컨트롤러 추가 버튼을 누르면 자신의 컴퓨터에 연결된 USB 컨트롤러가 나타난다. 이를 선택하여 등록해준다.

![https://mirukman.github.io/images/it/add-usb-controller-1.png](https://mirukman.github.io/images/it/add-usb-controller-1.png){: width="60%" height="60%"}

![https://mirukman.github.io/images/it/add-usb-controller-2.png](https://mirukman.github.io/images/it/add-usb-controller-2.png){: width="60%" height="60%"}

<br>

### 8. Windows10 부팅 후 부팅 USB 만들기 ###
---

이후 Windows10을 부팅하면 저장소 목록에 USB가 나타날 것이다.
이후 부팅 USB를 만드는 방법은 Windows OS에서 만드는 방법과 동일하다.
