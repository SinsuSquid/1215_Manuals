# R1215's Guide to SLURM

Written by SinsuSquid (bgkang) on  12 November 2024

## What is "SLURM?"
[Slurm Workload Manager](https://slurm.schedmd.com/documentation.html) 는 클러스터를 교수님처럼 철저하게 갈궈서 성능을 최대한으로 뽑아먹기 위한 노예 감독관이라고 생각해주시면 됩니다! (물론 교수님이 노예 감독관이란 뜻은 아니에요 히히)

여러 계산을 돌리다가 한번쯤은 이런 생각을 해봤을 것 같아요.
"아 emsenble을 10개 돌려야되는데 내가 일일이 가서 해줄게 아니라 한번에 쭈욱 줄 세워놓고 빈자리 생기면 알아서 들어가면 안되나?"
이런 생각을 가능하게 만들어주는게 workload manager의 역할입니다.

사실 예전에도 SLURM 설치 메뉴얼은 작성해놨는데, 막상 사용법에 대한 설명이 부족한거 같아서 오늘은 그부분을 더 알아보기로 할게요.

## How to Install SLURM
이건 다른 메뉴얼에서 작성해둔적이 있어요.
버전/환경에 따른 차이는 물론 존재하겠지만 그건 이제 더 이상 제 관심사가 아니죠 >_<
이 글을 읽는 연구자님이 직접 구글링을 통해 해결해보기로 해요~ 꺄~

## OK, before we start!
그럴 일은 일어나지 않으면 최선이겠지만, SLURM이 잘 돌아가지 않을 때를 대비해서 어떤 파일들을 확인해야 하는지 알아두는게 좋겠죠?

- SLURM 설정 파일 (`/etc/slurm/`이나 `/usr/local/etc/`)에 있어요
	- `slurm.conf` : SLURM 기본 설정 파일
	- `gres.conf` : GPU 관련 설정
	- `cgroup.conf` : 자원(?) 확인 - CPU 등 - 설정 (버전에 따라 불필요한 경우도 있음)

- SLURM log 파일 (`/var/log/`)
	- `slurmdctld.log` : `slurmctld` (head 데몬)의 log (head node에서 확인해야겠죠?)
	- `slurmd.log` : `slurmd` (slave 데몬)의 log (slave node에서 확인해야겠죠?)

뭔가 안된다 싶을때는 최소한 log라도 확인하고 administrator에가 달려가야 admin이 좋아하겠죠? \^_\^

## Some Useful Tools
- `sinfo` : SLURM node들의 상태 출력, `-R`으로 세부 정보까지
	- `sinfo -o "%n %e %m %a %c %C"` : SLURM node의 memory, CPU 상태 확인 (A/I/O/T - Allocated / Idle / Others / Total)

- `slurmd` : SLURM slave daemon이고, 주로 `systemctl`로 등록해놓기 때문에 직접 사용할 일은 많이 없습니다만, `-C` 옵션을 사용하면 CPU, Memory와 관련한 정보를 `slurm.conf`와 비슷한 형태로 출력해주기 때문에 유용합니다.

- `squeue` : 현재 queue에 등록되어 있는 작업들 확인
	- 기본값은 JOB NAME이 더럽게 짧게 설정해놓았더라고요, `squeue --format="%.18i %.9P %.30j %.8u %.8T %.10M %.9l %.6D %R" --me`처럼 format 지정해서 사용하는것도 가능합니다.

- `sbatch` : SLURM 사용의 꽃, 알파이자 오메가. 작업 배치 명령
	- 주로 `sbatch` 파일 하나 만들고 (진정해요, 나중에 설명해드릴게요) `sbatch slurm_script.sh`와 같이 실행합니다.
	- `for`문 사용해서 1000개정도 되는 작업들 한번에 주르륵 올려놓으면 그 쾌감이 진짜 엄청나요.

-`scontrol` : 원래는 node 상태 설정을 위해 사용
	- `scontrol show node` : detail한 node 상태 확인

명령어와 관련해서 더 필요한 정보들은 저보다 [이 친구](https://slurm.schedmd.com/documentation.html)가 더 확실하니까 필요할때 찾아보도록 해요.

## How to Set `sbatch` file
이제 작업들 줄 세워놓는 방법은 알았으니, 어떤 작업들을 해야되는지 알려줘야겠죠?
이런 역할을 하는 파일이 바로 `sbatch.sh`입니다.

저같은 경우는 주로 1 ensemble = 1 directory로 세팅해놓고 계산 수행하는데요, 그렇기에 directory 하나마다 `sbatch` 파일 하나 만들어주는 식으로 사용합니다.

`sbatch` 의 format은 `bash`랑 똑같다고 생각하면 되고, 몇가지 정보들만 살짝 추가되는 정도입니다.
아래는 `VASP`와 `lammps` 계산 수행할 때 썼던 파일들 예시를 나타내고 있어요.

- `VASP`
```bash
#!/bin/bash

#SBATCH --job-name=MY_JOB_NAME
#SBATCH --output=%x.out # %x - job-name의 wildcard
#SBATCH --error=%x.err
#SBATCH --cpus-per-task=4 # 작업에 필요한 cpu core 수
#SBATCH --mem=3072M # 작업에 필요한 memory

# PS : --cpus-per-task나 --mem은 작업 allocation에 필요한 정보를 제공해주는 역할입니다!

MLIP_ROOT=/path/to/mlip

mpirun -np 4 vasp_std # 이부분에서 실제로 사용하는 core 수를 알려줘야 해요
${MLIP_ROOT}/mlp convert --inpit_format=outcar --output_format=txt ./OUTCAR ./out.cfg
cat ./out.cfg >> ../dft.cfg
```

- `lammps`
```bash
#!/bin/bash

#SBATCH --job-name=NVT_loop00
#SBATCH --output=%x.out
#SBATCH --error=%x.err
#SBATCH --cpus-per-task=12

ROOT=`pwd`
LAMMPS_ROOT=/path/to/lammps
ANAL_ROOT=/path/to/analysis_tools

source /opt/intel/oneapi/setvars.sh &> /dev/null

# Melt-quench
mpirun -np 12 ${LAMMPS_ROOT}/lmp_oneapi -i input_melt.lammps -var SEED `date +%s`
${LAMMPS_ROOT}/lmp_oneapi -restart2data melt.restart melt.data
mpirun -np 12 ${LAMMPS_ROOT}/lmp_oneapi -i input_quench.lammps
${LAMMPS_ROOT}/lmp_oneapi -restart2data quench.restart quench.data

# NpT equilibration
mpirun -np 12 ${LAMMPS_ROOT}/lmp_oneapi -i input_NpT.lammps
${LAMMPS_ROOT}/lmp_oneapi -restart2data NpT.restart NpT.data

# NVT production
mpirun -np 12 ${LAMMPS_ROOT}/lmp_oneapi -i input_NVT.lammps

# If you need some post-simulation analyses,
${ANAL_ROOT}/bin/msd ./NVT.lammpstrj ./msd.out
...
```

예시에서 확인할 수 있는 것처럼, 실제로 SLURM을 쓸때 신경써주어야 하는 부분은 `#SBATCH ...`입니다.
이 파일에 몇가지 정리해놓긴 하겠지만, 꼭 [여기](https://slurm.schedmd.com/sbatch.html)에 들어가서 더 알아보기로 해요.

- Some useful `#SBATCH` options
	- `--job-name=NAME` : 작업 이름이겠죠?
	- `--output=%x.out` : `stdout` 저장 파일, `%x.out` 라고 설정하면 `JOB_NAME.out`으로 생성됩니다.
	- `--error=%x.err` : `stderr` 저장 파일
	- `--cpus-per-task=N` : 작업에서 사용하는 최대 cpu core의 수 (default : node 전체)
	- `--mem=M` : 작업에서 사용하는 최대 memory (default : node 전체)
	- `--=nodelist=node1` : 해당 작업에 사용할 node
	- `--gres=gpu:rtx3090:1` : GPU 사용 (`slurm.conf` 보고 사용하기)

이렇게 `sbatch.sh` 파일을 잘 만들어놓으면 head node에서 다음과 같이 사용하면 됩니다.
```bash
sbatch sbatch.sh # 참 쉽죠?
```

작업을 제출했으면 다음 명령어를 통해 잘 들어갔나 확인해보도록 해요.
```bash
squeue # 참 쉽죠?
```

## Troubleshooting

0. 작업이 안들어가요! (how NOT to upset an admin)
	- `sinfo`로 node 상태를 확인한다
	- 해당 node에 직접 들어가 `systemctl status slurmd`로 SLURM 데몬이 켜저있는지 확인한다.
	- `/var/log/slurmd.log`와 `/var/log/slurmctld.log`을 확인한다.

1. 작업이 한 node에 하나밖에 안들어가요!
`--mem`이나 `--cpus-per-task`를 설정하지 않으면 default로 node 1개를 통째로 잡아먹습니다.
이런 돼지같은 심보를 고쳐주기 위해서는 test로 계산 돌려보고 몇개의 cpu랑 얼마만큼의 memory가 필요한지 대충 계산해본 뒤 `sbatch.sh`파일에 넣어주도록 해요.

2. TO_BE_UPDATED

## Comment
1. 왜 torque 말고 SLURM을 쓰나요?
	- SLURM이 GPU 설정이 가능합니다.
	- 자료 찾기가 더쉽습니다.
	- 이름이 더 마음에 듭니다.
	- 그냥 연구실에서 쓰던거 쓰기 싫었습니다.

2. 어떻게 다 알고계신건가요?
	- 피, 땀, 눈물.

3. 하고 싶은 말은?
	- 잘 구성한 SLURM 하나 열명의 대학원생보다 낫다.

4. 졸업 후에도 물어봐도 되나요?
	- 뒤질래요?
