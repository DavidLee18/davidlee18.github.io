---
title: 'The GospelSub Journey: In Search of The Master Programming Language'
layout: post
date: '2022-04-13 01:12:43'
categories:
- functional-programming
---

이것은 GospelSub 이라는 프로그램을 개발하고 유지보수, 수정해온 나의 일화이며, 동시에 문제에 접근하고 해결하면서 나의 사고방식과 문제해결 방식의 변화를 담았고, 그 때문에 아마도 변한 나의 사고방식을 이해하고, 어쩌면 비슷한 경험을 겪는 이들이 이를 보고 도움을 얻지 않을까 하는, 일말의 기대도 있다.
각설하고 처음부터 설명하겠다. _다만 주의할 점은, 소스 코드가 1.0 버전부터 남아있는 것은 아니라(3.0부터 남음) 다소 코드의 시점이 정확하지 않고 나의 기억에 의존하는 점이 있다._

## 태동
당시 나는 성락교회 대학부의 인천 지역 쪽의 학생들이 모인 모임에서 있었다. 거기서는 예배 시간에 찬양을 같이 부르는데, 그때 노래의 가사를 컴퓨터와 연결한 빔 프로젝터에 보여준다. 이는 보통 하얀 화면에 검은 글씨로 보여준다. 

이를 보통 파워포인트 프레젠테이션(PPT)을 이용해서 작업하는데, 이 방식은 편하지만 몇 가지 문제점이 있었다.
- 노래를 몇 가지 부분으로 나누어서 A-A-B-C-A-C 와 같이 흐름을 만드는데, 이 흐름에 맞추어 화면을 만들면 이후에 노래 연습을 하다가 **흐름이 변경되면 굉장히 많은 부분**을 수정해야 했다.
- 같은 노래를 다시 부를 때, 인도자와 다른 사람이 인도하는 경우 흐름이 바뀌는 일이 생기는데, 그럴 때에 유연한 대처가 불가능했다. 프리젠테이션을 조작하는 사람 입장에서도, 프리젠테이션을 보는 사람들 입장에서도, 굉장히 불편한 경험이었다.
- 글자의 크기가 안 맞는 경우가 있는데, 예를 들어 어떤 부분에서는 글자가 너무 많아서 크기가 작아야 하고, 어떤 부분에서는 글자 크기가 커야 한다. 그리고 한 줄에는 _적당한 양의 글자만_ 있어야 하고, 글자들은 중앙 정렬이 되어 있어야 한다. 또한 멀리서도 볼 수 있도록 화면 위쪽에 자리하는 것이 좋다. 그런데 프리젠테이션으로 이런 것들을 다 설정하기에는 **귀찮고 번거로웠다.** 어떨 때는 이미 프리젠테이션 하고 있는 동안에 수정해야 하는 경우도 있었는데, **이럴 때는 수정 자체가 불가능했다.**
- 이전에 썼던 곡을 다음에 쓰는 것도 _번거롭고 상당히 불편해서_ 보통은 처음부터 다시 만드는 방식을 택했다. 다시 말해 **재사용이 굉장히 힘들었다.**

이와 같은 이유 때문에, 마침 그때 C#을 알고 있던 나는, '이걸 GUI 프로그램으로 만들어 보면 어떨까' 하는 생각을 하게 되었다. 기본 아이디어는 이와 같았다.
- 찬양들은 텍스트로 기록하여 파일 단위로 저장한다.
- 프로그램으로 그 텍스트 파일들을 그때그때 불러와서 노래의 순서를 그때그때 설정한다.

간단히 말하자면 **이미 타자로 친 노래의 내용을 재활용하자는 것이었고** 이것이 바로 GospelSub의 시작이다.

## GospelSub 1.0 in C#

C#은 이 프로그램을 개발하는 데 있어서 굉장히 도움이 된 언어였다. C#으로는 Windows에서 GUI 프로그램을 만드는 일이 쉬웠기 때문이다. 가장 간단하게는 WinForms, 조금 더 섬세한 애니메이션과 조정을 원한다면 WPF(Windows Presentation Foundation)이 있었다. 나는 이것들을 활용해서 프로그램을 만들기로 했다.

