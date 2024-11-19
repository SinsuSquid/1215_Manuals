# R1215's guide to the NEW Stokes

# (Summer 2023)

written by bgkang (SinsuSquid)
on the day of 08 AUG 2023

## Stokes Hardware Info.

- disk의 mount 위치는 변할 수 있습니다.
- [리눅스에서 fstab을 UUID로 변경](https://faq.hostway.co.kr/Linux_ETC/3033)
- | NAME         | CPU                                            | GPU                                                    | STORAGE                                                                                                                     | OS               | Etc.                              |
  |:------------ |:---------------------------------------------- |:------------------------------------------------------ |:--------------------------------------------------------------------------------------------------------------------------- |:---------------- |:--------------------------------- |
  | stokes1      | Intel(R) Xeon(R) E5-2620 v3 @ 2.4GHz (12 core) | Geforce GTX1080 * 2                                    | SSD: <br/>Samsung SSD 850 512 GB (/dev/sda)<br/>Samsung SSD 860 4TB (/dev/sdc)<br/>HDD:<br/>WDC WD60EFRX-68L 6TB (/dev/sdb) | Ubuntu 22.04 LTS |                                   |
  | stokes2, 4-7 | Intel(R) Core(TM) i7-5820K @ 3.30GHz (12 core) | GeForce RTX 2080 SUPER * 2                             | SSD:<br/>Samsung SSD 850 120GB (/dev/sda)                                                                                   | Ubuntu 22.04 LTS | 쿨링팬 issue, CPU 온도 확인 필요 (Stokes2) |
  | stokes3      | Intel(R) Core(TM) i7-5820K @ 3.30GHz (12 core) | Geforce RTX 2080 SUPER * 1<br/>Geforce RTX 2080 Ti * 1 | SSD:<br/>Samsung SSD 850 120GB (/dev/sda)                                                                                   | Ubuntu 22.04 LTS |                                   |
  | stokes8      | Intel(R) Core(TM) i7-5820K @ 3.30GHz (12 core) | GeForce RTX 2080 SUPER * 2                             | SSD: <br/>Samsung SSD 850 120GB (/dev/sda)<br/>Samsung SSD 860 4TB (/dev/sdb)                                               | Ubuntu 22.04 LTS |                                   |
  | stokes9      | Intel(R) Core(TM) i7-5820K @ 3.30GHz (12 core) | Geforce GTX 1080 * 2                                   | SSD: <br/>Samsung SSD 850 120GB (/dev/sda)                                                                                  | Ubuntu 22.04 LTS |                                   |

## Installation Guide

1. Create a bootable USB
   [Create a bootable USB stick on macOS](https://ubuntu.com/tutorials/create-a-usb-stick-on-macos#1-overview)
   
   * 생각보다 사용하는 tool을 좀 탑니다.
   * Canonical에서 권장하는 balenaEtcher를 사용했습니다.

2. Install Ubuntu Server
   [Basic Installation](https://ubuntu.com/server/docs/installation)
   
   > Troubleshooting : 설치 후 boot시 boot disk를 찾지 못하는 문제 발생
   > 
   > 설치 시 UEFI 모드로 접속하여 설치하였는지 다시 한번 확인합시다.
   > 
   > 왜인지는 모르겠지만 USB 하나당 2-3대 컴퓨터를 설치하면 더 이상 UEFI모드가 인식이 안되는 문제가 발생했습니다.
   > 원인을 찾기 보단 그냥 bootable USB를 여러번 만드는 방법으로 해결했습니다 😄

## Setup Guide

### 고정 IP 설정 - For all stokes's

- Overview
  
  ```
          Internet
  -------------+-------------
        Gateway|163.239.206.1/24 (학교)
               |
  External     |
         enp5s0|163.239.206.187/24
  +------------+------------+
  |                         |
  |         stokes1         |
  |                         |
  +------------+------------+
           eno1|192.168.0.100/24
  Internal     |
               |
  +------------+------------+
  |                         |
  |        stokes2-9        |
  |                         |
  +------------+------------+
           eno1|192.168.101-108/24
  ```

- `/etc/netplan/00-install-config.yaml` (파일 명 다를 수 있음) 파일 수정

- stokes1
  
  ```
  network:
        ethernets:
            en5s0: # 내부 컴퓨터와 소통하는 포트
                dhcp4: no
                addresses:
                    - 192.168.0.100/24
            enol1: # 외부 인터넷 소통하는 포트
                dhcp4: no
                addresses
                    - 163.239.206.187/24
                gateway4: 163.239.206.1
                nameservers:
                    addresses: [163.239.1.1, 8.8.8.8]
        version 2
  ```

- stokes2-9
  
  ```
  network:
        ethernets:
            enol1: # stokes1과 소통하는 포트
                dhcp4: no
                addresses:
                    - 192.168.0.101/24
                nameservers:
                    addresses: [163.239.1.1, 8.8.8.8]
        version 2
  ```
  
  - 변경 사항 저장
  
  ```bash
  netplan apply
  ```
  
  ### UFW 방화벽 설정 - For all stokes's

- [UFW : Basic Usage](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=ufw&f=1)

- 모든 stokes에서
  
  ```bash
  # root에서 실행
  ufw enable
  ufw allow ssh https ftp nfs # nfs - stokes2-9에서만
  ufw reload
  ```

- stokes1만 추가적으로
  
  ```bash
  ufw allow from 192.168.0.101 to 192.168.0.100
  ...
  ```

- stokes1
  
  ```
  root@stokes1:~# ufw status verbose
  Status: active
  Logging: on (low)
  Default: deny (incoming), allow (outgoing), allow (routed)
  New profiles: skip
  
  To                        Action        From
  --                        ------        ----
  
  22/tcp                     ALLOW IN     Anywhere
  80/tcp                     ALLOW IN     Anywhere
  21/tcp                     ALLOW IN     Anywhere
  192.168.0.100              ALLOW IN     192.168.0.101
  192.168.0.100              ALLOW IN     192.168.0.102
  ...
  22/tcp (v6)                ALLOW IN     Anywhere (v6)
  80/tcp (v6)                ALLOW IN     Anywhere (v6)
  21/tcp (v6)                ALLOW IN     Anywhere (v6)
  ```

- stokes2-9
  
  ```
  root@stokes2:~# ufw status verbose
  Status: active
  Logging: on (low)
  Default: deny (incoming), allow (outgoing), disabled (routed)
  New profiles: skip
  
  To                        Action        From
  --                        ------        ----
  
  22/tcp                    ALLOW IN      Anywhere
  80/tcp                    ALLOW IN      Anywhere
  21/tcp                    ALLOW IN      Anywhere
  2049                      ALLOW IN      Anywhere
  22/tcp (v6)               ALLOW IN      Anywhere (v6)
  80/tcp (v6)               ALLOW IN      Anywhere (v6)
  21/tcp (v6)               ALLOW IN      Anywhere (v6)
  2049                      ALLOW IN      Anywhere (v6)
  ```

### IP Masquerade 설정 - For stokes1

- [IP Masquerade란?](https://nsinc.tistory.com/100)

- [UFW : IP Masquerade](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=ufw&f=1)

- `/etc/ufw/before.rules` 수정
  
  ```
  # 파일 가장 아래의 "COMMIT" 밑에
  COMMIT
  
  # NAT
  
  *nat
  -F
  :POSTROUTING ACCEPT [0:0]
  -A POSTROUTING -s 192.168.0.0/24 -o enp5s0 -j MASQUERADE
  
  :PREROUTING ACCEPT [0:0]
  ```

### APT update

- Ubuntu를 default로 설치했다면 archive.ubuntu.com으로 저장소가 설정되어있는데, 아무래도 물건너 서버이다보니 속도가 조금 아쉬운 편이라 카카오에서 제공하는 미러로 변경하도록 한다.

- vi 로 /etc/apt/sources.list파일을 연 다음, 아래 명령어를 이용해 archive.ubuntu.com을 mirror.kakao.com으로 변경한다.

```vim
:%s/archive.ubuntu.com/mirror.kakao.com
```

- 다음 명령어를 통해 최신 버전으로 업그레이드 한다 - Linux 커널 설치 포함
  
  ```
  sudo apt update
  sudo apt upgrade -y
  ```
  
  > Troubleshooting : stokes2-9에서 업그레이드가 되지 않는 문제
  > IP Masquerade를 통해 stokes1에서 stokes 2-9로 데이터 전송 중 용량 초과인지 hash값이 맞지 않는다고 오류가 발생.
  > stokes1에 들어가는 외부 인터넷 선을 stokes 2-9에 연결한 후 /etc/netplan/00-installer-config.yaml 을 외부 ip로 수정, apt update 재수행.
  > 때론 무식해보이는 방법이 가장 좋은 방법일 수 있습니다.

### X11 Server Installation - For stokes1

- [ServerGUI](https://help.ubuntu.com/community/ServerGUI)

- [[이렇게 사용하세요!] Ubuntu GUI Server _X11 forwarding 설정 가이드](https://blog.naver.com/n_cloudplatform/221451046295)
  
  ```
  sudo apt install xorg xauth openbox
  sudo apt install yaru-theme-icon # icon package 다운로드
  xeyes # x11 setting 확인
  ```

### NFS : Network File System - For all stokes's

- [How To Set Up an NFS Mount on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)

- `/etc/exports` : on stokes1
  
  ```
  # /etc/exports: the access control list for filesystems which may be exported
  #                to NFS clients. See exports(5).
  /mnt/STORAGE 192.168.0.101(rw,sync,no_subtree_check)
  ...
  /mnt/STORAGE 192.168.0.108(rw,sync,no_subtree_check)
  /mnt/CALC_SSD 192.168.0.101(rw,sync,no_subtree_check)
  ...
  /mnt/CALC_SSD 192.168.0.108(rw,sync,no_subtree_check)
  /PROGRAMS 192.168.0.101(rw,sync,no_subtree_check)
  ...
  /PROGRAMS 192.168.0.108(rw,sync,no_subtree_check)
  ```

- `/etc/fstab` : on stokes2-9
  
  ```
  ...
  /swap.img    none    swap    sw    0    0
  
  192.168.0.100:/mnt/HOME /mnt/HOME nfs
  ```

### NIS : Network Information Service - For all stokes's

- [NIS : Configure Server](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=nis&f=1)

- [NIS : Configure Client](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=nis&f=2)
* Ubuntu 22.04로 올라오면서 세부내용이 조금 변했으니 버전에 맞는 자료를 참고하시길 바랍니다.
- `/etc/hosts/` : stokes1
  
  ```
  127.0.0.1 localhost
  192.168.0.100 stokes.nis
  127.0.1.1 stokes1
  ...
  ```

- `/etc/ypserv.securenets` : stokes1
  
  ```
  ...
  255.255.255.0    192.168.0.0
  ```

- `/etc/defaultdomain` : stokes1
  
  ```
  stokes.nis
  ```

- `/etc/yp.conf` : stokes2-9
  
  ```
  ...
  domain stokes.nis server stokes1
  domain stokes.nis server 192.168.0.100
  ```

- `/etc/nsswitch.conf` : stokes2-9
  
  ```
  passwd:         files systemd nis
  group:          files systemd nis
  shadow:         files nis
  gshadow:        files
  
  hosts:          files dns nis
  ```

- `/etc/defaultdomain` : stokes2-9  
  
  ```
  stokes.nis
  ```

- `/etc/pam.d/common-session` : stokes2-9
  
  ```
  session optional        pam_mkhomedir.so skel=/etc/skel umask=077
  ```

### Intel oneAPI - For stokes1

- NFS로 mount되는 PROGRAMS에 설치

- [Intel® oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html#gs.3uilpr)

- `/etc/profile.d/programs.sh`
  
  ```bash
  export INTEL_oneAPI_ROOT="/PROGRAMS/intel/oneapi"
  alias setvar="source ${INTEL_oneAPI_ROOT}/setvars.sh"
  export PATH=${INTEL_oneAPI_ROOT}/compiler/latest/linux/bin/intel64:${PATH}
  ```

### CUDA - For all stokes

- [CUDA Toolkit 12.2 Update 1 Downloads](https://developer.nvidia.com/cuda-downloads)

- 각 컴퓨터마다 다른 그래픽카드를 사용해서 그냥 각 컴퓨터에 설치하기로 했습니다.

- CUDA 설치시 GPU 드라이버도 같이 설치됩니다.

- Target Platform Information
  
  ```
  Operating System : Linux
  Architecture : x86_64
  Distribution : Ubuntu
  Version : 22.04
  Installer Type : runfile (local)
  # network 설치로 하면 stokes2-9 인터넷 문제때문에 runfile로 진행합니다.
  # deb (local)도 결국 인터넷에서 nvidia driver를 다운로드합니다.
  ```

- nouveau (Open Source Nvidia Driver) 비활성화
  
  - `/etc/modprobe.d/blacklist-nouveau.conf`
    
    ```
    blacklist nouveau
    options nouveau modeset=0
    ```
  
  - ```bash
    sudo update-initramfs -u
    sudo reboot
    ```

- 설치 command
  
  ```bash
  wget https://developer.download.nvidia.com/compute/cuda/12.2.1/local_installers/cuda_12.2.1_535.86.10_linux.run
  sudo sh cuda_12.2.1_535.86.10_linux.run
  ```

- ${PATH} 설정
  
  ```
  # in /etc/profile.d/programs.sh
  
  # CUDA-12.2
  export CUDA_ROOT="/usr/local/cuda-12.2"
  export PATH=${CUDA_ROOT}/bin:${PATH}
  export LD_LIBRARY_PATH=${CUDA_ROOT}/lib64:${LD_LIBRARY_PATH}
  ```

### miniconda3 (optional)

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html)

- `/PROGRAMS`에 설치 후 `/etc/profile.d/programs.sh`에 다음 추가
  
  ```bash
  # miniconda3
  # >>> conda initialize >>>
  # !! Contents within this block are managed by 'conda init' !!
  __conda_setup="$('/PROGRAMS/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
  if [ $? -eq 0 ]; then
    eval "$__conda_setup"
  else
    if [ -f "/PROGRAMS/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/PROGRAMS/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/PROGRAMS/miniconda3/bin:$PATH"
    fi
  fi
  unset __conda_setup
  # <<< conda initialize <<<
  ```

### OpenMPI

- [Open MPI: Version 4.1](https://www.open-mpi.org/software/ompi/v4.1/)

- `/etc/profile.d/programs.sh`
  
  ```bash
  # Open MPI-4.1.5 
  export OPEN_MPI_ROOT=/PROGRAMS/openmpi-4.1.5/
  export PATH=${OPEN_MPI_ROOT}/bin:${PATH}
  ```

### fail2ban - For stokes1, highly recommended

```bash
sudo apt install fail2ban
```
