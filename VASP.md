# R1215's Guide to VASP

Written by SinsuSquid (bgkang) on 02 October 2024
Edited by SinsuSquid (bgkang) on 29 October 2024

먼저, 어떠한 연유이든 VASP까지 건드시게 되어버린 연구자님께 심심한 위로의 말씀을 드립니다.

## What is VASP?
![VASP logo](https://www.vasp.at/wiki/images/vasp_logo_alpha.png)
VASP is short for "Vienna Ab initio Simulation Package". VASP는 양자 계산을 위한 시뮬레이션 패키지로, 재료 분야 등에서도 매우매우매우매우 활발하게 사용되고 있어 정보 찾기가 용이하다는 것이 장점입니다. 논문들 보면 상당수가 VASP 써서 양자계산을 하는데, MLP 학습을 위한 dataset의 상당수가 VASP format을 사용합니다. 편한 대신 상용 프로그램이라 돈 내고 써야되는게 흠이죠, 흠.

## VASP Portal
연구실에서 구매한 VASP 라이센스가 몇년짜리인지 저도 잘 모르겠네요. 연구자님 파이팅!

[VASP Portal](https://www.vasp.at/)

우측 하단에 "Portal" 통해서 로그인 하면 되고, 로그인 정보는 다음과 같습니다.

Username : [안알랴줌]
Password : [안알랴줌]

저는 로그인하니까 바로 다운로드 페이지로 넘어가네요. 해당 페이지에서 제공하는 파일들은 아래와 같습니다.

- VASP source files : 당연하게도 VASP 설치를 위해 필요하겠죵
- PAW POTCAR files : pseudopotential (potential 계산을 위한 기본값으로 생각하면 될 것 같습니다, 논문들 보면 어떤 pseudopotential 쓰는지 써있어요 - PBE, LDA 등)

Portal에 로그인 한 다음 다운로드 받아야해서 그런가 클러스터에서 wget으로 다운로드가 안됩니다. 귀찮지만 본인 컴퓨터에 다운로드받아서 옮길게요.

P.S. vdw_kernel.bindat 같은 파일은 optB88-vdW 계산같은거 할때 필요했던 걸로 기억합니다. 그냥 PBE에서는 안썼던 것 같아요.

P.S. 아래 부분을 읽기 전에 [VASP Wiki](https://vasp.at/wiki/index.php)를 꼭 들어가봅시다. 상상 가능한 대부분의 정보가 잘 정리되어있어요.

## How to install VASP

[Installing VASP.6.X.X](https://www.vasp.at/wiki/index.php/Installing_VASP.6.X.X)

vasp.6.x.x.tgz 파일의 압축을 풀면 다음 파일들이 생길거에요.
```bash
tar xvf ./vasp.6.x.x.tgz
```
- `README.md`
- `arch` (directory)
- `bin` (directory)
- `build` (directory)
- `makefile`
- `src` (directory)
- `testsuite` (directory)
- `tools` (directory)

먼저 `arch` 에 들어가서 내가 사용할 컴퓨터 환경에 해당하는 makefile.include 파일을 ./vasp.6.x.x/으로 옮겨줍니다.

```bash
# oneapi를 사용한다면,
cp ./vasp.6.x.x/arch/makefile.include.oneapi ./vasp.6.x.x./makefile.include
```

참고로, VASP에서 GPU 사용을 위해서는 A100 을 비롯한 계산용 GPU를 필요로 합니다. 그런 대단한거 연구실에는 없으니까 intel CPU 버전으로 깔아주도록 할게용.

복사한 파일을 열어주면 다양한 환경을 설정할 수 있습니다.
대부분 바로 되는데, 저의 경우에는 `MKLROOT` 라는 variable을 못 잡아주더라고요. `makefile.include` 파일을 열어 다음 부분을 찾아 수정해줍니다. 참고로, mkl은 math library인데 oneapi를 기본값으로 설치했으면 같이 딸려옵니다.

```
FCL		+=	-qmkl=sequential
MKLROOT	?=	/opt/intel/oneapi/mkl/latest # 본인 컴퓨터의 mkl library 경로 입력
LLIBS	+= -L$(MLKROOT)/lib/intel64 -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64
INCS	 = -I$(MKLROOT)/include/fftw
```

VASP의 장점은 이것만 되면 설치 준비 끝! 이란점이죠. 역시 돈의 힘인 걸까요?

```bash
make DEPS=1 -jN <target>
```

`target`에 해당하는 value는 `std`, `gam`, `ncl` 혹은 `all`이 있는데, `std` 또는 `all`로 설치하는걸 추천해요.

build가 끝나면 `./bin/` 에 binary 파일들이 생성됩니다. 적당히 `PATH`에 등록해주세요.

build가 정상적으로 되었는지 검사하는 test도 제공합니다

```bash
make test # all로 설치했을때만 모든 test를 통과하니 참고하세용!
```

## Basic Usage - Setup

VASP 시뮬레이션을 위한 기본 파일은 아래와 같아요.

- `INCAR` : input 파일
- `POSCAR` : initial 파일
- `KPOINTS` : *k*-point 파일
- `POTCAR` : pseudopotential 파일, (또는 `POTCAR.spec `, license가 걸려있어서 어떤 pseudopotential이 필요한지만 정의하고 배포하는 경우도 있습니다.)

### POSCAR
말 그대로 initial 파일입니다. ion position 정의해주는 파일이고, ovito에서 `POTCAR` format으로 export가 가능하니까 참고하세용.

### POTCAR
POTCAR에는 ion 종류에 따른 pseudopotential이 정의되어있는데, 이게 type별로 순서대로 들어가야 합니다. (*e.g.* type 1,2,3이 Li, P, S라면 POTCAR에는 Li_sv, P, S 순으로 입력되어야 함)

VASP portal에서 다운받을 수 있는 POTCAR file로부터 만들어집니다.

```bash
# An example - LiAlCl4
cat ../PAW_PBE/Li_sv/POTCAR ./POTCAR
cat ../PAW_PBE/Al/POTCAR ./POTCAR
cat ../PAW_PBE/Cl/POTCAR ./POTCAR
```

### KPOINTS
*k*-point가 어떤 개념인지는 꼭 YouTube등을 통해 공부해보도록 해요.

*e.g.* regular gamma-centered 1 x 1 x 1 mesh
```
# THIS IS THE COMMENT LINE
0
Gamma
1 1 1
0 0 0
```

### INCAR
이제 VASP의 꽃, `INCAR` 입니다. 시뮬레이션을 위해 필요한 대부분의 설정들을 `INCAR` 안에서 건드리게 되는데요, 이걸 전부 알려드리는것이 불가능할 정도로 다양한 customizing이 가능합니다. AIMD, Geometry Optmization, DFP energy calculation중 어떤 시뮬레이션을 할지도 이 파일을 통해 결정할 수 있어요.

어차피 다 설명드리지 못하기 때문에 LiAlCl4의 optB88-vdW 계산을 위해 사용했던 예시 파일을 첨부합니다.

```
PREC = Accurate
ALGO = Normal
ENCUT = 520
EDIFF = 0.0012
ISMEAR = 0
LREAL = Auto
SIGMA = 0.5
ISYM = -1

IBRION = -1
ISIF = 2

# For obtB88-vdW
GGA = B0
PARAM1 = 0.183
PARAM2 = 0.22
AGGAC = 0.0
LUSE_VDW = .TRUE.
LASPH = .TRUE.

NCORE = 12
```

위와 같은 INCAR는 ion 이동 없이 POSCAR에 정의된 configuration의 에너지를 계산합니다.
particle의 위치를 변경시키며 *ab initio* MD (AIMD)를 수행한다면, [Molecular dynamics calculations](https://www.vasp.at/wiki/index.php/Molecular_dynamics_calculations) 페이지를 참고하여 추가로 필요한 정보를 입력해주세요!

꼭꼭꼭 [VASP wiki](https://vasp.at/wiki/index.php)에서 해당 line이 어떤 설정을 건드리는건지 알아보고 계산 돌리도록 해요!

P.S. System별로 서로 다른 parameter를 사용하게 될텐데,  [Materials Project](https://next-gen.materialsproject.org) 에서 structural relaxation을 위한 input을 다운로드 받을 수 있으니 이부분을 잘 참고해보아요!

## Basic Usage - Simple Run
모든 준비가 완료되었으면 이제 폴더에서 VASP를 실행시켜보도록 해요.

```bash
mpirun -np NCORE vasp_std
```

이때 `N`에 해당하는 값은 `INCAR`의 `NCORE`와 동일해야 한다는걸 꼭 기억해요~!

출력되는 `log`에서 warning은 없는지 꼭 체크해보고, 필요에 따라서 `INCAR`를 수정하도록 해요.

계산이 수행되면 이런저런 파일들이 많이 생기는데, 자주 쓰는 몇가지를 확인해보면

- `XDATCAR` : AIMD 등을 수행할 때 ion들의 trajectory를 저장하는 파일입니다. ovito에서도 잘 열려요.
- `OUTCAR` : 경과 및 결과를 저장하는 파일인데, 단순한 log파일이라 하기에는 엄청 많은 정보가 들어있습니다. 어떤 MLP는 `OUTCAR` 파일 하나로 전부 다 처리하기도 해요! 계산 시간등에 대한 정보도 이 파일에 있으니까 확인해보아요~

## Extras
- optB88-vdW를 비롯한 vdW (van der Waals) 계산을 위해서는 DFT 계산을 위한 폴더 내에 vdw_kernel.bindat 파일을 필요로 합니다. 없어도 돌아가긴 하는데 kernel initialize를 위해 어마어마한 시간이 필요하니 꼭 넣어주도록 해요!

## Comments 
저 역시 양자라고는 "재드래곤의 양자가 되고싶다." 정도밖에 모르는 상태에서 작성했고, 거의 대부분의 정보는 [VASP Wiki](https://vasp.at/wiki/index.php)에서 가져왔어요. Wiki에서 해결되지 않는 질문들은 Google이나 [VASP Forum](https://vasp.at/forum)에서 더 찾아볼 수 있으니까 꼭꼭꼭 여기저기 뒤져보며 문제를 해결하는 방법을 배워나가보도록 해요.

P.S. 더 이상은 저한테 물어보지 마세요 >:D
