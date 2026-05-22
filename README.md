# Assignment 9: Prolog로 작성한 GCD 및 LCM

학번: 202255601  
이름: 전준  

---

## 1. 문제

이번 과제는 Prolog로 두 양의 정수를 입력받아 최대공약수와 최소공배수를 구하는 프로그램을 작성하는 것이다.

입력은 양의 정수 두 개가 각각 한 줄씩 주어진다.  
프로그램에서는 먼저 두 수의 최대공약수인 GCD를 구하고, 그다음 최소공배수인 LCM을 구한다.  
마지막으로 `LCM - GCD` 값을 출력하면 된다.

예를 들어 입력이 다음과 같으면,

```text
12.
15.
```

12와 15의 최대공약수는 3이고, 최소공배수는 60이다.  
따라서 출력값은 `60 - 3`인 `57`이 된다.

이번 과제에서는 SWI-Prolog에서 제공하는 내장 `gcd` 기능을 사용하지 않고, 직접 술어를 작성해서 최대공약수를 구하도록 구현하였다.

---

## 2. 전략

최대공약수는 유클리드 호제법을 이용해서 구하였다.  
두 수가 같아지면 그 값이 최대공약수이고, 두 수가 다르면 더 큰 수에서 작은 수를 빼는 과정을 반복하면 된다.

그래서 GCD를 구하는 술어를 다음과 같이 작성하였다.

```prolog
jj_gcd(X, X, X) :- !.

jj_gcd(X, Y, G) :-
    X > Y,
    X1 is X - Y,
    jj_gcd(X1, Y, G).

jj_gcd(X, Y, G) :-
    X < Y,
    Y1 is Y - X,
    jj_gcd(X, Y1, G).
```

여기서 `jj_gcd(X, Y, G)`는 `X`와 `Y`의 최대공약수가 `G`라는 뜻이다.  
처음에는 함수처럼 값을 반환하는 방식으로 생각했는데, Prolog에서는 결과값도 인수에 넣어 관계로 표현한다는 점이 달랐다.

최소공배수는 최대공약수를 이용해서 구하였다.

```text
LCM = X * Y / GCD
```

이를 Prolog로 작성하면 다음과 같다.

```prolog
jj_lcm(X, Y, L) :-
    jj_gcd(X, Y, G),
    L is X * Y // G.
```

`jj_lcm` 안에서 먼저 `jj_gcd`를 이용해 최대공약수를 구하고, 그 값을 이용해 최소공배수를 계산하였다.  
두 수가 양의 정수이므로 최소공배수도 정수이고, 그래서 정수 나눗셈인 `//`를 사용하였다.

전체 코드는 다음과 같다.

```prolog
:- initialization(main, main).

jj_gcd(X, X, X) :- !.

jj_gcd(X, Y, G) :-
    X > Y,
    X1 is X - Y,
    jj_gcd(X1, Y, G).

jj_gcd(X, Y, G) :-
    X < Y,
    Y1 is Y - X,
    jj_gcd(X, Y1, G).

jj_lcm(X, Y, L) :-
    jj_gcd(X, Y, G),
    L is X * Y // G.

main :-
    read(X),
    read(Y),
    jj_gcd(X, Y, G),
    jj_lcm(X, Y, L),
    Answer is L - G,
    writeln(Answer),
    halt.
```

실행 흐름은 간단하다.

1. 첫 번째 수를 입력받는다.
2. 두 번째 수를 입력받는다.
3. `jj_gcd`로 최대공약수를 구한다.
4. `jj_lcm`으로 최소공배수를 구한다.
5. `LCM - GCD` 값을 출력한다.

---

## 3. 언어의 모듈 기능

Prolog에서도 기능별로 코드를 나누기 위해 모듈을 사용할 수 있다.  
SWI-Prolog에서는 보통 파일 위쪽에 `module/2`를 작성해서 해당 파일을 모듈로 만든다.

예를 들어 GCD 기능만 따로 모듈로 만들면 다음과 같이 쓸 수 있다.

```prolog
:- module(gcd_module, [jj_gcd/3]).
```

여기서 `gcd_module`은 모듈 이름이고, `[jj_gcd/3]`은 이 모듈 밖에서도 사용할 수 있게 공개할 술어를 의미한다.  
`jj_gcd/3`에서 3은 인수가 3개라는 뜻이다.

이번에 내가 작성한 코드는 프로그램이 길지 않아서 `gcdlcm.pl` 하나에 모두 작성하였다.  
GCD, LCM, main이 모두 한 파일에 있어도 코드가 복잡하지 않기 때문에 이 방식이 더 간단하다고 판단했다.

