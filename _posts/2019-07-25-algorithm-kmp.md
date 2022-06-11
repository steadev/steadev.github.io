---
title: "문자열 탐색 알고리즘 KMP"
author: steadev
date: 2019-07-25 17:24:00 +0900
categories: [Algorithm]
tags: [algorithm]
---


특별한 의미는 없고 Knuth, Morris, Pratt 이 세 사람이 만든 알고리즘이라 하여 KMP라고 하네요 ㅎㅎ 

| **A** | **B** | **A** | **C** | **A** | **B** | **A** | **B** |

| A | B | A | B |   |   |   |   |

만약 위와 같이 ABACABAB문장에서 ABAB를 찾는다 할때 보통 ABC를 한칸씩 오른쪽으로 옮기면서 비교하는 방법을 생각하기 쉽습니다. 하지만 그렇게하면 loop의 이중 중첩으로 문장이 매우 길어졌을 때 시간 복잡도가 (On2)이 되어 시간초과가 나곤 합니다.

그래서 나온것이 이 알고리즘! 

간단히 말하면, 찾고자하는 패턴의 반복되는 부분만큼(A)은 건너뛰어서 불필요한 검색(B)을 줄이는 것입니다. 

위의 예를 보면 앞의 AB와 뒤 AB가 겹치고 2자리까지 일치했습니다. 이때 겹치는 2칸만큼 건너띄워서 검색을 합니다.

| **A** | **B** | **A** | **C** | **A** | **B** | **A** | **B** |

| A | B | A | B |   |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
|   |   | A | B | A | B |   |   |
|   |   |   | A | B | A | B |   |
|   |   |   |   | A | B | A | B |

2번째에서 2칸이 건너띄워졌는데 왜 그런지 잘 이해가 가지 않을 수 있습니다. 하지만 아래 흐름을 따라오시면 자연스레 이해가 되실 겁니다 :)

먼저 빨간 A부분을 구해야겠죠?? 

여러 블로그에서 구하는 것에 대한 설명을 봤는데 정확한 설명이 많이 없고 그냥 앞뒤로 같은 것의 개수를 헤아리는 식의 설명이 많았습니다. 하지만 이렇게 하니 안되는 경우가 발생하더군요. 

| **0** | **A** | **0** |
| --- | --- | --- |
| 1 | AB | 0 |
| 2 | ABA | 1 |
| 3 | ABAB | 2 |

위 같은 경우는 상관 없지만, 아래처럼 AAAA인 경우는 헷갈립니다.

| **0** | **A** | **0** |
| --- | --- | --- |
| 1 | AA | 1 |
| 2 | AAA | 1?? |
| 3 | AAAA | 2?? |

물론.. 이렇게 얘기하는건 정답이 아니라는 거겠죠??

정확한 방법은 각 index의 prefix와 postfix를 비교하여 자신을 제외한 것 중 길이 최대인 것을 세는 것입니다.

ABAB에서,

| **index** | **prefix** | **postfix** | **p\[i\]** |
| --- | --- | --- | --- |
| 0 | A | A | 0 |
| 1 | A, AB | B, AB | 0 |
| 2 | A, AB, ABA | A, BA, ABA | 1 |
| 3 | A, AB, ABA, ABAB | B, AB, BAB, ABAB | 2 |

초록색은 자기 자신이라 제외합니다. 빨간색이 겹치는 것이고 그 길이가 답이 됩니다!

AAAA를 보면, 

| **index** | **prefix** | **postfix** | **p\[i\]** |
| --- | --- | --- | --- |
| 0 | A | A | 0 |
| 1 | A, AA | A, AA | 1 |
| 2 | A, AA, AAA | A, AA, AAA | 2 |
| 3 | A, AA, AAA, AAAA | A, AA, AAA, AAAA | 3 |

겹치는(빨간색) 것 중 제일 긴 것으로 작성하면 오른쪽과 같은 결과가 나옵니다.

이를 통해서 얼마나 건너뛸지를 알 수 있습니다.

저 숫자만큼의 prefix와 postfix가 같다는 것이기에 그만큼 움직일 수 있는 것입니다.

제일 처음의 예를 다시 보면,

| **A** | **B** | **A** | **C** | **A** | **B** | **A** | **B** |
| --- | --- | --- | --- | --- | --- | --- | --- |

| A | B | A | B |   |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
|   |   | A | B | A | B |   |   |

index 2까지가 일치했으니 p\[2\] = 1인것을 활용하면 됩니다.

이 말은 즉, 접두사와 접미사가 1개만 일치한다는 것이므로 2칸 이동하여

빨간 부분처럼 접두사와 접미사 겹치는 부분이 1개가 되게끔 해주는 것입니다.

그러면 이를 바탕으로 다른 문장에서 패턴 찾는 연습을 해보겠습니다. 

문장  :  ABABCABABA

패턴  :  ABABA

