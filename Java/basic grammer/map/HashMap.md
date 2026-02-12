# 26.1.29

HashMap의 간단한 사용법
map은 키-값 쌍(key-value pair)으로 데이터를 저장하는 컬렉션이다. 
키는 고유해야 하며, 값은 중복을 허용한다.

HashMap,TreeMap,LinkedHashMap등이 대표적인 Map 구현체이다. 

    import java.util.HashMap;
    HashMap<String, Integer> map = new HashMap<>();

    //put 함수
    map.put("apple", 5);     // Key: "apple", Value: 5
    map.put("banana", 10);   // Key: "banana", Value: 10
    map.put("cherry", 7); 

    //get 함수
    System.out.println(map.get("apple"));  // 5
    System.out.println(map.get("pear"));   // null (존재하지 않는 키)

    //containKey함수
    if (map.containsKey("banana")) {
        System.out.println("Value: " + map.get("banana"));
    }

https://sxbxn.tistory.com/64

더 자세한 내용은 HashMap에 대한 따로 내용을 만들어야 겠다.
https://orange-makiyato.tistory.com/117
