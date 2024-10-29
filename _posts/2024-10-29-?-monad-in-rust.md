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
`Monad`는 하스켈의 타입 클래스 중 하나입니다. "타입 클래스는 또 뭔데?" 아, 걱정 마세요. 타입 클래스는 일종의 인터페이스이지만, 러스트의 트레잇과 더 비슷합니다. 말 그대로 타입들을 나누는 기준이 되는 것입니다. 
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

표준 입력에서 문자열을 한 줄씩 받는 함수를 생각해 봅시다. 이 함수는 여러 가지에 영향을 받아서 실패할 수도 있습니다. 운영체제에 오류가 났을 수도 있고, 입력이 끝났을 수도 있죠. 이런 경우 보통의 프로그램은 예외(Exception)을 던지는데, 이 예외를 추적하는 작업이 또 골 때립니다. 눈에 핏줄을 세우고 예외를 추적해야 하죠. 하스켈에서는 이런 경우를 위해 `Either` 타입을 정의합니다:
```haskell
data Either a b = Left a | Right b
```
방금 말한 함수는 간단히 이와 같이 정의할 수 있습니다:
```haskell
getNextLine :: () -> Either IoError String
```
그럼 또 이 함수를 여러 번 사용해 볼까요? 표준 입력에서 3줄을 읽는 함수를 생각해 봅시다. 아, 또 벌써 귀찮군요. 아까처럼 간단하게 만들 수는 없을까요?

물론 가능합니다! `Either` 역시 `Monad`거든요!
```haskell
instance Monad (Either a) where
  Left a >>= _ = Left a
  Right b >>= f = f b
```
우리의 함수는 이렇게 됩니다:
```haskell
read3Lines :: () -> Either IoError String
read3Lines () = do
  s1 <- getNextLine ()
  s2 <- getNextLine ()
  s3 <- getNextLine ()
  return (s1 ++ s2 ++ s3)
```
어떻습니까? 여기서 모나드의 문법은 예외가 발생할 경우 실행을 멈추고, 가장 먼저 발생한 예외를 반환합니다. 따라서 어디서 문제가 생겼는지 바로 알 수 있죠. 이와 같이 편리한 것이 바로 모나드입니다. 
저는 이와 같은 편리함 때문에, 하스켈과 비슷한 러스트에도 모나드가 있기를 바래 왔습니다. 하지만 처음에는 찾지 못했죠.

## Try (`?`)
하지만 러스트에는 모나드가 있었습니다! 바로 `?` 연산자가 그것입니다. 앞서 들었던 `Maybe`와 `Either`가 러스트에도 비슷하게 있습니다. 바로 `Option`과 `Result`입니다.
```rust
enum Option<T> { None, Some(T) }

enum Result<T, E> { Err(E), Ok(T) }
```
이전 예제를 반복해서 보여 드리겠습니다. `fn parent_dir(dir: Directory) -> Option<Directory>` 가 있을 때, 이와 같이 작성할 수 있습니다:
```rust
fn third_parent(dir: Directory) -> Option<Directory> {
  let d1 = parent_dir(dir)?;
  let d2 = parent_dir(d1)?;
  let d3 = parent_dir(d2)?;
  Some(d3)
}
```
마찬가지로 `fn get_next_line() -> Result<String, io::Error>`이 있을 때, 이와 같이 작성할 수 있지요:
```rust
fn read3lines() -> Result<String, io::Error> {
  let s1 = get_next_line()?;
  let s2 = get_next_line()?;
  let s3 = get_next_line()?;
  Ok(format!("{}{}{}", s1, s2, s3))
}
```
여기서 바로 `?` 연산자가 바로 모나드의 역할을 그대로 하고 있습니다. 그러면 궁금하지 않습니까? 어떻게 `?`같은 것이 러스트 코드일까요? 특수 문법일까요? 그것도 맞지만, `Monad`와 비슷한 트레잇이 존재합니다. 바로 `Try`입니다.
```rust
pub trait Try: FromResidual {
    type Output;
    type Residual;

    fn from_output(output: Self::Output) -> Self;
    fn branch(self) -> ControlFlow<Self::Residual, Self::Output>;
}
```
여기서 `branch` 메서드가 바로 일찍 반환할지, 아니면 값을 추출해서 계속 이어나갈지를 결정합니다.
