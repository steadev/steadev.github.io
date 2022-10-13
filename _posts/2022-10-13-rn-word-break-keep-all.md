---
title: "[React Native] word-break"
author: steadev
date: 2022-10-13 18:10:00 +0900
categories: [React Native]
tags: [React Native, React Native Style]
---

## word-break와 같은 부분이 적용이 안되는 이슈!

RN에는 CSS와 조금 다르게 스타일이 적용되는 부분들이 있는데요..!

그 중에서 `word-break: keep-all`을 적용하고 싶은데 안되는 이슈가... 왜 이것도 지원 안해주지? 하는 의문이 들었습니다.

아직 RN 버전이 0.대라 부족한 부분이 많은 것 같긴 합니다. 버전 1.대까지 간다면 생기지 않을까 싶습니다..!

하지만, 뭐든 방법은 있는법..!

string을 띄어쓰기를 기준으로 나눈 뒤, `flexWrap` 속성을 활용하면 됩니다.

이 속성은 나열된 요소들의 총 너비가 부모보다 크면 다음 줄에 이어서 나열하도록 합니다.

이를 바탕으로 컴포넌트를 만들어봤습니다.

```typescript
import React from "react";
import { StyleSheet, Text, TextStyle, View } from "react-native";

const WordBreakKeepAllText = ({
  text,
  textStyles,
}: {
  text: string;
  textStyles?: TextStyle;
}) => {
  return (
    <View style={styles.container}>
      {text.split(" ").map((word, index) => (
        <Text key={`${word}-${index}`} style={textStyles}>
          {word}{" "}
        </Text>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    display: "flex",
    flexDirection: "row",
    flexWrap: "wrap",
  },
});

export default WordBreakKeepAllText;
```

### 결과 비교

위는 그냥 `<Text>blabla</Text>`의 결과이고

아래는 위 컴포넌트 적용했을 때의 결과입니다.

<img src="https://steadev.github.io/assets/images/rn/2022-10-13-1.png" />
