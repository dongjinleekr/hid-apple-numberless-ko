hid-apple-numberless-ko
=====

> hid-apple-numberless-ko: Linux용 Apple Magic Keyboard (wo/ Numeric Keypad) 한국어 키보드 드라이버 ([hid-apple-patched](https://github.com/free5lot/hid-apple-patched) 기반)

> hid-apple-numberless-ko: A Linux driver for Apple Magic Keyboard (wo/ Numeric Keypad) Korean Keyboard Support - A Fork of [hid-apple-patched](https://github.com/free5lot/hid-apple-patched).

[Apple Magic Keyboard (wo/ Numeric Keypad)](https://www.apple.com/kr/shop/product/MQ5L2KH/A/magic-keyboard-%ED%95%9C%EA%B5%AD%EC%96%B4)를 Linux 상에서 사용하기 위한 드라이버입니다.

Linux에서 한국어를 입력하기 위해서는 오른쪽 Alt, Ctrl 키를 각각 Hangul, Hanja 키로 사용할 수 있어야 합니다. 하지만 [Apple Magic Keyboard (wo/ Numeric Keypad)](https://www.apple.com/kr/shop/product/MQ5L2KH/A/magic-keyboard-%ED%95%9C%EA%B5%AD%EC%96%B4)는 Space Bar와 화살키 사이에 Command 키와 Option 키밖에 없죠. 게다가 F1 ~ F12 키가 media key 기능을 우선하도록 되어 있기 때문에 function key 가 디버깅 툴 단축키로 할당되어 있는 경우가 많은 개발자 입장에서는 사용하기가 좀 불편합니다 - 일일이 FN 키와 함께 눌러야 하니까요. 심지어 쓰다 보면 Backspace 가 아닌 Delete 키가 필요한 경우도 있는데 Delete는 없고 대신 Eject 키가 장착되어 있기도 하죠.

이 드라이버는 위에 열거한 단점들을 해결하기 위해 제작되었습니다. ~~결코 제작자가 곧 죽어도 애플 키보드를 써야겠다는 중증 앱등이여서 만든 것이 아닙니다~~ 이 driver는 apple keyboard에 대해 다음과 같은 기능들을 추가합니다:

- F1 ~ F12 키의 동작을 function key 우선으로 변경합니다.
- Eject 키가 Delete 키로 동작하도록 변경합니다.
- 오른쪽 Command 키가 Ctrl 키로 동작하도록 변경합니다.

+1. Linux System에서 오른쪽 Alt, Ctrl 키를 Hangul, Hanja 키로 인식시키는 방법에 대해서는 아래 '부록 1'을 참조해 주세요.

# Requirements

이 드라이버를 사용하기 위한 최소 사양입니다.

- [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support)가 지원되는 Linux 배포판

  2020년 현재 거의 대부분의 리눅스 배포판이 여기 해당됩니다.

- git 클라이언트 (Optional)

  아래 '사용법'은 git으로 코드를 clone 받는 것을 기준으로 설명하고 있기 때문에 git 역시 필요합니다. 소스코드를 직접 다운로드 하실 경우에는 해당 사항 없겠지만요.

Ubuntu 등 Debian 기반 System에서는 아래 command로 필요한 프로그램들을 한 번에 설치할 수 있습니다:

```sh
sudo apt install git dkms
```

# 작업 환경

이 드라이버를 개발하는 데 사용된 + 기준으로 삼은 시스템들입니다.

- Ubuntu Linux 20.04
- [Apple Magic Keyboard (MQ5L2KH)](https://www.apple.com/kr/shop/product/MQ5L2KH/A/magic-keyboard-%ED%95%9C%EA%B5%AD%EC%96%B4)

# 설치법

## 1. 소스코드 clone

```sh
git clone https://github.com/dongjinleekr/hid-apple-numberless-ko
cd hid-apple-numberless-ko
```

## 2. dkms에 디렉토리 등록 및 빌드

```sh
sudo dkms add .
sudo dkms build hid-apple/1.0
sudo dkms install hid-apple/1.0
```

+1. 빌드된 내용물을 삭제할 때는 아래와 같이 해 주면 됩니다. (소스코드를 수정한 뒤 다시 빌드할 때 사용합니다.)

```sh
sudo dkms uninstall hid-apple/1.0
sudo dkms remove hid-apple/1.0
```

## 3. 설정 파일 생성

`/etc/modprobe.d/hid_apple.conf` 파일을 생성하고 아래와 같은 내용을 넣어 줍니다:

```
options hid_apple fnmode=2
options hid_apple rightcmd_as_rightctrl=1
options hid_apple ejectcd_as_delete=1
```

각각의 항목은 아래와 같은 의미를 가집니다:

- fnmode: F1 ~ F12 키가 동작하는 방법을 결정합니다.

  - 0: FN 키를 사용할 수 없도록 막습니다. F1 ~ F12 키는 media key로만 동작합니다. (예: F10은 음량 감소, F11은 음량 증가.)
  - 1: F1 ~ F12 키를 기본적으로 media key로 동작하도록 하고, FN 키와 함께 눌렀을 때만 function key로 동작하도록 합니다. (default)
  - 2: F1 ~ F12 키를 기본적으로 function key로 동작하도록 하고, FN 키와 함께 눌렀을 때만 media key로 동작하도록 합니다.

- rightcmd_as_rightctrl: 오른쪽 Command 키가 Ctrl 키로 동작하도록 합니다.

  - 0: 해당 기능을 끕니다. 오른쪽 Command 키는 원래대로 Command 키로 동작합니다. (default)
  - 1: 해당 기능을 켭니다. 오른쪽 Command 키는 Ctrl 키로 동작합니다.

- ejectcd_as_delete: Eject 키가 Delete 키로 동작하도록 합니다.

  - 0: 해당 기능을 끕니다. Eject 키는 원래대로 Eject 키로 동작합니다. (default)
  - 1: 해당 기능을 켭니다. Eject 키는 Delete 키로 동작합니다.

즉, 위 설정 파일 내용은 다음과 같은 기능을 합니다:

- fnmode를 2로 설정한다. 즉, F1~F12는 기본적으로 function key로 동작하며 FN 키와 함께 눌렀을 때만 media key로 동작한다.
- 오른쪽 Command 키를 Ctrl 키로 동작하도록 한다.
- Eject 키를 Delete 키로 동작하도록 한다.

## 4. 모듈 적재 + 부트 이미지 업데이트

다음 command로 기본 드라이버 모듈 (hid_apple) 을 메모리 상에서 삭제하고 대신 새로 빌드된 드라이버 모듈을 적재합니다:

```sh
sudo modprobe -r hid_apple && sudo modprobe hid_apple
```

이제부터 드라이버 기능을 사용할 수 있지만, 리부팅하면 변경 사항이 사라집니다. 아래 command로 부팅할 때마다 자동으로 적재되게 할 수 있습니다:

```sh
sudo update-initramfs -u
```

# 라이센스

이 프로젝트는 원본인 [hid-apple-patched](https://github.com/free5lot/hid-apple-patched)와 마찬가지로 [GNU 일반 공중 사용 허가서 2.0](LICENSE) 라이센스를 따릅니다.

# 부록

## 부록 1: Ubuntu 20.04에 한국어 키보드 설정하기

Ubuntu 상에서 한글 키보드를 인식되도록 하는 방법입니다. 구체적으로는, 아래 두 기능을 활성화합니다:

- 오른쪽 Alt 키를 한글키로 매핑합니다.
- 오른쪽 Ctrl 키를 한자키로 매핑합니다.

여기서 한 설정은 laptop에 붙어 있는 키보드건 유/무선으로 연결한 키보드건 상관 없이 전부 다 적용됩니다.

### 1. 오른쪽 Alt 키가 Hangul 키로 매핑되도록 키보드 설정 변경

`/usr/share/X11/xkb/symbols/altwin` 의 `xkb_symbols "meta_alt" { ... }` 부분을 아래와 같이 수정합니다:

```
// Meta is mapped to second level of Alt.
partial modifier_keys
xkb_symbols "meta_alt" {
    key <LALT> { [ Alt_L, Meta_L ] };
    key <RALT> { type[Group1] = "TWO_LEVEL",
                 symbols[Group1] = [ Hangul ] };
    modifier_map Mod1 { Alt_L, Alt_R, Meta_L, Meta_R };
//  modifier_map Mod4 {};
};
```

### 2. 오른쪽 Ctrl 키가 Hanja 키로 매핑되도록 키보드 설정 추가

`/usr/share/X11/xkb/symbols/altwin` 에 아래 부분을 추가합니다:

```
// Ctrl is mapped to Hangul_Hanja.
partial modifier_keys
xkb_symbols "ctrl_hangul_hanja" {
    key <LCTL> { [ Control_L, Control_L  ] };
    key <RCTL> { type[Group1] = "TWO_LEVEL",
                 symbols[Group1] = [ Hangul_Hanja ] };
    modifier_map Control { Control_L, Control_R };
};
```

### 3. 키보드 입력 설정에 2 에서 생성한 설정을 추가

`/usr/share/X11/xkb/symbols/pc` 의 `include "altwin(meta_alt)"` 아래에 다음과 같이 `include "altwin(ctrl_hangul_hanja)"`를 추가합니다:

```
    key <ALT>  {    [ NoSymbol, Alt_L   ]   };
    include "altwin(meta_alt)"
    include "altwin(ctrl_hangul_hanja)"

    key <META> {    [ NoSymbol, Meta_L  ]   };
    modifier_map Mod1   { <META> };
```

### 4. 기존 설정 mapping cache 삭제

```sh
sudo rm /var/lib/xkb/*
```

이제 리부팅하면 수정된 설정이 반영됩니다.

+1. 테스트는 해보지 않았으나 Ubuntu가 기반하고 있는 Debian 기반 System 모두에서 작동할 것으로 생각합니다.

+2. `${HOME}/.xinputrc`을 사용하는 방법도 있다고 알고 있습니다만, 왠일인지 제 경우는 그 방법이 통하지 않더군요. 위 방법을 사용하면 단일 사용자 뿐만 아니라 한국어 키보드를 사용하는 사용자 전체에 오른쪽 Alt 키 = Hangul 키, 오른쪽 Crtl 키 = Hanja 키로 강제할 수 있다는 차이점이 있습니다.

## 부록 2: Fcitx 한국어 설정하기

Ubuntu 상에 다국어 입력기 모듈인 fcitx를 설치하는 방법입니다.

(작성중)

## 부록 3: Ctrl, Eject 키를 다른 용도로 사용하기

[Numeric Keypad가 붙어 있는 모델](https://www.apple.com/kr/shop/product/MRMH2KH/A/magic-keyboard-with-numeric-keypad-%ED%95%9C%EA%B5%AD%EC%96%B4-%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4-%EA%B7%B8%EB%A0%88%EC%9D%B4)을 사용 중이신 분이라면 오른쪽 Command, Eject 키를 다른 키로 mapping하고 싶으실 수 있습니다. 이럴 때는 아래와 같이 해 주시면 됩니다.

예를 들어서, Eject 키를 Delete 키가 아닌 Scroll Lock 키로 동작하도록 바꿔 보겠습니다. `hid-apple.c`에서 아래와 같은 부분을 찾습니다.

```c
static const struct apple_key_translation ejectcd_as_delete_keys[] = {
	{ KEY_EJECTCD,	KEY_DELETE },
	{ }
};
```

설정 파일에 `ejectcd_as_delete=1`가 설정된 상태에서 위 `ejectcd_as_delete_keys` 함수를 아래와 같이 변경한 뒤 드라이버를 다시 빌드 + 설치해 주면 Eject 키는 이제부터 Scroll Lock 키로 동작합니다.

```c
static const struct apple_key_translation ejectcd_as_delete_keys[] = {
	{ KEY_EJECTCD,	KEY_SCROLLLOCK },  // KEY_DELETE → KEY_SCROLLLOCK
	{ }
};
```

마찬가지로 오른쪽 Command 키를 다른 키로 사용하고 싶을 경우 `rightcmd_as_rightctrl_keys` 함수를 같은 방식으로 수정해 주면 됩니다.

Linux 커널에서 사용하는 Keycode 값은 [여기](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h)을 참조하세요.

