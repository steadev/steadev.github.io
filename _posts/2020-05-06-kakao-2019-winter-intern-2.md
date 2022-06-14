---
title: "[카카오 코딩 테스트] 튜플"
author: steadev
date: 2020-05-06 15:07:00 +0900
categories: [Algorithm]
tags: [Kakao, algorithm]
---

[튜플 문제 보기](https://tech.kakao.com/2020/04/01/2019-internship-test/)
 
2번 튜플 문제입니다.
 
원리만 알면 쉽게 풀 수 있는 문제입니다. 
튜플들이 순서가 바뀔수 있고 집합마다 원소 순서도 다 바뀔 수 있지만, 걱정할 필요가 없습니다.
집합이 4개라면, 결국 제일 첫번째 있는 원소는 4번, 두번째 원소는 3번, 세번째는 2번, 네번째는 1번 나올 수 밖에 없습니다.
그래서 string에서 숫자를 추출해내고 각 숫자가 몇번 반복됬는지만 확인해 주어 정렬만 해준다면 쉽게 해결할 수 있는 문제입니다.
 
아래는 풀이한 코드입니다.

```c++
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
int cnt[100001] = { 0 };
vector<int> solution(string s) {
    vector<int> answer;
    int num = 0;
    for (int i = 0; i < s.length(); i++) {
        if (s[i] != '{' && s[i] != '}' && s[i] != ',') {
            num *= 10;
            num += s[i] - '0';
        }
        else {
            if (num > 0) {
                cnt[num]++;
                num = 0;
            }
        }
    }
    vector<pair<int, int>> result;
    for (int i = 0; i < 100001; i++) {
        if (cnt[i] > 0) {
            result.push_back({ cnt[i],i });
        }
    }
    sort(result.begin(), result.end());
    for (int i = result.size() - 1; i >= 0; i--)
        answer.push_back(result[i].second);
    return answer;
}
int main() {
    {% raw %}// string s = "{{2},{2,1},{2,1,3},{2,1,3,4}}"{% endraw %};
    {% raw %}// string s = "{ {1, 2, 3}, { 2,1 }, { 1,2,4,3 }, { 2 }}";{% endraw %} 
    {% raw %}// string s = "{{20,111},{111}}"{% endraw %};
    {% raw %}// string s = "{{123}}"{% endraw %};
    string s = {% raw %}"{{4,2,3},{3},{2,3,4,1},{2,3}}"{% endraw %};
    vector<int> result = solution(s);
    for (auto x : result)
        cout << x << " ";
    system("pause");
    return 0;
}
```
질문이나 지적사항은 댓글 부탁드립니다 :)