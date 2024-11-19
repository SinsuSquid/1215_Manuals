# How to upgrade your CentOS 7 Kernel
written by SinsuSquid (bgkang) on 07 May 2024

어김없이 계속되는 해킹의 위협에 이번엔 ㄹㅇ로 진절머리가 난 나머지 귀찮음을 감수하고 모든 CentOS 7 서버의 리눅스 커널을 업데이트 하기로 했습니다.

지금 CentOS 7의 경우 리눅스 커널 버전 3.x 정도를 사용하고 있습니다.
비유를 들어보자면 여러분이 쓰시는 노트북에 보안수준은 윈도우 xp 라고 생각해주시면 됩니다.
더 이상 CentOS 7 의 커널 업데이트를 공식 채널에서는 지원하지 않기 때문에, 다른 경로를 통해 커널을 업데이트하게 됩니다.

[여기](https://cjwoov.tistory.com/48) 를 참고하여 작성하였습니다.
- Installing elrepo
```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org # public key download
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```

- Installing New Kernel
```bash
yum --enablerepo=elrepo-kernel install kernel-ml kernel-ml-devel
```

- Finding Bootable Kernel
 아래 명령어를 통해 설치된 커널의 버전을 확인하고, 최신 버전을 확인해주세요.
```bash
grep ^menuentry /boot/grub2/grub.cfg | cut -d "'" -f2
```

현 시점 기준 다름과 같이 출력됩니다.
```bash
CentOS Linux (6.8.7-1.el7.elrepo.x86_64) 7 (Core)
```

- Set Booting Option
```bash
grub2-set-default "CentOS Linux (6.8.7-1.el7.elrepo.x86_64) 7"
```
부팅 순서 변경 확인
```bash
grub2-editenv list
```
잘 됐다면
```bash
reboot
```
재부팅이 완료되었다면
```bash
uname -msr
```
을 통해 제대로 부팅되고 있는지 확인합니다.

# On einstein

왜인지는 모르겠지만 `reboot` 명령어를 적용한 이후에 node가 자동으로 켜지지 않는걸 확인했습니다.
이런 문제는 해결하기보다 그냥 손으로 다시 켜주는게 빨라요 ㅎ

`163.239.206.189` : einstein CMM에 접속해서 해당 node를 on/off합니다.

