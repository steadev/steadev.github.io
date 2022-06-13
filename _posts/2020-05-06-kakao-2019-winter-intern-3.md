---
title: "[카카오 코딩 테스트] 징검다리 건너기"
author: steadev
date: 2020-05-06 19:42:00 +0900
categories: [Algorithm]
tags: [Kakao, algorithm]
---

[징검다리 건너기](https://tech.kakao.com/2020/04/01/2019-internship-test/)
 
5번 징검다리 건너기 문제입니다.
 
이 문제는 그냥 다 해보면 되지만, 효율성 점수까지 있기 때문에 잘 생각해서 풀어야 합니다. 
풀이 방법부터 말하자면, 이 문제는 전형적인 binary search 문제입니다. 
몇명이 건너는지 미리 중간값으로 정해놓고 조건에 만족하는지 체크 후, 값을 올리든 내리든 하면 끝입니다.
 
무슨 알고리즘을 써야하는지 알면 순식간에 풀 수 있는 문제지만, 문제를 보고 바이너리 서치를 떠올리기엔 생각보다 쉽지 않았습니다.. 역시 많은 문제를 풀어봐야 감이 올듯 싶은 부분이네요!
 
코드 작성 후 처음에는 제 컴퓨터에선 잘 되지만, 프로그래머스에서는 메모리 에러가 났습니다.
재귀 방식으로 구현했었는데, 함수 스택이 계속 쌓이면서 stones 변수가 차지하는 공간이 늘어나고 해당 에러가 난 것 같습니다. 아래처럼 그냥 반복문으로 하면 별 문제없이 클리어!
 
아래는 구현한 코드입니다.

```c++
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#define MAX 200000000
using namespace std;
int solution(vector<int> stones, int k) {
    int answer = 0;
    int from = 1; 
    int to = MAX;
    while (from <= to) {
        int mid = from + ((to - from) / 2);
        int seq = 0;
        bool flag = true;
        for (int i = 0; i < stones.size(); i++) {
            if ((stones[i] - mid) < 0)
                seq++;
            else
                seq = 0;
            if (seq == k) {
                flag = false;
                break;
            }
        }
        if (flag) {
            answer = max(answer, mid);
            from = mid + 1;
        }
        else {
            to = mid - 1;
        }
    }
    return answer;
}
int main() {
    vector<int> stones = { 2, 4, 5, 3, 2, 1, 4, 2, 5, 1 };
    int k = 3;
    cout << solution(stones, k);
    return 0;
}
```

질문이나 지적사항은 댓글 부탁드립니다 :)