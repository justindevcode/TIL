# 2023/10/10

자바 해쉬맵 키값이 있으면 등록하면서 value값 화살표 함수로 넣는방식 
hashMap.computeIfAbsent(name, k -> 0);
이 코드를 사용해서 name의 값이 해쉬에 존재하면 아무것도 안하고
해쉬안에 값이 존재하지 않으면 그 name을 key값으로 추가하며 value값으로 화살표함수 0을 추가하는 코드