우선, p\[i\]값을 찾아야겠죠??

위에처럼 시각적으로 찾는 방법 말고 프로그래밍적으로 하면, 패턴을 옆으로 옮기며 prefix와 postfix를 비교하면 됩니다.

index == 0 일때, 자기자신뿐이기에 p\[0\] = 0,

index == 1 일때, 

| **0** | **1** | **2** | **3** | **4** |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | A |   |   |   |
|   | A | B | A | B | A |   |   |

AB

  AB

이렇게 보면 접두, 접미사가 일치하지 않으니 p\[1\] = 0

index == 2 일때,

| **0** | **1** | **2** | **3** | **4** |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | A |   |   |   |
|   |   | A | B | A | B | A |   |

하나가 일치하죠? 고로 p\[2\] = 1

| **0** | **1** | **2** | **3** | **4** |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | A |   |   |   |
|   |   | A | B | A | B | A |   |

두개 일치! p\[3\] = 2

| **0** | **1** | **2** | **3** | **4** |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | A |   |   |   |
|   |   | A | B | A | B | A |   |

마지막으로 p\[4\] = 3 

정리하면 **p\[i\] = { 0, 0, 1, 2, 3 }** 입니다. 

여기서는 index가 1일때만 불일치가 나왔고 나머지는 다 일치가 나왔지만, 만약 뒤에서 불일치가 나왔다 한다면,

이전의 p\[i\] 정보를 이용해서 건너뛸 수도 있습니다. 

만약 ABABABC

          ABABABC

이런 상황이라면 p\[3\]을 활용하여 두칸 뒤로 옮겨서 불필요한 비교를 피할 수 있다는 것입니다.

아래에서 보일, 패턴 찾는 것을 p\[i\]값 찾는데서도 활용할 수 있다는 얘기입니다 :)

그러면 이제 패턴을 찾아보겠습니다.

| **0** | **1** | **2** | **3** | **4** | **5** | **6** | **7** | **8** | **9** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| A | B | A | B | C | A | B | A | B | A |
| A | B | A | B | A |   |   |   |   |   |
|   |   | A | B | A | B | A |   |   |   |
|   |   |   |   | A | B | A | B | A |   |
|   |   |   |   |   | A | B | A | B | A |

첫번째는 3번째까지 겹치니 p\[3\] = 2 를 통해 2칸 이동,

두번째는 p\[1\] = 0 이기에 다음 index쪽으로 바로 이동,

세번째는 0번째부터 일치하지 않으니 바로 다음칸으로 이동합니다.

위 과정을 코드로 작성해 봤습니다.

```c++
#include <iostream>
#include <vector>
#include <string>
using namespace std;
 
// 패턴의 중복되는 부분(수치) 찾기
vector<int> getPi(string pattern) {
    // 모두 0으로 세팅(중복되는 거 모두 없다고 우선 세팅)
    vector<int> p(pattern.length(), 0);
    int correct_cnt = 0;
    for (int i = 1; i < pattern.length(); i++) {
        // 해당 부분이 일치할 시에 일치하는 숫자(correct_cnt)만 증가시키고 p[i]값 set(마지막부분)
        if (pattern[i] == pattern[correct_cnt]) correct_cnt++;
        else {
            // 불일치할 경우 중복되는 만큼(p[correct_cnt-1]의 수치) correct_cnt를 변경 
            // correct_cnt는 1부터, p[i]는 0부터이므로 -1을 해준다
            // 그리고 i-1 하여 다시한번 해당 index를 반복한다.
            if (correct_cnt > 0) {
                correct_cnt = p[correct_cnt - 1];
                i--;
                continue;
            }
            // 만약 correct_cnt가 0이라면 일치하는게 없는 것이므로 그대로 p[i]에 세팅해준다.
        }
        p[i] = correct_cnt;
    }
    return p;
}
 
// KMP 알고리즘
int kmp(vector<int> p, string pattern, string sentence) {
    int correct_cnt = 0;
    int pattern_len = pattern.length();
    for (int i = 0; i < sentence.length(); i++) {
        if (sentence[i] == pattern[correct_cnt]) correct_cnt++;
        else {
            if (correct_cnt > 0) {
                correct_cnt = p[correct_cnt - 1];
                i--;
            }
        }
        // 
        if (correct_cnt == pattern_len) return i;
    }
    return -1;
}
 
int main(void) {
    string sentence;
    string pattern;
 
    // 띄어쓰기가 있을 경우를 대비해서 getline 사용!
    getline(cin, sentence);
    getline(cin, pattern);
 
    vector<int> p;
 
    p = getPi(pattern);
    
    cout << kmp(p, pattern, sentence);
    
    return 0;
}
```

질문이나 오류 지적 사항등은 댓글 부탁드립니다 :)

참고) 왜모르는가?([https://vvshinevv.tistory.com/2](https://vvshinevv.tistory.com/2))