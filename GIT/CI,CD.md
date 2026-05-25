# 목차


# CI 

CI(Continuous Integration)란 개발자가 코드를 자주 통합하고, 각 통합을 자동으로 빌드하고 테스트하는 프로세스를 의미.

# CD

CD(Continuous Deployment)란 코드 변경 사항이 자동으로 프로덕션 환경에 배포되는 프로세스를 의미.
젠킨스는 오픈 소스 자동화 서버로, 다양한 플러그인을 통해 소프트웨어 개발의 빌드, 배포, 자동화를 지원.

# 젠킨스 vs github actions

- 젠킨스는 자체 서버를 설정하고 유지 관리해야 하지만, GITHUB ACTIONS는 GitHub 리포지터리와 통합되어 별도의 서버 설정이 필요 없다.
- 젠킨스는 다양한 플러그인을 통해 확장 가능하지만, GITHUB ACTIONS는 GitHub 생태계와의 통합이 더 원활하다.
- GITHUB ACTIONS는 YAML 파일을 사용하여 워크플로우를 정의하며, 젠킨스는 주로 Groovy DSL을 사용한다.


# github actions 예시
    //.github/workflows/work-1.yml

    name: work-1
    on:
        push:
            branches:
            - main
    jobs:
        hello1:
            runs-on: ubuntu-latest
            steps:
            - name: sayHi
                run: echo "Hi"
            - name: sayMyName
                run: echo "${{ github.actor }}"

- .github/workflows/work-1.yml 파일은 **main 브랜치에 커밋이 발생할 때마다 실행**
- 이 워크플로우는 두 개의 스텝으로 구성되어 있으며, 첫 번째 스텝은 "Hi"를 출력하고, 두 번째 스텝은 GitHub 사용자의 이름을 출력한다.


## job의 분리 

    name: work-1
    on:
        push:
            branches:
            - main
    jobs:
        hello1:
            runs-on: ubuntu-latest
            steps:
            - name: sayHi
                run: echo "Hi"
            - name: Wait for 10 seconds
                run: sleep 10  # 10초 대기
            - name: sayMyName
                run: echo "${{ github.actor }}"
        hello2:
            runs-on: ubuntu-latest
            steps:
            - name: sayHi
                run: echo "Hi"
            - name: Wait for 20 seconds
                run: sleep 20  # 20초 대기
            - name: sayMyName
                run: echo "${{ github.actor }}"

- 1개의 리포지터리안에 브랜치들, 워크플로우들이 들어갈 수 있다.

- 1개의 워크플로우 파일안에 발동조건이 걸려 있고 여러개의 job이 들어간다.
- 1개의 job 안에는 여러 개의 스텝이 존재한다.

## git tag란

- git 태그란 특정 커밋을 가리키는 참조 포인터로, 주로 릴리즈 버전을 표시하거나 중요한 지점을 기록하는 데 사용된다.

> 태그는 두 가지 유형이 있다:
1. 경량 태그: 단순히 특정 커밋을 가리키는 포인터입니다.
2. 주석 태그: 태그 이름, 이메일, 날짜 및 메시지를 포함하는 객체로 저장됩니다.

# github action 관련 사이트

- [공식 사이트](https://github.com/actions)
- [공식/비공식 actions 마켓 플레이스 ](https://github.com/marketplace?type=actions)
- [actions/checkout(공식 액션, 공식 액션은 actions/로 시작)](https://github.com/marketplace/actions/checkout)
- [mathieudutour/github-tag-action(비공식 액션)](https://github.com/marketplace/actions/github-tag)

# 릴리즈란

- 소프트웨어의 특정 버전을 배포하는 과정이다. 주로 새로운 기능 추가, 버그 수정, 성능 개선 등을 포함한다.

- 릴리즈는 태그를 기반으로 생성되며, 릴리즈 노트와 함께 배포된다.