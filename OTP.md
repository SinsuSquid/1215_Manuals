# Softmatter's Guidebook to OTP (One-Time Password)

- Written by SinsuSquid (bgkang) on 20 Feb 2024

지속되는 해킹 위협으로 인해 진절머리가 난 나머지 보안에 조금이라도 더 보탬이 되보고자 curie를 비롯한 모든 연구실 클러스터에 OTP (One-Time Password)를 도입하고자 합니다.
OTP를 간단하게 설명드리면, 등록된 key와 시간 정보를 이용해 난수를 발생시켜, 컴퓨터와 OTP 단말기 사이 passcode가 일치할 경우에만 로그인을 허락하는 방식입니다.
감사하게도 Google에서 개인 컴퓨터에 사용할 수 있는 OTP를 제공하고 있어 이를 활용하는 방법을 정리해보도록 하겠습니다.

편의상 사용자가 연구실의 컴퓨터 관리자(힘내세요!)에게 계정 생성을 요구한 후, 컴퓨터 관리자가 초기 비밀번호와 홈 디렉토리를 설정한 후 처음 로그인했다고 가정한 상태에서 설명을 시작하도록 하겠습니다.

새로운 클러스터를 구성한 다음 OTP를 설치하는 방법은 본문 가장 끝에 따로 정리하도록 하겠습니다.

## 개인별 OTP 설정하기 (User)

컴퓨터 관리자로부터 다음의 정보를 전달받았다고 가정합니다.

- 클러스터 ip : 000.111.222.333 (hostname : ador)
- username : NewJeans
- 초기 pw : haerinLove

1. ssh 이용해 클러스터 접속하기

```bash
# 단말기에서
$ ssh NewJeans@000.111.222.333
(NewJeans@000.111.222.333) Password: # haerinLove 입력

# 클러스터 접속 완료
NewJeans@ador:~$ 
```

2. 핸드폰에 google authenticator 설치하기

Google Play Store 혹은 App Store에서 "Google Authenticator" 어플리케이션을 다운로드받고, Google 계정으로 로그인합니다.

3. `google-authenticator` 실행

```bash
NewJeans@ador:~$ google-authenticator

Do you want authentication tokens to be time-based (y/n)
# y : 시간 기반 토큰 생성
```
y를 입력하면 화면에 다음과 같은 정보가 표시됩니다.

```bash
+---------------------------+
|                           |
|                           |
|                           |
|                           |
|          QR Code          |
|                           |
|                           |
|                           |
|                           |
+---------------------------+
Your new secret key is: [SECRET KEY]
Enter code from app (-1 to skip)
```

핸드폰 Google Authenticator App에서 "Add a code"를 선택하고 화면의 QR code를 스캔합니다.
스캔 후 6자리의 숫자가 표시되는데, 이 숫자를 '공백 없이' `Enter code from app (-1 to skip)` 이후에 입력합니다.

입력 후 몇가지 (y/n)을 설정하는 부분이 있는데 **반드시** 읽어본 후 결정합니다.
대부분 보안에 도움이 되는 방법이므로 y를 선택하길 추천합니다. (해킹좀 그만 당하고 싶어요... ㅠ)

4. 설정 후 로그인 하기

`exit`명령을 이용해 로그아웃 하고, 다시 한번 클러스터에 접속합니다.

```bash
# 단말기에서
$ ssh NewJeans@000.111.222.333
(NewJeans@000.111.222.333) Password: # haerinLove 입력
(NewJeans@000.111.222.333) Verification code: # App에 표시되는 6자리 숫자 공백없이 입력

# 클러스터 접속 완료
NewJeans@ador:~$ 
```

5. 주의사항

- User가 `google-authenticator`를 실행하면 $HOME에 '.google_authenticator'파일이 생성됩니다. 이 파일이 없으면 로그인에 매우 큰 애로 사항이 있으니 Home 폴더를 옮기는 등의 작업을 할 때 유의하시길 바랍니다.

- 컴퓨터 설정에 따라 상이하지만, rsa 인증 키(`~/.ssh/authorized_key` 에 등록)를 이용하는 경우 OTP를 무시하고 바로 로그인합니다. 편하다고 생각할수 있지만 보안에 좋은 일은 아니므로 숙고 후 사용하시길 바랍니다.

## 클러스터에 OTP 설치하기 (Administrator, Ubuntu 22.04.3 기준)

먼저 이 부분을 읽게 되신 연구실 컴퓨터 관리자분께 깊은 애도의 말씀 올립니다. 건강하세요!

1. google-authenticator 설치

- apt 및 dnf, yum 기본 repository에도 있습니다 (버전은 좀 낮은 편)

```bash
$ sudo apt update
$ sudo apt install libpam-google-authenticator -y
```

- apt랑 dnf랑 패키지 이름이 살짝 다르니 확인해주세요.

2. `/etc/pam.d/sshd` 파일 수정

```bash
$ sudo cp /etc/pam.d/sshd /etc/pam.d/sshd.backup
$ sudo vi /etc/pam.d/sshd
```
가장 마지막 줄에 `auth    required    pam_google_authenticator.so nullok`추가
(nullok : 사용자가 google-authenticator 설정하기 전에는 password만으로 로그인 가능 - 신규 사용자 생성시 반드시 OTP 사용하도록 안내할것!)

3. `/etc/ssh/sshd_config.d/google-authenticator.conf` 파일 수정

```bash
$ sudo touch /etc/ssh/sshd_config.d/google-authenticator.conf
$ sudo vi /etc/ssh/sshd_config.d/googl-authenticator.conf
```

아래 내용 입력합니다.

```conf
PasswordAuthentication no
ChallengeResponseAuthentication yes
UsePAM yes
PermitEmptyPasswords no
```
4. sshd 재시작
```bash
$ sudo systemctl restart sshd
```
