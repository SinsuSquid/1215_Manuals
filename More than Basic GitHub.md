# More than Basic GitHub
written by SinsuSquid (bgkang) on 20 November 2024

원래는 간단한 `git clone` 정도 쓰는 법만 알려주고 끝내려고 했는데, JWS와 YJ가 author를 충분히 motivate해서 GitHub 사용법에 대해 조금 더 정리해볼까 해요.
이번에는 repository를 만들고, 만들어진 repository에 내가 작성한 code를 `add` / `commit` / `push`하는 방법을 알려드릴까 해요.
저도 GitHub의 주 사용 목적이 내가 작성한 code 저장이다보니, branching까지 크게 신경쓰면서 관리하고 있진 않은 것 같아요.

## Opening a repository
open이란 단어를 고르긴 했지만, 실제 GitHub을 잘 쓰시는분들도 open이라 하시는진 잘 모르겠네요.
어쨌든 repository를 생성하는 방법입니다.
먼저 본인 GitHub에 repository 탭에 들어가면 다음과 같은 페이지를 볼 수 있을거에요.
![Figure02](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/02.png)
빨간 박스로 표시한 부분을 눌러 새로운 repository를 생성해봅시다.
![Figure03](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/03.png)
repository를 설정하기 위해 이것저것 건드릴 수 있는데, 하나씩 확인해보기로 하죠.

- red :  말 그대로 repo의 이름입니다. repo 주소도 https://github.com/SinsuSquid/[REPO_NAME] 같은식으로 정해지니까 참고하길 바래요. (생성 후 수정 가능)
- green : repo에 대한 간단한 설명입니다. (생성 후 수정 가능)
- blue (중요) : repository의 공개 범위를 설정하는 부분입니다. Public / Private로 구분되며 글자 그대로 공개를 하느냐 혼자 볼거냐 하는 걸 결정하죠. Public repo는 다른 사람이 여러분의 code를 읽을 수 있지만 계정별 용량 할당량을 사용하지 않습니다. (그렇다고 클라우드처럼 사용할 수 있다는 건 아닙니다. GitHub repo는 기본적으로 약 30 Mb의 용량 제한이 있어요.) Private로 설정하는 경우 오직 repo 주인만 code를 볼 수 있지만 개정별 용량 할당량을 사용합니다. 물론 Pro 버전을 쓴다거나 해서 용량 할당량을 늘릴 수 있습니다. 반드시 꼭 그래야 하는건 아니지만, 저같은 경우 분석용 code들은 Public으로 공개해서 쓰고있고, GNN Project같은 경우는 Private로 작업하다 논문 submit하면서 Public으로 전환했습니다. (생성 후 수정 가능)
- purple : `.gitignore`파일을 설정하는 부분입니다. 아무래도 GitHub같이 공개되는 장소에서는 code 실행시 생성되는 log파일까지 같이 올릴 필요는 없겠죠? `.gitignore`파일에 `*.log`와 같이 추가하게 되면 해당 repo 안에 있는 *.log 파일은 무시하고 업로드하지 않습니다. 여기서는 이 `.gitignore`파일의 기본 template를 설정해달란 뜻입니다. "난 이 프로젝트 Python으로 짜겠다!" 하면 `.gitignore` template를 `Python`으로 설정하면 되겠습니다. (나중에 개인화 가능)

자, 이제 `Create repository`를 눌러 repo를 만들어보도록 하겠습니다.

처음 만든 repo를 들어가보면 아래와 같은 모습을 확인할 수 있을거에요 (물론 아직은 file이 없겠지요?)
![Figure00](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/00.png)
![Figure01](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/01.png)
만든 repo를 내 컴퓨터로 clone해옵시다.
```bash
git clone https://github.com/SinsuSquid/[REPO_NAME]
```

# Workflow with an example
지금부터는 우리가 "Analysis-Toolbox"라는 reposity를 만들었고, 이 안에는 trajectory를 분석하는 code를 저장할거라고 생각해보죠.
저같으면 directory 구조를 다음과 같이 구성할 것 같네요.
```bash
Analysis-Toolbox/
├── .git
├── .gitignore
├── README.md
├── bin
│   └── test.x
├── src
│   └── test.c
├── test.lammpstrj
└── test.out
```

뭐, 간단하게만 설명을 하자면, `./src/test.c`를 compile해서 `./bin/test.x`를 만들고, 이걸 `./bin/test.x ./test.lammpstrj ./test.out`이런식으로 써서 결과를 얻는 code겠죠?
여기서 깔끔한 repository를 위해 좀만 thinking을 해봅시다.

1. `./bin/`에 있는 파일은 어차피 다른 컴퓨터에서 사용하려면 새로 compile을 해줘야 할거에요. 그렇다면 굳이 `./bin/`을 같이 공개할 필요는 없겠죠? 그래서 저는 `.gitignore`에 `/bin/*`을 추가하겠어요.
2. `test.lammpstrj`나 `test.out`같은 파일들은 code test용으로 돌려본 파일이고, 실수로 결과들까지 Public repo에 올린다면 누군가 내 data를 가져갈수도 있을거에요! 그러니까 `.gitignore`에다 `*.lammpstrj`랑 `*.out`을 추가하는게 안전하겠죠.

일단 이런 식으로 설정을 해놓은 다음, 나는 `./src/test.c`파일을 수정하며 code를 작성하겠죠.
code 작성이 끝나고 이제 공개를 하겠다! 라고 결정을 하면 다음 command를 사용하게 됩니다.
```bash
git add ./* # 해당 폴더의 모든 file을 repository에 추가 - 단, .gitignore에 등록된 파일은 제외
git commit -m "[COMMIT_MESSAGE]" # commit - 해당 상태까지의 변경사항을 확정하는 단계 - commit 한번이 record 한개, commit message에는 code의 어떤 부분을 수정했는지 써주면 좋겠죠?
git push # 완료된 commit을 online repository에 업로드
```

이렇게 push까지 완료했다면, 이번엔 https://github.com/SinsuSquid/Analysis-Toolbox에 들어가서 내가 원하는데로 잘 반영되었나 확인해보면 될 것 같아요!

## Comment
여기서 설명한 `git`의 사용법은 기초중의 기초만 정리한거고, 다른 사람과 협업등을 하는 경우 무궁무진하게 다양하게 활용할 수 있어요!
일단 제가 알려드릴 수 있는 부분은 여기까지니까, 앞으로 더 알고싶은 내용이 있으면 Google과 YouTube의 도움을 받아 해결해나가길 기대할게요!