처음에 구상한 찬양의 데이터 모델은 노래의 이름과 절 단위로 나눠진 가사들의 배열을 포함하는 것이었다.
```csharp
class Gospel
{
    public string Name { get; }
    public List<string> Lyrics { get; }
}
```

처음 생각으로는 노래 파일들을 읽어와서 리스트로 보여주고, 그 중에 몇 개를 선택한 다음, 순서를 지정하고, 화면으로 띄워주는 방식이었다.
```csharp
class Setting
{
    public List<Gospel> Gospels { get; set; } = new List<Gospel>();
    public List<Gospel> Sequence { get; set; } = new List<Gospel>();
}
```
이런 기본적인 토대가 잡히자, 가능성은 무궁무진해졌다. 보통은 하얀 화면에 검은 글씨로 보이던 것을 검은 화면에 하얀 글씨로 보이게 하면 어두운 곳에서 상대적으로 더 잘 볼 수 있었다. 이것은 색깔만 바꿔주면 되기에 GospelSub에서 적용하기에 매우 간단했다. 화면에 보이는 글씨체를 설정하는 것도 간단했고, 글씨의 크기를 조절하는 것도 프리젠테이션 도중에 얼마든지 가능했다. 배경은 보통 단색으로 설정하지만, 원한다면 이미지를 불러와서 배경으로 설정하는 것도 식은죽 먹기였다. 내가 꿈꾼 그대로 이룬 것에 아주 기분이 좋았고 새로운 가능성을 열은 기분이었다.

