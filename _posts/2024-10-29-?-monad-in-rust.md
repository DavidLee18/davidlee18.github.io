---
title: "?: 러스트에 있는 모나드"
layout: post
date: 2024-10-29 14:12:43 +0900
categories:
- functional-programming
- rust
---
함수형 언어에만 있는 "모나드"라는 것이 있습니다. 이것은 매우 편리해서 "프로그래밍 가능한 세미콜론"이라는 명칭까지 있는데요, 이 모나드를 이용하면 어떤 문장마다 일어나야 할 동작을 입맛대로 설정해서 사용할 수 있습니다. 
순수한 함수형 언어인 하스켈에서는 변경 작업, 리스트 연산, 각종 부작용 등의 동작에 모나드를 적극 활용하고 있습니다. 심지어는 모나드를 활용해서 실제 환경을 모방, 실제에서 작동하는 것처럼 프로그램을 테스트하는 데에도 사용하죠. 
하스켈에서 많은 부분 타입 시스템을 가져온 러스트에서도 이런 모나드가 있었으면 좋았을 텐데, 처음에는 러스트에는 모나드가 없다고 생각했었습니다. 하지만 러스트에서는 제한된 형태이지만, 모나드가 있었습니다. 
하지만 "대체 그래서 모나드가 뭔데?"라고 하실 분들을 위해, 먼저 모나드를 소개하겠습니다.

## Monad
`Monad`는 하스켈의 타입 클래스 중 하나입니다. "타입 클래스는 또 뭔데?" 아, 걱정 마세요. 타입 클래스는 일종의 인터페이스이지만, 더 정확히는 러스트의 트레잇과 더 비슷합니다. 말 그대로 타입들을 나누는 기준이 되는 것입니다. 
하여튼 하스켈에서 모나드는 이렇게 정의되어 있습니다:
```haskell
class Applicative m => Monad (m :: Type -> Type) where
  (>>=) :: m a -> (a -> m b) -> m b
```
그래서 이걸 어디다 써먹냐구요? 예시를 한번 보여드리겠습니다. 아마 널(NULL)의 공포에 대해서는 아마 알고 계실 겁니다. 어떤 값이든 널이 될 수 있죠. 하스켈에는 기본적으로 널이 없습니다. 
대신, 비어 있을 수도 있는 타입을 위한 제네릭 타입 `Maybe`를 정의합니다:
```haskell
data Maybe a = Nothing | Just a
```
문제는 여기서 생깁니다. 만약 널일 수도 있는 값을 반환하는 함수가 있다고 합시다. 예를 들어, 디렉토리의 상위 디렉토리를 반환하는 함수 같은 거요:
```haskell
parentDir :: Directory -> Maybe Directory
```
그런데 우리는 이것을 3번 반복한, 상위 3층에 있는 디렉토리를 접근하고 싶다고 해 봅시다. 그러면 일단 이렇게 작성할 겁니다:
```haskell
thirdParent :: Directory -> Maybe Directory
thirdParent dir = case parentDir dir of
  Nothing -> Nothing
  Just d1 -> case parentDir d1 of
    Nothing -> Nothing
    Just d2 -> case parentDir d2 of
      Nothing -> Nothing
      Just d3 -> Just d3
```
좀 귀찮네요. 그냥 값이 있는 것으로 가정해서 접근하다가 값이 없으면 일찍 반환하면 좋을 텐데요. 그렇지 않습니까?

다행히, `Maybe`는 `Monad`입니다:
```haskell
instance Monad Maybe where
  Nothing >>= _ = Nothing
  Just a >>= f = f a
```
여기서 주목할 부분이 바로 값이 비어 있을 때입니다. 하스켈은 "게으른" 언어여서, 필요하지 않으면 값을 계산하지 않습니다. 따라서, `(>>=)` 앞에 있는 값이 비어 있을 경우, 뒤의 값을 계산하지 않고 바로 빈 값을 반환합니다! 
우리가 필요했던 부분이죠, 여기에 하스켈의 특수 문법인 `do` 기법을 얹으면 이렇게 됩니다:
```haskell
thirdParent :: Directory -> Maybe Directory
thirdParent dir = do
  d1 <- parentDir dir
  d2 <- parentDir d1
  d3 <- parentDir d2
  return d3
```
어떻습니까, 훨씬 간단하면서 깔끔하지 않습니까? 이건 단지 `Maybe` 모나드의 작동 방식일 뿐입니다. 매우 똑같이 보이는 식도 전혀 다른 의미를 부여할 수 있지요. 또다른 예를 들어 보겠습니다.

