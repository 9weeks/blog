---
title: react native - SafeAreaView cookbook
date: "2020-10-03T05:25:03.284Z"
description: "Tips for SafeAreaView"
---

오늘은 SafeAreaView를 조금 더 잘 활용하는 방법에 대해서 공유하고자 한다.

# 해결사 SafeAreaView

Notch(노치)는 iOS, Android에 상관없는 특징으로 이로 인해 layout이 각각 다르게 보이는 문제가 있다. 이는 SafeAreaView가 제공되기 전에는 다소 귀찮게 해결해야 했으나, 지금은 아주 쉽게 해결할 수 있게 되었다.

## 과거에는…

2017년에 iPhone X가 나오면서, Notch(노치) 디자인으로 인한, 전체 height를 구하기는게 약간 까다로워한졌다. 또한, Home bar로 인한 사용할 수 없는 하단 영역도 생기게 되었다. 초기에는 이를 해결하기 위한 react-native-phone-x-helper 와 같은 도구를 사용했으나, RN(react-native)의 Platform을 이용한 간단한 library에 지나지 않았다.

## 현재는…

SafeAreaView를 공식적으로 제공하므로, SafrAreaView만 사용한다면 사실 전혀 문제가 없다. 어떤 기기든지(물론 미래의 기기는 아무도 모른다) SafeAreaView안에만 제공하면 크게 화면을 벗어나는 문제는 없다. 모든 화면이 SafeAreaView일 필요는 없으므로, 최상단 화면만 여기에 감싸면 된다.

```
const App = () => (
    <SafeAreaView style={{flex: 1}}>
        <MyCustomRoute />
    </SafeAreaView>
)
```

이렇게 작성하면 모든 Routing을 통해 보여지는 페이지는 SafeArea 안에서 작동하게 된다.

다만, 몇 가지 문제가 있는데…

# Home bar’s Background

iOS에서 위의 앱을 살펴보면, Home bar 주변에는 아무 영향을 미치지 못함을 알 수 있다. 이 자체로는 크게 문제가 되지 않는다. 하지만, Slide up animation을 구현하는 경우 화면 아래에서 transition이 일어나게 되면 Home bar의 배경 화면에 그대로 노출된다는 점이다. 이러한 경우를 위해서, SafeAreaView가 이후의 영역에 대한 정보도 같이 주어야 한다.

const App = () => (
    <>
        <SafeAreaView style={{flex: 1}}>
            <MyCustomRoute />
        </SafeAreaView>
        <SafeAreaView style={{flex: 0, backgroundColor: 'white'}}>
        </SafeAreaView>
    </>
)
그렇다. flex: 0이라는 점이다. 무척 의미없어 보이는 구문이지만, 이로 인해 Home bar의 배경 화면이 transparent(투명)이 아닌 흰색이 되며, transition animation에서 군더더기 없는 모습을 보일 수 있다. 여담이지만, <SafeAreaView>가 2개가 되어 단일 JSX 반환이 안되므로 <>으로 감싸 Fragment 처리하였다.

## Status bar

Notch로 인한 Status bar를 관리하기 위해서는 RN의 StatusBar를 이용하면 된다. 여기에 같이 언급하는 이유는 다음 섹션 때문이다.

## Custom SafeAreaView

예제는 단순하게 표현하기 위해서 Routing을 감싸는 형태로 작성했지만, 현실은 전혀 그렇지가 않다. 개별 Screen에서 Safe하게 그려져야 할지 혹은 전체 화면 형태로 그려져야 할지 선택하는 편이 오히려 더 효율적이다. 따라서, SafeAreaView를 customization한 Component로 관리하는 것이 적합하다.

간략히 표현해 보자면,

```
const Content = ({chidren}) => {
    const { color: {backgroundColor}, isNightMode } = useContext(Theme);
    const barStyle = isNightMode ? 'dark-content' : 'light-content';
    return (
        <>
            <StatusBar barStyle={barStyle}>
            <SafeAreaView style={{flex: 1, backgroundColor}}>
                {children}
            </SafeAreaView>
            <SafeAreaView style={{flex: 0, backgroundColor}}>
            </SafeAreaView>
        </>
    )
};
```

이러한 형태로 Content라는 Component로 처리하는 것이 App의 관리 차원에서 더욱 효율적이다.

# Conclusion

RN의 이미 엄청난 편의를 제공한다. SafeAreaView는 필수적이며, 기존의 View에서 SafeAreaView로 이름만 바꾸면 의미 없는 코드들을 쉽게 제거할 수 있다.