## GospelSub 3.0까지
그와 함께 코드의 확장도 일어났다. 우선 `Gospel` 은 `string` 타입의 이름과 `List<string>` 타입의 가사들로 이루어져 있으므로 그 자체를 `List<string>`으로 다룰 수도 있었다. 이것을 쉽게 하기 위해 [인덱서](https://en.wikipedia.org/wiki/Indexer_(programming))와 `Count`를 도입했다. 또한 사용자가 선택한 찬양들을 구별하기 위해 `bool` 타입의 플래그를 넣었다. 마지막으로, 찬양의 재사용성을 더 확장해서 한 노래도 여러 부분으로 쪼개서 특정 키를 누르면 그 부분으로 바로 이동하도록 하는 "Tag"를 구상했다. 예를 들어 기존에는 이와 같이 구성되어 있던 노래를
```
-sing sing sing-

Sing sing sing          --------------------
천국에 울리는 노래                              |
                                            |
We will sing sing sing                     |
주님 들으시네                                 C
                                           |
감사 드리며                                   |
예수 이름 높이세           --------------------

사랑스러운 주님            --------------------
땅과 하늘 찬양해                              |
                                          |
열방이 경배해                                 A
예수님 당신은 주                              |
                                          |
유일하신 삶의 이유          -------------------

자유케하는 사랑            --------------------
타오르는 불처럼                               |
                                          |
인도하시는 빛                                B
예수님 당신은 주                              |
                                          |
유일하신 삶의 이유          -------------------

Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세

자유케하는 사랑 
타오르는 불처럼

인도하시는 빛 
예수님 당신은 주 

유일하신 삶의 이유

Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세

Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세

Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세

Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세.
```
C-A-B-C-B-C-C-C-C 와 같이 되어 있는 기존의 노래에서 반복되는 부분을 단순화 함과 동시에 단축키를 추가해서 바로 갈 수 있도록
```
-Sing Sing Sing(Tagged)-

[h]Sing sing sing
천국에 울리는 노래

We will sing sing sing
주님 들으시네

감사 드리며 
예수 이름 높이세

[1]사랑스러운 주님 
땅과 하늘 찬양해

열방이 경배해 
예수님 당신은 주

유일하신 삶의 이유

[2]자유케하는 사랑 
타오르는 불처럼

인도하시는 빛 
예수님 당신은 주 

유일하신 삶의 이유
```
위와 같이 바꾸는 것이 태그 적용이다. 이렇게 함으로써 노래의 재활용성을 훨씬 높였다. 노래에 절이 표시되어 있는 경우는 자동으로 단축키가 등록되도록 했다. 이렇게 하기 위해서 `Gospel`에 태그의 단축키와 그 단축키를 누르면 이동하는 위치를 기록하는 데이터를 추가했다.
```csharp
class Gospel
{
    // ....
    public Dictionary<Key, int> Tags { get; } = new Dictionary<Key, int>();
}
```
그리고 태그는 명시적인 경우와 암시적인 경우의 두 가지 타입이 있었다. 이것을 구분하는 함수, 즉 절마다 태그가 있는지, 있으면 어떤 타입의 태그인지 확인하는 함수와 노래에서 태그를 인식해서 `Tags`에 등록하는 작업도 필요했다.
```csharp
class Gospel
{
    // ....
    public static int IsTag(string s)
    {
        if (s[0] == '[' && s[2] == ']' && char.IsLetterOrDigit(s[1])) return 1;
        if (s[1] == '.' && char.IsDigit(s[0])) return 2;
        return 0;
    }
    
    public void ExtractTags()
    {
        if (Tagged) return;
        var tags = Lyrics.Select(s => IsTag(s));
        if (tags.All(i => i == 0)) { Taggable = false; return; }
        for (int i = 0; i < tags.Count(); i++)
        {
            switch (Lyrics[i].IsTag())
            {
                case 1: Tags[Lyrics[i][1].ToKey()] = i + 1; break;
                case 2: Tags[Lyrics[i][0].ToKey()] = i + 1; break;
                default: break;
            }
        }
    }
}
```
이 태그들을 그때그때 등록해서 단축키를 누를 때마다 그 위치로 가게 하면 되었다. 혼란을 피하기 위해 단축키는 한 자리인 숫자와 알파벳 한 글자로 제한했다.

## 가능성과 함께 오는 그림자들
나는 이것을 내가 있는 곳에서도 쓰고, 다른 곳에서도 이렇게 편리한 것을 쓰면 좋겠다는 생각이 들었다. 그래서 다른 지역 사람들에게도 추천하여 같이 쓰게 했다. 그러나 그럴 무렵, 환상과도 같던 가능성들은 그림자를 이끌고 돌아왔다. 이 그림자는 내가 새로운 기능을 개발할수록 짙어져서, 개발하면 할수록 새로운 것을 하기가 점점 더 힘들게 되었다. 이미 개발한 기능들은 저마다 그 속에 작든 크든 오류의 가능성을 가지고 있었고, 이 오류의 가능성은 어떤 작업으로도 배제할 수 없었다. 그렇기 때문에 나는 개발 속도는 점점 느려지고 디버깅 시간만 점점 늘어나게 되었는데, 디버깅을 해도 문제가 원활하게 잡히지 않았다. 그 이유는 다음과 같은 전제가 프로그램 안의 모든 요소들에 있었기 때문이었다. 
- 모든 것은 객체인데, 이 객체는 비어 있을 수도 있고, 엉뚱한 메모리 주소를 가리킬 수도 있다. (`null`)
- 모든 기능(메소드 또는 함수라고 부르는)들은 언제나 어떤 작업을 하고 결과값을 돌려 주는데, 이 작업은 성공할 수도 있고, 실패할 수도 있다. (`Exception`)
- 작업들이 모두 성공해도, 어떤 작업에서 한 영향이 다른 작업을 할 때에 오류를 발생시키기도 한다.
- 객체지향에서 모든 객체는 살아서 상호작용하고 요동치기 때문에 객체에서 발생하는 오류들은 다른 객체들을 타고 전염된다.

내가 알기로는 기능을 구현하기 위해서는 속성과 기능을 추상화해서 구분한 객체를 더 많이 만들어야 하는데, 아이러니하게도 이 객체가 늘어날수록 오류의 혼돈성도 증가하는 것이었다. 언제든지 서로 감염될 수 있고, 언제든지 바이러스가 발생할 수 있었기에, 나는 언제나 이 넘쳐나는 광기를 잠재워야 했다. 게다가 이 오류가 격리되지 못하고 서로 전염되기 때문에, 디버깅을 해도 문제의 근원이 쉽사리 발견되지 않았다. 코드가 많아질수록 나는 내가 작성한 코드와 다른 사람들이 이미 만들어 둔 라이브러리, 심지어는 표준 라이브러리의 코드마저 믿지 못하게 되었고, 자연스레 새로운 기능을 추가하는 것은 꿈도 꿀 수 없게 되었으며, 기존의 코드를 유지보수하는 것조차 힘에 벅차기 시작했다. 그래서 GospelSub 3.1 버전에 이를 즈음에는, 아마 손을 완전히 놓아 버렸던 것 같다.

## 표현의 자유를 찾아서
나는 C#이라는 언어를 C를 배우고 나서 C++을, C++을 배우고 나서 C#을 접하게 되었다. 그래서 C보다 더 간결하게 동일한 작업을 표현할 수 있는 C++에 열광했고, 마찬가지로 C++보다 더 간결하게 표현할 수 있는 C#이 굉장히 매력적으로 느껴졌다. 그래서 C#을 쓸 때쯤에는, 나름 프로그래밍 언어는 이래야 한다는 철학이 자리잡았던 것 같다:
- 읽기 쉽고
- 쓰기 쉽고
- 디버깅도 잘 되는

다시말해 **간결하게 작업을 표현하고, 그래서 이해력을 높이며 타자 수도 줄이는** 언어가 좋다라고 생각하게 되었다. C++과 C#같은 경우는 [연산자 오버로딩](https://en.wikipedia.org/wiki/Operator_overloading)으로 이것을 가능하게 했다.

한편 나는 이 프로그램을 처음에 WinForms로 만들다가 WPF(Windows Presentation Foundation)으로 옮겨가서 만들었는데, 써 보니 WinForms보다 WPF에 열광하게 되었다. 대표적인 이유는 역시 간결함이었다. 예를 들어, UI에서 어떤 부분이 변경되면 그 부분을 수동으로 업데이트해 주어야 했는데, 이는 코드 3~5줄은 항상 차지했다. 그런데 WPF에서는
```xml
<TextBlock Text="{Binding Source={StaticResource myDataSource}, Path=Name}"/>
```
와 같이 간단하게 표현할 수 있었다. 이것은 [MVVM](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)이라고 불리는 방식이었는데 기존에 비해서 아주 간단하게 두 값을 연결해서 표현함으로써 변경시 반영하는 작업을 원활하게 만들어주었다.

또, 나는 그 이후에는 웹을 다루게 되었다. 주로 웹사이트 클라이언트를 다루는 작업이었는데, [AngularDart](https://angulardart.xyz/)를 찾아서 그것으로 시작했다. 그 후에는 [Angular](https://angular.io)로 넘어갔지만, 기본 아이디어는 같았다. 여기서는 [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)를 배웠고 이 방식에 익숙해지게 되었다. 확실히 MVC는 기능을 분리하고 각 가능에 집중함으로써 코드 오류를 막고 이해하기 좀더 쉽게 했다.

나는 방금 설명한 이런 간결함들이 필요했다. 또 나는 예측할 수 없는 오류로부터의 자유를 원했다. 또 나는 한번 기능을 머리쓰고 고민해서 만들었으면 그것은 생각한 대로 잘 작동하기를 원했다. 나는 이것들을 구현한 언어, 나의 이 갈망을 해결해 줄 수 있는 언어를 누군가 이미 만들지 않았을까 싶어서 찾아 헤메기 시작했다.

## 느릅나무(Elm) 아래에서
나는 Angular에도 한계를 느껴서 다른 프레임워크가 없을까 찾고 있었다. 그러다가 [Elm](https://elm-lang.org/)을 알게 되었는데, 홍보 문구부터가 눈길을 끌었다.
> No Runtime Exceptions (실행할 때에 오류 없음)


> My favorite thing about Elm is that I don’t have to worry when coding (내가 Elm을 제일 좋아하는 이유는 코딩할 때 걱정하지 않아도 되서이다)
(David Andrews, Software engineer)


이런 비슷한 것을 찾아 헤메던 나로써는 눈이 돌아가는 문구였다. 나는 당장 언어를 배우고 프로젝트를 만들어보기 시작했다. 그러면서 [함수형 언어](https://en.wikipedia.org/wiki/Functional_programming)라는 것을 처음 보게 되었다. 이 함수형 언어의 기본 아이디어는 이것이었다.
> 수학에서의 함수는 입력을 출력으로 변환하는 역할만 한다. 그러므로 컴퓨터 프로그래밍도 이 함수 단위로 쪼개서 만들자. 그러면 입력을 출력으로 변환시키는 것 외의 다른 효과(Side Effect, 부작용)은 일어나지 않을 것이다.

완전히 이해되지는 않았지만, 어쨌든 Elm은 놀라운 언어였다. 아주 작업을 간결하게 함수를 적용하는 식들로 프로그램을 만들 수 있었고, 홍보한 대로 실행할 때 오류가 전혀 발생하지 않았다. 실로 놀라울 따름이었다. 제한되기는 했지만 기존 Javascript로 만든 라이브러리들과도 호환이 되었다. 이것은 또다른 가능성이었다.

## 순수할 수 있다는 희망
Elm을 보며 나는 신기했다. "어떻게 이렇게 간단한 함수들간의 조합으로만 프로그램을 나타낼 수 있지?" 왜냐하면 내가 아는 한으로는 오류가 발생할 수 있게 되는 이유는 오류가 발생하는, 다시말해 입력을 출력으로 변환시키는 작업이 아니라 외부 세계와 소통해서 상호작용하는 작업에서, 오류가 전염되기 때문이었는데, Elm은 그런게 하나도 없었기 때문이었다. 그냥 모두 값이었다. 나의 이런 의문은, 생각보다 간단한 답을 가지고 있었다. Elm 안에서는 이런 함수들만 이용하는 "순수한" 작업들만 하고, 외부 세계와의 소통은 타입으로 '이렇게 할 것이다'라고 정의한 다음, 나머지는 Elm 런타임이 '알아서 해주는' 것이었다.

나는 이것을 보면서, 새로운 가능성을 가지게 되었다. 이런 "순수한" 언어가 더 있을 것 같았다. 이 "순수함"은 수학에서 말하는 함수를 사용하기 때문에 당연하게도 **간결하고, 한번 고민해서 만들면 다음번에도 동일하게 작동하고, 예측할 수 없는 오류들은 아예 상종을 하지 않았다.** 그러나 Elm은 불편한 점이 있었는데, 이것은 범용 프로그래밍 언어가 아니라 웹사이트, 그 중에서도 웹 클라이언트를 작성하는 데에만 쓸 수 있었기 때문이다. 그래서 나는 만약 이런 좋은 방식이 있다면, 이것으로 뭐든지 만들 수 있기를 원했다. 다시 말해, 나는 "순수한 함수형" 범용 프로그래밍 언어가 필요했다. 나는 이것을 또 찾아 나섰다. 

## 순수할 것이냐 아니할 것이냐 그것이 문제로다
나는 Elm 주변을 뒤지면 비슷한 언어가 나올 것이라 생각했다. 그러다 Elm이 [Haskell](https://www.haskell.org)이라는 언어로 만들어진 것을 알게 되었다. 나는 무작정 Haskell을 따라갔고, Haskell 역시 순수한 함수형 언어였다. 또한 Haskell은 범용 프로그래밍 언어였으므로, 내 기준에 아주 적합했다. 그러나 나는 곧 문제에 직면했는데, 그것은 바로 Haskell이 어렵다는 점이었다. 나는 나름 많은 언어들을 배우고, 봐오고, 그 중의 몇몇은 쓰던 사람으로서 C, C++, C#, Visual Basic, Python, Java, Kotlin, Javascript, Typescript 등 시중에서 많이 쓰는 언어는 거의 안다고 자부하는 사람이었고 많은 언어들을 보아 와서 처음 보는 언어도 대충 문법 보고 파악해서 바로 쓸 수 있는 수준이 되었다고 생각했다. 그런데 (나름 Elm에서 적응되기는 했지만) Haskell의 문법은 난생 처음 보는 문법이었다. 예를 들어 퀵소트는 이렇게 구현한다:
```haskell
quicksort :: Ord a => [a] -> [a]
quicksort []     = []
quicksort (x:xs) = (quicksort lesser) ++ [x] ++ (quicksort greater)
    where
        lesser  = [y | y <- xs, y < x]
        greater = [z | z <- xs, z >= x]
```
간단하다는 건 알 수 있었지만 나에게 써 보라고 한다면 쉽게 쓰기 어려운 코드였다. 연산자가 많아서 좋긴 하지만, 다시 말하면 연산자를 이해하지 못하면 이상한 외계어가 될 뿐이었다. 또다른 예로 구구단을 출력하는 예제를 보면:
```haskell
main :: IO ()
main = mapM_ putStrLn $ stringify <$> [2..9] <*> [1..9]
    where 
        stringify :: Int -> Int -> String
        stringify x y = show x ++ " * " ++ show y ++ " = " ++ show (x * y)
```
Haskell에서 말하는 Functor, Applicative, Monad를 알지 못하면 이 코드를 이해할 수 없었다. Haskell은 [람다 대수](https://en.wikipedia.org/wiki/Lambda_calculus)와 [범주론](https://en.wikipedia.org/wiki/Category_theory)을 기반으로 하여졌고 수학에서 아이디어를 더 많이 끌어 쓰기 때문에, 수학에서 가져온 아이디어인 Semigroup, Monoid, Functor, Applicative, Monad, Comonad, Bifunctor, Profunctor, Arrow, Lense 등등 괴상하고 무서워 보이는 용어들과 개념들이 넘쳐났다. (나중에 이것들은 단지 수학에서 단어와 아이디어만 그대로 가져온 것일 뿐 수학에서 말하는 개념을 이해하지 않아도 된다는 것을 알았지만, 그것은 나중의 일이었다.)

나는 이 때문에 또다시 고민에 빠졌다. '아, 정녕 "순수함"을 어디서나 쓰려면 이런 개념들을 다 알아야 한단 말인가?' '프로그래밍이 원래 이렇게 어려운 일이었나?' '오류가 없고 간결한 건 좋지만 그걸 위해서 너무 많이 희생하는 건 아닌가?' 등 여러 생각들이 나를 쪼아댔다. 그래서 나는 Haskell과 Elm 사이에 있을만한, Elm보다는 범용적이면서 Haskell보다는 간단한 언어를 찾아 나섰다. 구체적으로 설정한 조건들은 다음과 같다:
- 되도록이면 순수 함수형 언어일 것, 그러나 비순수 함수형 언어도 가능
- 간결함과 재사용성
- 프론트엔드 프로그램밍이 가능할 것
  - 기존 Javascript로 만든 라이브러리와 호환될 것
  - Javascript로 컴파일이 되면 좋다. Node.js 를 이용해서 벡엔드에서도 실행할 수 있기 때문


## 자유와 익숙함 사이에서
후보는 여럿 찾을 수 있었다. 일단 C#과 비슷한 [F#](https://en.wikipedia.org/wiki/F_Sharp_(programming_language))이 있었고, [Clojure](https://en.wikipedia.org/wiki/Clojure)라는 언어를 브라우저에서 구동하는 ClojureScript도 있었다. 또 [Ocaml](https://en.wikipedia.org/wiki/OCaml)이라는 언어를 개량하고 확장해서 JS와 같이 쓸 수 있게 한 [ReasonML](https://reasonml.github.io/)이 있었고, 마지막으로 Haskell을 거의 Javascript로 바로 컴파일할 수 있게 한 듯한 언어인 [PureScript](https://www.purescript.org/)가 있었다. Elm 과 Haskell을 포함해서 난이도를 표현하자면 이렇다:
```
Elm < F# =~= ReasonML < ClojureScript < PureScript < Haskell
```
순수함과 그로 인한 자유도는 다음과 같다:
```
ReasonML < ClojureScript < F# < Elm < PureScript < Haskell
```
JavaScript와의 호환성은 비슷하므로, 기존 명령형 문법처럼 익숙한지의 정도를 표기하자면:
```
ReasonML < F# < Elm < ClojureScript < PureScript < Haskell
```
대강 이러한 관계를 가지고 있었다. 그리고 나는 이들 중에서 선택을 해야 했다. 왜냐하면 나는 여기서 선택한 언어로 프론트엔드를 짜고 싶었기 때문이다. 나는 글로 본 이들 언어들의 특성으로는 부족하다고 판단하고 이들 중 ReasonML과 F#, PureScript를 직접 사용해 보고, 그 언어들의 철학도 간단히 배웠다. 프로그래밍 언어는, 다른 언어들과 마찬가지로, 언어의 문법이 사고방식을 반영하는 그 동시에 역시 얽매이는 것을 잘 알고 있었기 때문에, 나는 그 언어들의 이면에 있는 이념들도 필요했다. 내가 계속, 어쩌면 평생, 쓸 언어들이었기 때문이다. 

그런데 이렇게 장기적으로 쓸 것이라고 생각하니, 결정하기가 수월해졌다. 왜냐하면 어렵다는 말과 익숙하지 않다는 말은 동의어이기 때문이다. 그 말은 어려움과 익숙하지 않은 것은 시간이 해결해 줄 것이라는 말이었다. 그리고 나는 언어의 표현력이 나의 사고를 제한한 경혐을 이미 충분히 했다. 그리고 나 스스로도 생각해 보았을 때에, 나에게 가장 중요한 것은 자유였다. **생각할 자유, 생각한 대로 프로그래밍할 자유, 그리고 생각한 대로 오류는 신경쓰지 않을 자유.** 그래서 나는 결국, Haskell 밑으로 가장 어렵지만, 그만큼 자유도 또한 높은 PureScript를 선택했다. 물론 배움의 양도 많이 필요했지만, 어차피 개발자는 죽을 때까지 계속 배우는 직업으로 알고 있었기 때문에, 별로 신경쓰지 않았다. 또 다행히도, PureScript 커뮤니티는 [예제로 배우는 PureScript](https://book.purescript.org/)라는 교재를 온라인으로 준비해 주어서, 별로 마땅한 교재가 없어서 배우기가 힘들었던 Haskell과 달리, 훨씬 배우기가 수월했다. 

## 더 문제를 잘 해결할 자유
나는 PureScript를 배우는 데에 꽤나 시간이 걸렸고, 마침내 이것으로 gospelsub을 [다시 만들었다.](https://github.com/DavidLee18/gospelsub-pure) 나는 PureScript로 프로그램을 작성하면서 놀라움을 금할 수 없었다. 그와 동시에 C#으로 작성했을 때는 왜 그렇게 오류가 많았는지 깨달았다. 이유는 간단했다, C#은 찬양이라는 데이터를 이렇게 말할 능력이 없었기 때문이다:
```haskell
data Gospel
    = Gospel { name :: String, lyrics :: Array Verse }
    | Hymn { index :: Int, name :: String, lyrics :: Array Verse }
    
data Verse
    = Verse String
    | TaggedVerse Key String
```
PureScript는 Tag 자체를 데이터 안에 내장하고 그 경우의 수를 설계할 수 있게 해 주었다. 데이터는 살아 요동하고 변화하는 `Object`가 아니라, 죽어있고 절대 변하지 않는 값인 상태가 더 나았다.

또한, gospelsub 프로그램에서 찬양을 표시할 때, 현재 표시되는 화면은 어는 찬양의 어느 부분인지 기억하는 것이 필요했고, 이것을 나는 C# 에서는 `Current`와 `Index`로 구현하고, 표시되는 내용은 `Text` 프로퍼티로 구현했었다:
```csharp
public partial class DisplayWindow : Window
    {
        private Gospel gospel;
        private int current = 0;
        private int index = 0;
        private string raw = string.Empty;
        private readonly List<TextBlock> texts;

        internal int Index
        {
            get => index;

            set
            {
                try
                {
                    Text = gospel[value].IsTag() == 1 ? gospel[value].Substring(3) : gospel[value];
                    index = value;
                }
                catch (IndexOutOfRangeException e)
                {
                    try
                    {
                        switch (e.Message)
                        {
                            case "Over": Current++; Index = 0; break;
                            case "Below": Current--; Index = 0; break;
                        }
                    }
                    catch (ArgumentException) { }
                    catch (IndexOutOfRangeException) {  }
                }
            }
        }

        internal int Current
        {
            get => current;
            set
            {
                if (value < 0 || value >= Setting.Current.Sequence.Count) throw new IndexOutOfRangeException();
                current = value;
                Cursor = Cursors.Wait;
                DisposeTags();
                gospel = Setting.Current.Sequence[current];
                if (gospel.Tagged) RegisterTags();
                Cursor = Cursors.None;
            }
        }

        private string Text
        {
            get => raw;
            set
            {
                raw = value;
                if (raw == gospel.Name)
                {
                    (texts[0].Text, texts[1].Text) = (string.Empty, string.Empty);
                    texts[2].Text = raw;
                    linePanel.Visibility = Visibility.Collapsed;
                }
                else
                {
                    linePanel.Visibility = Visibility.Visible;
                    string[] vs = raw.Split(new string[1] { @"
" }, StringSplitOptions.RemoveEmptyEntries);
                    texts[2].Text = vs.Length == 2 ? vs[1] : string.Empty;
                    double sigma = vs[0].Length / 2;
                    int ideal = (int)Math.Round(sigma);

                    var res = from i in Enumerable.Range(0, vs[0].Length)
                              where vs[0][i] == ' '
                              orderby Math.Abs(sigma - i)
                              select i;
                    int tocut = res.First();
                    texts[0].Text = vs[0].Substring(0, tocut);
                    texts[1].Text = vs[0].Substring(tocut);
                }
            }
        }
    }
```
PureScript로는 이것을 합쳐서 `position`으로 추상화하고 구현하면 되었다. 그냥 코드의 양이 줄은 것만 감상해도 된다:
```haskell
type State =
    { blind :: Boolean
    , gospels :: Array Gospel
    , movingGospel :: Maybe Gospel
    , position :: Maybe (Gospel /\ (Either String Verse))
    , queue :: Array Gospel
    , route :: Route
    , subId :: Maybe SubscriptionId
    , textSize :: Int
    }
    
render :: forall w. State -> HTML w Action
render { blind, position, route: Display, textSize } = HH.div_ [
        HH.h2 [ style $ "font-size: " <> show textSize <> "px;" ] if blind then [] else fromMaybe [ HH.text "Loading Gospels..." ] titleOrVerse
        ]
    where titleOrVerse = Array.intersperse HH.br_ <<< map HH.text <<< String.split (Pattern "\\n") <<< either identity Verse.string <<< extract <$> position
```
PureScript에는 이렇게 말할 능력이 있었고 따라서 코드는 더 간결하면서 동시에 더 견고해졌다. C#에서는 이렇게 말할 능력이 없어서 그렇게 장황하게 말했던 것이다.
