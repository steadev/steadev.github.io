---
title: "[카카오 코딩 테스트] 호텔 방 배정"
author: steadev
date: 2020-05-07 15:12:00 +0900
categories: [Algorithm]
tags: [Kakao, algorithm]
---

[호텔 방 배정 문제 보기](https://tech.kakao.com/2020/04/01/2019-internship-test/)

4번 호텔 방 배정 문제입니다.
구체적인 원리 및 풀이 과정은 위 링크의 해설에 그림과 함께 자세하게 설명이 되어 있습니다. 
 
간단하게 요약하자면,
붙어있는 방 끼리 한 그룹이 되고 그 그룹들은 모두 다음의 빈방을 가리키고 있으면 
다음에는 그 그룹이 가리키는 빈방을 바로 배정하면 되는 것입니다. 
 
이는 결국 Union-Find 문제입니다. 
어렵게 생각하면 코드가 엄청 길어질 수 있지만, 이 알고리즘을 활용하면 아래와 같이 
알고리즘 부분이 20줄도 채 안될만큼 간단한 문제입니다.
 
아래는 구현한 코드입니다.

```c++
#include <iostream>
#include <vector>
#include <map>
using namespace std;
 
map<long long, long long> room;
 
long long getEmpty(long long num) {
    if (room[num] == 0) return num;
    return room[num] = getEmpty(room[num]);
}
 
vector<long long> solution(long long k, vector<long long> room_number) {
    vector<long long> answer;
    for (auto num : room_number) {
        long long empty = getEmpty(num);
        answer.push_back(empty);
        room[empty] = empty + 1;
    }
    return answer;
}
int main() {
    long long k = 10;
    vector<long long> room_number = { 1,3,4,1,3,1 };
    vector<long long> result = solution(k, room_number);
    for (auto x : result) {
        cout << x << ' ';
    }
    return 0;
}
```
질문이나 지적사항은 댓글 부탁드립니다 :)