---
title: "[카카오 코딩 테스트] 크레인 인형 뽑기 게임"
author: steadev
date: 2020-05-06 14:39:00 +0900
categories: [Algorithm]
tags: [Kakao, algorithm]
---

[크레인 인형 뽑기 게임 문제 보기](https://tech.kakao.com/2020/04/01/2019-internship-test/)
 
1번 크레인 인형 뽑기 게임 문제입니다.
 
1번 문제 답게 stack 활용하면 쉽게 해결할 수 있는 문제입니다.
stack의 top과 비교해서 같으면 2씩 더해주기만 하면 끝나는 문제!
 
아래는 풀이한 코드 입니다.
 
```c++
#include <iostream>
#include <vector>
#include <stack>
using namespace std;
int solution(vector<vector<int>> board, vector<int> moves) {
    int answer = 0;
    stack<int> s;
    for (int i = 0; i < moves.size(); i++) {
        int col = moves[i]-1;
        for (int j = 0; j < board.size(); j++) {
            if (board[j][col] != 0) {
                int cur = board[j][col];
                board[j][col] = 0;
                // 스택이 비었거나 상단의 인형과 뽑은 인형이 다를 경우 
                if (s.empty() || s.top() != cur)
                    s.push(cur);
                // 인형이 같을 경우
                else {
                    answer += 2;
                    s.pop();
                }
                break;
            }
        }
    }
    return answer;
}
int main() {
    vector<vector<int>> board = { 
        {0, 0, 0, 0, 0},
        {0, 0, 1, 0, 3},
        {0, 2, 5, 0, 1},
        {4, 2, 4, 4, 2},
        {3, 5, 1, 3, 1} 
    };
    vector<int> moves = { 1,5,3,5,1,2,1,4 };
    cout << solution(board, moves);
    system("pause");
    return 0;
}
```

질문이나 지적사항은 댓글 부탁드립니다:)