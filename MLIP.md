# R1215's Guide to MLIP
Written by SinsuSquid (bgkang) on 04 November 2024

가봤던 길이라 이것저것 다 보다보니 누구보고 따라오라고 하기가 참으로 마음이 찢어지네요.
연구자님, 행복하세요.

## What is MLIP?
### Intro
MLIP는 (아마도?) Machine Learning Interatomic Potential의 약자로, 이름이 얘기하는 것 처럼 머신러닝을 통해 atom 사이의 potential을 학습하여 force를 계산하는 방법입니다.

### Moment Tensor Potential (MTP)
MTP : Polynomial의 linear combination을 다차원 fit이라 생각하시면 될 것 같습니다. 자세한 description은 [J. Chem. Phys. 159, 084112 (2023)](https://pubs.aip.org/aip/jcp/article/159/8/084112/2908187/MLIP-3-Active-learning-on-atomic-environments-with)을 참고해주세요.

MTP를 비롯한 다수의 machine-learning potential (MLP)는 아래 가정을 기반으로 출발합니다.

$E^{\rm MLP} = \sum^{N}_{i=1}V^{\rm MLP}({\mathbf n}_i)$

각 입자의 potential을 linear combination하여 하나의 configuration의 전체 에너지를 결정한다는 의미입니다.

이때 $V^{\rm MLP}({\mathbf n}_i) = V^{\rm MTP}({\mathbf n}_i)$는 다음의 형태로 정의되는데요,

$V^{\rm MLP}({\mathbf n}_i) = \sum^{N\_{lin}}\_{\alpha = 1}{\xi\_\alpha B\_\alpha({\mathbf n}_i)}$

여기서 $\xi_\alpha$는 linear parameter (fitting 통해 최적화) $B_\alpha({\mathbf n}_i)$는 basis function (predefined)이라 얘기합니다.

더 자세한 basis function의 기술 방법은 논문에서 소개되고 있는데, 이 부분은 생략하도록 하겠습니다.

결과적으로 DFT 결과를 기반으로 하여 다차원 fitting을 통해 원하는 형태의 interatomic potential을 기술하는 방법이라는 점을 아시면 됩니다.

### Active Learning
저는 passive learning 조차도 하기 싫은데 대단하네요.

아무리 잘 학습된 potential이라 하더라도 training set과 다른 data가 있다면 accuracy는 조금 떨어질 수 밖에 없습니다. 그렇기 때문에 어떠한 configuration에 대한 energy를 계산할 때 모델이 "아 이건 좀 자신 없는 예측인데"를 판단하여 자신 없는 예측에 해당하는 configuration을 따로 추려내고, 추려낸 configuration의 DFT 계산을 통해 "정답"에 해당하는 energy를 계산합니다. 해당 데이터를 training set에 추가하여 다시 training하면 이전 단계보다 더 general한 potential을 얻을 수 있습니다. 

와! 제가 공부를 저렇게 했었다면 하버드에 갔겠는데요?

MLIP에서는 이런 과정을 learning on the fly (LOFT)라 지칭합니다. 그렇다면 무엇이 "자신 없는 예측"인지 평가하는 방법 또한 필요할건데요, 이를 위해 extrapolation grade ($\gamma$)라는걸 계산합니다. 논문에 계산 과정이 자세하게 설명되어있지만 사실 저도 잘 이해를 못했는데요, 이해해본 바에 따르면 더 "자신 없는" 예측값일수록 큰 $\gamma$값을 가지게 되고, 일정 $\gamma$이상의  configuration들을 추리게 됩니다. 만약 엄청 큰 $\gamma$값이 계산된다면 더 이상의 simulation은 의미가 없다고 판단하고 중간에 MD 계산을 종료합니다. 각 기준을 논문에서는 $\gamma_{\rm save}$와 $\gamma_{\rm break}$로 정의합니다.

 
## Strategy
먼저, 이 전략은 inorganic glass electrolyte 시스템에서 적용해본 결과에 따른 것이며 아직 논문이 accept되지 않아 peer review를 거치지 않았음을 밝힙니다. "강력한 권고사항" 정도로만 받아들이시면 될 것 같습니다.

MTP 학습을 위한 데이터는 높은 정확도의 (configuration, energy) pair입니다. 경험적으로 확인하였을 때 quenching처럼 system density가 변하는 경우, 하나의 density에서 학습된 MTP가 불안정함을 확인했습니다.

그렇기에 non-Eq. configuration이나 다양한 density scenario를 training set에 포함시켜야 했는데, 이를 그나마 효과적으로 수행해보고자 다음과 같은 전략을 세웠습니다.

1.  Crystalline 상태의 density ($d_{\rm crystal}$)와 비슷한 정도의 density에서 낮은 정확도의 high Temp. AIMD를 이용해 atomic trajectory를 만든다 (핵심 - 빠른 계산)
2. (1)의 중간중간 configuration을 뽑아내고, 이를 initial로 하는 새로운 낮은 정확도의 high Temp. AIMD를 추가로 수행한다 (핵심 - 빠른 계산)
3. (1-2)로부터 $d_{\rm crystal}$에 해당하는 configuration을 많이 만든다 (configuration 사이 correlation이 작다는 가정, 경험적으로 300-500개, 자신 없음)
4. [필요시] AIMD로부터 configuration을 가지고 높은 정확도의 DFT (*e. g.* optB88-vdW basis set)를 다시 해본다 (핵심 - 높은 정확도)
5. (4)까지 과정으로 얻은 data로 MTP 학습하고 MLMD 돌려본다 (LOFT-MD 포함)
6. (5)에서 결과가 시원찮은 경우에,
7. $d_{\rm crystal}$의 configuration을 manipulate하여 원하는 density ($d_{\rm target}$)의 configuration을 만든다
8. $d_{\rm target}$의 configuration 가지고 DFT 계산을 하여, 이를 data에 추가하여 학습한다 (DFT converge가 안되는 경우는 버린다)
9. 자신이 믿는 신께 간절히 기도하며 제발 MLMD가 잘 되고 시간 안에 졸업할 수 있길 빈다

저는 inorganic glass system에서 0.5, 1.0, 1.5 $d_{\rm crystal}$ 에 해당하는 configuration각 300개로 진행한 결과 적당한 정확도의 MTP를 얻을 수 있었습니다만, 책임은 지지 않겠습니다.

## Installation
- MLIP-3 : [mlip-3/INSTALL.md](https://gitlab.com/ashapeev/mlip-3/-/blob/main/INSTALL.md) 에 매우 잘 설명되어있어 생략합니다
- LAMMPS interface : LAMMPS에서  MLIP-3을 사용하기 위해 사용합니다. 설치 방법은 [LAMMPS-MLIP-3 interface ](https://gitlab.com/ivannovikov/interface-lammps-mlip-3/)에 잘 설명되어 있습니다

설치가 잘 되었는지는 `${MLIP-3}/bin/mlp help`를 활용하거나 LAMMPS script 내부 `pair_style mlip load_from = ${POTENTIAL}`을 이용해서 알아보아요.

## MLIP.howto
`${MLIP-3}/bin/mlp list`및 `${MLIP-3}/bin/mlp help [command]`를 잘 활용하면 MLIP의 활용법을 잘 배워볼 수있을거에요. 간단하게 어떤 역할인지만 설명해보면

- `${MLIP-3}/bin/mlp calculate_efs` : `.cfg`파일로부터 MTP를 이용해 energy 및 force 계산 (parity plot 그릴 때 주로 사용)
- `${MLIP-3}/bin/mlp select_add` : `pretrained.cfg` (LOFT-MD에서 추려진 configuration들)로 부터 새로운 DFT 계산이 필요한 cfg를 추리기
- `${MLIP-3}/bin/mlp train` : MTP 모델 학습
- `${MLIP-3}/bin/mlp convert` : `.cfg`, `.lammps_data`, `POSCAR`, `OUTCAR` 사이의 converting. Trajectory의 경우 여러개의 POSCAR 파일로 변환되니 신경써주길 바래요

[MLIP-2 Tutorials GitLab](https://gitlab.com/ashapeev/mlip-2-tutorials/-/wikis/home)에 MTP training scheme 진행 방법이 잘 설명되어 있으니 참고할 수 있길 바래요.

자세한 command는 위의 wiki에서 확인하기로 하고, 저는 workflow를 간단하게 요약하도록 할게요.

1. AIMD 및 DFT 계산을 통해 (configuration, energy) pair를 얻는다
2. `convert` 기능을 이용해 `OUTCAR`를 `.cfg` 형태로 변환한다
3. untrained MTP (주로 `.almtp` 사용)에 (2)의 data를 학습시킨다
4. 학습된 MTP를 가지고 AIMD랑 비슷한 system에서 learning-on-the-fly molecular dynamics (LOFT-MD)를 수행한다
5. LOFT-MD 중간에 만들어진 `preselected.cfg`로부터 실제로 추가 계산이 필요한 `selected.cfg`를 추려낸다
6. 추려진 `selected.cfg`를 `POSCAR`로 변환한다
7. (6)에서 얻어진 `POSCAR`를 가지고 DFT 계산을 수행한다
8. (7)을 다시 `.cfg`로 변환하고 기존 training set에 추가한다
9. (5-8)을  NpT, NVT에서 `preselected.cfg`가 만들어지지 않을 때까지 반복한다
10. 자신이 믿는 신께 간절히 기도하며 제발 앞으로 MD 돌리면서 터지는 일이 없게끔 기원한다

읽으면서 머리가 아프시죠? 옆에 선배한테 물어보면서 배워보도록 해요. 그래도 위안을 주면 이 단계만 거치고 나면 앞으로는 MTP 건드릴 일이 많이 없답니다.  

LAMMPS에서 `.almtp`를 사용하는 경우 아래와 같이 load합니다.
```
pair_style	mlip load_from=myMTP.almtp
pair_coeff	* *
```

LOFT-MD의 경우 조금 더 복잡해지는데요,
```
pair_style	mlip load_from=myMTP.almtp \
			extrapolation_control=true \
			extrapolation_control:threshold_save=2 \
			extrapolation_control:threshold_break=10 \
			extrapolation_control:save_extrapolative_to=preselected.cfg
pair_coeff	* *
```

## Comments
일단 생각나는데로만 적어보았는데, 프로젝트를 진행하며 잘 모르시는 부분을 여쭈어주시면 이 메뉴얼에 살이 더 붙어나갈 것 같아요!

메뉴얼을 읽으면서 감정기복이 느껴졌다면 정확하게 짚었어요! 두 자아가 감정의 주도권을 위해 싸워가며 작성한 메뉴얼이 맞아요!

## References
- [Evgeny Podryabinkin, Kamil Garifullin, Alexander Shapeev and Ivan Novikov, MLIP-3: Active learning on atomic environments with moment tensor potentials, J. Chem. Phys. 159, 084112 (2023)](https://pubs.aip.org/aip/jcp/article/159/8/084112/2908187/MLIP-3-Active-learning-on-atomic-environments-with)
- [Skoltech MLIP](https://mlip.skoltech.ru)
- [MLIP-3 GitLab](https://gitlab.com/ashapeev/mlip-3)
- [MLIP-2 Tutorials GitLab](https://gitlab.com/ashapeev/mlip-2-tutorials)
