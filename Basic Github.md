# Basic GitHub

written by SinsuSquid (bgkang) on 19 November 2024

GitHub는 주로 code나 package등을 공유하기 위해 주로 사용되는 platform이고, 쉽게 말하면 전 세계의 나서기 좋아하는 찐따들이 (뜨끔!) 자기가 만든거 자랑하는 곳이에요.
흔히 GitHub를 Open Source의 성지라고 말하는데, 이 말은 내가 짠 모든 code들이 문자 그대로 '공개'되는 장소라는 의미에요.
그러니까 음침한 애니프사 전문가가 어느날 갑자기 와서는 "님 code 그렇게 짜면 안됨 ㅉㅉ" 할 수 있는 공간이라는 것이죠.

자, 그래도 우린 순기능에 집중해보기로 하죠.
다른 사람이 작성한 code를 command 몇줄로 내 컴퓨터로 가져와서 바로 쓸 수 있다는게 아마 우리가 GitHub을 사용하는 주 목적이 될겁니다.
그렇기 때문에 이 글에서는 'GitHub에서 repository를 clone해서 내 컴퓨터에 가져와서 사용하는법'에 집중해서 설명하도록 하죠.
"아뉜데 아뉜데! 난 내 code 자랑하고슆은데!"하는 사람은 여기서 알짱거리지 말고 YouTube검색이나 먼저 하고 오세요.

## How to clone a repository

먼저 repository는 'code 저장소'라고 생각하면 될거에요.
이런 repository를 'clone'한다는건 저장소로부터 code를 내 컴퓨터를 복사한다는 개념입니다.
예시로 [이런 repository](https://github.com/SinsuSquid/Wildflower-Class)가 있다고 해보죠. (조회수 올릴라는거 아닙니다 흠흠. star라도 눌러주면 조금 고맙긴 하겠네요, 흥!)
들어가면 이런 화면이 보일 터인데, 아무래도 빨간 상자는 눌러보라고 있는거겠죠?
![Figure00](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/00.png)
누르면 아래처럼 될거고,
![Figure01](https://github.com/SinsuSquid/Wildflower-Class/blob/main/GitHub/Figure/01.png)

빨간 버튼을 클릭하면 이번엔 repository의 주소를 복사할거에요.

자, 주소를 복사했으면 이번에는 clone할 컴퓨터의 terminal을 접속해봅시다.
```bash
git clone https://github.com/SinsuSquid/Wildflower-Class
```
- PS1. `git`이 설치 안되어있다면 컴퓨터 관리자한테 달려가 설치해달라고 합시다.
- PS2. 만약 당신이 컴퓨터 관리자라면, `git`도 설치 안하고 뭐했어요?

이런식으로 clone을 해오면 이제 여러분의 컴퓨터에 code가 다운받아져있을거에요.
여기서 code 수정한다고 해서 online repository에 있는 code가 바뀌는건 아니니까 마음대로 수정하면 됩니다.

- PS. 만약 '난 어차피 `git`도 잘 안쓰는데, 자꾸 terminal에 정보 뜨는게 거슬린다 싶으신 분은 해당 폴더 내 `.git` 폴더를 삭제하면 됩니다.
  ```bash
  rm -rdf ./.git
  ```
  물론 이 폴더를 삭제할 경우 더 이상 `git` 기능은 사용할 수 없어요.

## Wrap Up!
오늘 한건 컴공과 애들이 보면 코웃음칠정도로 기본에기본에기본의 기능만 사용한거라 `git`을 조금 더 잘 써보고싶은 사람은 따로 공부가 필요할거에요.
물론 제가 알려주진 않을겁니다 ㅎ (YouTube 봐요 ㅎㅎ)
앞으로 남의 코드를 잔뜩 배껴서 내 마음데로 사용해보도록 해요!
