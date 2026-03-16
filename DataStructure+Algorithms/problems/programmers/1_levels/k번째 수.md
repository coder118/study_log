
[문제로](https://school.programmers.co.kr/learn/courses/30/lessons/42748)

# 문제정의

1. 배열 내부의 i,j,k를 구해야 한다.
2. Arrays.sort를 이용해서 배열을 정렬해야 한다.
3. i,j,k에서 -1을 해서 인덱스 위치를 맞춰줘야 한다.

# 코드

    int len = commands.length;
    int[] answer = new int[len];

    for(int ch = 0; ch<len; ch++){
        
        int i = commands[ch][0]-1;
        int j = commands[ch][1]-1;
        int k = commands[ch][2]-1;
        
        List<Integer> temp = new ArrayList<>();
        for(int index = i; index<=j; index++){
            if(index<array.length){
                temp.add(Integer.valueOf(array[index]));
            
                }
            }
        // temp를 배열로 변환하고 정렬
        Integer[] tempArray = temp.toArray(new Integer[0]); // 배열로 변환
        Arrays.sort(tempArray); // 정렬
        answer[ch] = tempArray[k]; // k번째 요소를 answer에 저장    
               
    }

i,j,k를 구하는 것은 문제강 없었다. 문제는 array에 있는 배열에 i,j 범위의 인덱스로 접근해서 배열을 만들어야 한다는 건데 배열을 만드는 과정이 쉽지 않았다. 그래서 리스트를 사용해서 array의 값을 Integer로 바꿔서 넣어주었다. 
<br>
리스트에 들어간 값을 배열로 정리시키고 sort를 실행했다. 이후 ansewer에 값을 넣어서 해결했다.

# 피드백

정답을 맞추는 것에 성공했지만 굳이 리스트를 만들고 그걸 다시 배열로 만드는 작업은 메모리와 시간 복잡도 측면에서 낭비가 심하다.

## 다른 사람 코드
    
    for(int i=0; i<commands.length; i++){
        int[] temp = Arrays.copyOfRange(array, commands[i][0]-1, commands[i][1]);
        Arrays.sort(temp);
        answer[i] = temp[commands[i][2]-1];
    }

copyOfRange라는 것을 사용해서 array 배열 내부 i와 j의 범위를 정하고 temp에 복사했다. 내가 리스트를 만들고 그것을 다시 배열로 만드는 작업을 생략할 수 있는 방법이다. 이렇게 간단하고 가독성이 좋은 코드를 참고 해야 겠다.
 