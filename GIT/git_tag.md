
# 사용방법

만약 배포를 한 커밋이 있다고 할때, 그 커밋의 내용을 좀 더 명확하게 표시해두기 위해서 태그라는 기능을 사용할 수 있다.

    git tag -a <태그이름> -m <추가메시지>

    git tag -a step-01-complete -m "Step 01 complete" # 태그 생성
    git tag # 태그 목록 조회
    git show step-01-complete # 특정 태그 상세 정보
    git push origin step-01-complete # 태그 원격 저장소에 푸시