하지만 기능을 나눈다면 GCD와 LCM을 각각 다른 모듈로 작성할 수도 있다.

예를 들어 `gcd_module.pl`은 다음처럼 작성할 수 있다.

```prolog
:- module(gcd_module, [jj_gcd/3]).

jj_gcd(X, X, X) :- !.

jj_gcd(X, Y, G) :-
    X > Y,
    X1 is X - Y,
    jj_gcd(X1, Y, G).

jj_gcd(X, Y, G) :-
    X < Y,
    Y1 is Y - X,
    jj_gcd(X, Y1, G).
```

그리고 `lcm_module.pl`에서는 GCD 모듈을 불러와서 사용할 수 있다.

```prolog
:- module(lcm_module, [jj_lcm/3]).
:- use_module(gcd_module, [jj_gcd/3]).

jj_lcm(X, Y, L) :-
    jj_gcd(X, Y, G),
    L is X * Y // G.
```

마지막 실행 파일에서는 두 모듈을 불러와서 다음처럼 사용할 수 있다.

```prolog
:- initialization(main, main).

:- use_module(gcd_module, [jj_gcd/3]).
:- use_module(lcm_module, [jj_lcm/3]).

main :-
    read(X),
    read(Y),
    jj_gcd(X, Y, G),
    jj_lcm(X, Y, L),
    Answer is L - G,
    writeln(Answer),
    halt.
```

이렇게 나누면 GCD를 계산하는 부분과 LCM을 계산하는 부분이 분리된다.  
지금처럼 짧은 프로그램에서는 오히려 한 파일이 편하지만, 프로그램이 길어지면 모듈로 나누는 편이 수정하거나 확인하기 쉬울 것 같다.

---

## 4. 토의

Prolog에서 모듈을 사용하는 방법은 여러 가지가 있다.

가장 기본적인 방법은 `use_module`을 이용해 필요한 파일을 불러오는 것이다.

```prolog
:- use_module(gcd_module).
```

이렇게 하면 해당 모듈에서 공개한 술어를 사용할 수 있다.  
또는 필요한 술어만 골라서 불러올 수도 있다.

```prolog
:- use_module(gcd_module, [jj_gcd/3]).
```

나는 두 번째 방식이 더 명확하다고 생각했다.  
어떤 술어를 가져와서 쓰는지 코드에 바로 보이기 때문이다.  
파일이 많아지면 그냥 전체를 불러오는 것보다 필요한 것만 불러오는 방식이 더 안전할 것 같다.

또 다른 방법은 모듈 이름을 앞에 붙여서 술어를 호출하는 것이다.

```prolog
gcd_module:jj_gcd(12, 15, G).
```

이 방식은 `jj_gcd`가 어느 모듈에 있는지 확실히 보인다는 장점이 있다.  
다만 매번 모듈 이름을 붙여야 하므로 코드가 조금 길어진다.

간단하게 파일을 불러오는 방법으로는 `consult`도 있다.

```prolog
:- consult('gcd_module.pl').
```

이 방법은 사용하기 쉽지만, 모듈처럼 어떤 술어를 공개하고 감출지 구분하기는 어렵다.  
작은 코드에서는 괜찮지만, 코드가 커지면 이름이 겹치거나 관리하기 어려울 수 있다.

이번 과제에서는 코드가 짧아서 하나의 파일로 작성하는 것이 더 편했다.  
하지만 GCD나 LCM처럼 기능이 분명히 나뉘는 경우에는 모듈을 사용하면 역할을 나누기 쉽다는 점을 알 수 있었다.

---

## 5. 결론

이번 과제를 하면서 Prolog에서는 결과를 `return`으로 돌려주는 방식이 아니라, 결과가 들어갈 변수를 술어의 인수로 함께 넘긴다는 점이 가장 다르게 느껴졌다.

예를 들어 Python이라면 `gcd(12, 15)`처럼 호출해서 결과를 반환받는 식으로 생각하기 쉽다.  
하지만 Prolog에서는 `jj_gcd(12, 15, G)`처럼 작성하고, 이 관계를 만족하는 `G`를 찾는 방식으로 동작한다.

처음에는 이 방식이 조금 헷갈렸다.  
특히 `G`를 어디서 계산해서 넣는다는 느낌보다, `X`와 `Y`의 최대공약수가 `G`가 되도록 Prolog가 맞춰간다는 느낌으로 이해해야 했다.

---

## 참고 문헌

1. 부산대학교 CSE 프로그래밍언어론 강의자료, prolog(Part 1)
2. 부산대학교 CSE 프로그래밍언어론 강의자료, prolog(Part 2)
