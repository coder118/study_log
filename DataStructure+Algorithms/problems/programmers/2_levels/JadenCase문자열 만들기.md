1. [두번째 풀이](#34-두번째-풀이)

# 사고 과정

> 조건 생각

문자를 반환할 공간을 StringBuilder로 구성
stack을 이용해서 이전 문자열 값을 저장하고 비교할때 pop한다.

> 알고리즘 풀이

1. 문자열을 하나씩 반복한다.
2. 공백 다음에 나오는 문자열이 소문자일 경우
3. 공백 다음에 나오는 문자열이 대문자일 경우
4. 문자 중간에 나오는 대문자는 소문자로 바꿔줘야 한다.
5. 공백 다음에 나오는 문자열이 숫자일 경우

# 내 코드

<details>

<summary>눌러서 코드 확인</summary>

    import java.util.*;
    class Solution {
        public String solution(String s) {
            String answer = "";
            
            StringBuilder sb = new StringBuilder();
            ArrayDeque<Character> compare = new ArrayDeque<>();
            int len = s.length();
            for(int i =0; i<len; i++){
                
                char word = s.charAt(i);
                
                if(compare.isEmpty()){
                    //첫번째 값은 어떤 값도 존재하지 않는다.
                    if(Character.isLowerCase(word)){
                        sb.append(Character.toUpperCase(word));
                        compare.push(word);
                        continue;
                    }else{
                        compare.push(word);
                    }
                }
                
                //스택이 조건문을 비교할 수 있게 peek를 사용한다. pop하면 == 조건에 비교할 값이 없어진다.
                if(compare.peek()==' ' && Character.isLowerCase(word)){
                    sb.append(Character.toUpperCase(word));
                    compare.pop(); //안에 값을 비운다.
                    compare.push(word);
                }else if(compare.peek()!=' '&&Character.isUpperCase(word)){
                    //문자 중간에 대문자가 있으면 소문자로 바꾼다.
                    sb.append(Character.toLowerCase(word));
                    compare.pop();
                    compare.push(word);
                }else{
                    sb.append(word);
                    compare.pop();
                    compare.push(word);
                }
                
            }
            answer = sb.toString().trim();
            
            return answer;
        }
    }

</details>

## 막혔던 부분

nosuchelement가 나오는 부분이 있었다. 찾아보니 조건문에서 stack을 pop했을때 stack이 비워지고 그 다음 조건문에서 pop을 실행시켜서 발생하는 문제였다. 

코드를 짜면서도 너무 하드코딩 방식이라는 생각을 했다. 

# 다른 사람 풀이

    class Solution {
        public String solution(String s) {
            String answer = "";
            String[] arr = s.split(" ");
            for(int i = 0; i < arr.length; i++){
                // 추가한 if문
                if(arr[i].length() == 0){
                    answer += " ";
                }
                else{
                    answer += arr[i].substring(0, 1).toUpperCase();
                    answer += arr[i].substring(1).toLowerCase();
                    answer += " ";
                }
            }
            // 추가한 if문
            if(s.substring(s.length()-1, s.length()).equals(" ")) 
                return answer;
            return answer = answer.substring(0, answer.length()-1);
        }
    }


> 알고리즘 풀이 과정

조건으로 문자열을 하나하나 비교하지 않았다.
1. 문자열을 공백 기준으로 쪼갠다.
2. 여러개의 공백이 올 수 있으니 아무것도 존재하지 않는 배열이 있을 수 있다. 그 경우 문자열에 " "을 추가한다.
3. 문자열 단위로 쪼갰기 때문에 첫 문자는 항상 대문자로 바꾼다.
4. 나머지는 소문자로 만들고 공백을 추가한다.
5. 만약 입력값이 아무 문자열이 없는 공백일 경우 공백을 반환한다. 
6. 문자열을 반환한다.

## 피드백

너무 과하게 생각한다. 굳이 안써도 되는 stack을 사용함. 그리고 조건을 너무 과하게 만든다.

문제를 너무 복잡하게 생각하는 것 같다. 
이 문제의 경우 현재 문자가 단어의 첫 글자인지 판단하는 문제다.

첫 글자인지 알면 대문자로 바꾸고 나머지는 소문자로 바꾸면 된다. 


# 3.4 두번째 풀이

간결하게 생각하려 노력함

> 풀이 과정
공백문자로 분류하고 다시 붙여서 반환한다.
split을 사용하고 나중에 배열을 for문을 돌려 + 로 문자열을 붙인다.

1. 소문자가 첫문자일 경우 대문자로 바꾼다.

2. 첫 문자가 숫자나 대문자면 pass하면된다.

3. 공백이 연속으로 들어올경우 배열의 사이즈는 0이된다.


<details>

<summary>코드 정리</summary>


    String[] arr = s.split(" ");
    
    StringBuilder save = new StringBuilder();
    
        if(arr.length ==0){
            return " ";
        }
    
    for(String word: arr){
        char checking = word.charAt(0);
        if(Character.isLowerCase(checking)){
            char change = Character.toUpperCase(checking);
            
            String str = remain_str_toLower(word.substring(1,word.length()));
            
            save.append(change+str);
        }else{
            
            String str2 = remain_str_toLower(word.substring(1,word.length()));
            save.append(checking+str2);
            
        }
        save.append(" ");
        
    }

    save.deleteCharAt(save.length()-1);
    answer = save.toString();
    
    return answer;

    public static String remain_str_toLower(String str){
    
        String remain_str = str.toLowerCase();
        return remain_str;
        
    }

</details>


## 배운점
코드를 분리시키는 것이 좋다고 생각했는데 아니다 오히려 더 복잡해진다.
코드가 복잡해지면 차라리 다 지우고 새로 짜는게 맞출 확률이 높다.