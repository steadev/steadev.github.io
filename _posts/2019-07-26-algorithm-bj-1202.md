---
title: "[백준] 1202번 보석도둑"
author: steadev
date: 2019-07-26 13:05:00 +0900
categories: [Algorithm]
tags: [Baekjoon, algorithm]
---


백준 알고리즘 1202번 보석도둑 문제풀이

# 문제

각 보석은 무게 M와 가격 V를 가지고 있다.

가방이 K개 있고, 각 가방에 담을 수 있는 최대 무게는 C이다. 가방에는 최대 한 개의 보석만 넣을 수 있다.

훔칠 수 있는 보석의 최대 가격을 구하는 프로그램을 작성하시오.

# 풀이

복잡하게 생각할수록 어려워지는 문제인것 같습니다. 

풀이 방향은 맞췄지만 너무 복잡하여 예외 처리가 되질 않아 자꾸 틀렸습니다.. ㅠㅠ

꾸준함[(https://jaimemin.tistory.com/760)](https://jaimemin.tistory.com/760)님의 블로그를 참조하여 코드를 단순하고 깔끔하게 수정하였습니다.

풀이 방식의 핵심은 Max-Heap을 사용하는 것입니다. 

해당 가방의 무게만큼 heap에 넣고 최대값만 쏙쏙 빼내는 방식입니다.

1\. 가방과 보석을 모두 무게를 기준으로 오름차순 정렬합니다.

2\. i번째 가방의 무게에 맞는 만큼 Priority Queue에 보석 가격들을 넣어줍니다.

3\. PQ의 top에 있는 것이 가장 높은 가격이기에 결과 값에 top의 값을 넣어주고 pop 합니다.

4\. 가방의 개수(k) 만큼 반복합니다. 

아래는 풀이한 코드입니다 !! 

```c++
#include <iostream>
#include <algorithm>
#include <queue>
#include <vector>
using namespace std;
 
int main(void) {
    int n, k;
    cin >> n >> k;
    vector<int> bag(k);
    vector<pair<int, int> > jewel(n);
    long long max_jewel = 0LL;
    priority_queue<int> pq;
    
    for (int i = 0; i < n; i++) {
        int m, v;
        cin >> m >> v;
        jewel[i] = make_pair(m, v);
    }
    
    for (int i = 0; i < k; i++) cin >> bag[i];
    
    // 오름차순 정렬
    sort(jewel.begin(), jewel.end());
    sort(bag.begin(), bag.end());
    
    for (int i = 0, jewel_idx = 0; i < k; i++) {
        while (jewel_idx < n && jewel[jewel_idx].first <= bag[i]) {
            // 가방의 무게보다 작거나 같은 보석들을 pq에 넣어줍니다.
            pq.push(jewel[jewel_idx++].second);
        }
        
        if (!pq.empty()) {
            // 젤 위의 값(최대값)을 넣어주고 pq에서 빼버립니다.
            max_jewel += (long long)pq.top();
            pq.pop();
        }
    }
    cout << max_jewel;
    return 0;
}
```

질문이나 지적사항 댓글 부탁드립니다 :)