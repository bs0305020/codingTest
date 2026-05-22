# Assignment 9: Prolog로 작성한 GCD 및 LCM

학번: 202255601  
이름: 전준  

---

## 1. 문제

이번 과제는 Prolog를 이용하여 두 양의 정수의 최대공약수와 최소공배수를 구하는 프로그램을 작성하는 것이다.

입력은 두 줄로 주어지며, 각 줄에는 양의 정수가 하나씩 들어온다.  
프로그램은 두 수의 최대공약수인 GCD와 최소공배수인 LCM을 계산한 뒤, `LCM - GCD` 값을 출력해야 한다.

예를 들어 입력이 다음과 같다면,

```text
12.
15.
```

12와 15의 최대공약수는 3이고, 최소공배수는 60이다.  
따라서 출력은 `60 - 3 = 57`이 된다.

이번 구현에서는 SWI-Prolog의 내장 `gcd` 기능을 사용하지 않고, 직접 만든 술어를 이용하여 GCD와 LCM을 계산하였다.

---

## 2. 전략

먼저 최대공약수는 유클리드 호제법을 이용하여 계산하였다.  
두 수가 같으면 그 값이 최대공약수가 된다.  
두 수가 다르면 더 큰 수에서 작은 수를 빼는 과정을 반복하여 결국 같은 값이 나올 때까지 계산한다.

작성한 GCD 술어는 다음과 같다.

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

여기서 `jj_gcd(X, Y, G)`는 `X`와 `Y`의 최대공약수가 `G`일 때 성립하는 술어이다.  
예를 들어 `jj_gcd(12, 15, G)`를 실행하면 `G = 3`이 된다.

최소공배수는 다음 공식을 이용하였다.

```text
LCM = X * Y / GCD
```

이를 Prolog 코드로 작성하면 다음과 같다.

```prolog
jj_lcm(X, Y, L) :-
    jj_gcd(X, Y, G),
    L is X * Y // G.
```

`//`는 정수 나눗셈을 의미한다.  
두 수의 최소공배수는 항상 정수이기 때문에 이 연산을 사용하였다.

마지막으로 `main`에서는 두 수를 입력받고, GCD와 LCM을 각각 구한 뒤 차이를 출력하도록 작성하였다.

```prolog
:- initialization(main, main).

main :-
    read(X),
    read(Y),
    jj_gcd(X, Y, G),
    jj_lcm(X, Y, L),
    Answer is L - G,
    writeln(Answer),
    halt.
```

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

프로그램의 실행 흐름은 다음과 같다.

1. 첫 번째 양의 정수를 입력받는다.
2. 두 번째 양의 정수를 입력받는다.
3. `jj_gcd`를 이용해 최대공약수를 구한다.
4. `jj_lcm`을 이용해 최소공배수를 구한다.
5. `LCM - GCD` 값을 계산하여 출력한다.

---

## 3. 언어의 모듈 기능

Prolog에서도 코드를 여러 파일이나 모듈로 나누어 작성할 수 있다.  
SWI-Prolog에서는 보통 `module/2` 지시어를 사용하여 모듈을 정의한다.

예를 들어 어떤 파일을 모듈로 만들려면 파일 맨 위에 다음과 같이 작성할 수 있다.

```prolog
:- module(gcd_module, [jj_gcd/3]).
```

여기서 `gcd_module`은 모듈 이름이고, `[jj_gcd/3]`은 외부에서 사용할 수 있도록 공개하는 술어 목록이다.  
`jj_gcd/3`에서 `/3`은 인수가 3개인 술어라는 뜻이다.

이번 과제에서 내가 실제로 작성한 코드는 하나의 파일인 `gcdlcm.pl` 안에 GCD, LCM, main을 모두 넣는 방식이다.  
프로그램 규모가 크지 않기 때문에 한 파일에 작성해도 전체 흐름을 이해하기 어렵지 않았다.

하지만 GCD와 LCM을 다른 모듈로 나누어 작성한다면 다음과 같은 방식도 가능하다.

먼저 GCD만 담당하는 `gcd_module.pl` 파일을 만들 수 있다.

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

그리고 LCM을 담당하는 `lcm_module.pl` 파일에서는 GCD 모듈을 불러와 사용할 수 있다.

```prolog
:- module(lcm_module, [jj_lcm/3]).
:- use_module(gcd_module, [jj_gcd/3]).

jj_lcm(X, Y, L) :-
    jj_gcd(X, Y, G),
    L is X * Y // G.
```

마지막으로 실제 실행을 담당하는 파일에서는 두 모듈을 불러와 사용할 수 있다.

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
따라서 나중에 GCD 계산 방식을 바꾸더라도 LCM이나 main 코드 전체를 건드리지 않아도 된다는 장점이 있다.

---

## 4. 토의

Prolog에서 모듈을 사용하는 방법은 여러 가지가 있다.

첫 번째 방법은 이번 과제에서 설명한 것처럼 `module/2`와 `use_module/1` 또는 `use_module/2`를 사용하는 방법이다.

```prolog
:- use_module(gcd_module).
```

또는 필요한 술어만 지정해서 불러올 수도 있다.

```prolog
:- use_module(gcd_module, [jj_gcd/3]).
```

필요한 술어만 불러오는 방식은 어떤 술어가 외부에서 사용되는지 명확하게 보인다는 장점이 있다.  
특히 파일이 많아질수록 불필요한 술어까지 가져오지 않아도 되므로 코드 관리가 더 깔끔해진다.

두 번째 방법은 모듈 접두사를 붙여서 사용하는 방법이다.

```prolog
gcd_module:jj_gcd(12, 15, G).
```

이 방식은 `jj_gcd`가 어느 모듈에 있는 술어인지 바로 알 수 있다는 장점이 있다.  
다만 코드가 조금 길어질 수 있다.

세 번째 방법은 단순히 파일을 consult해서 사용하는 방식이다.

```prolog
:- consult('gcd_module.pl').
```

이 방법은 간단하지만, 모듈처럼 공개할 술어와 숨길 술어를 명확하게 구분하기는 어렵다.  
작은 프로그램에서는 편할 수 있지만, 코드가 커지면 이름 충돌이 생기거나 관리가 어려워질 수 있다.

이번 과제는 코드 길이가 짧아서 하나의 파일로 작성해도 충분했다.  
하지만 GCD와 LCM을 각각 다른 기능으로 보고 관리한다면, 모듈로 나누는 방식이 더 깔끔할 수 있다고 생각했다.

---

## 5. 결론

이번 과제를 하면서 Prolog에서는 값을 반환하는 방식이 일반적인 언어와 다르다는 점을 다시 느꼈다.  
C나 Python에서는 함수가 `return`으로 결과를 돌려주는 방식에 익숙하지만, Prolog에서는 `jj_gcd(X, Y, G)`처럼 결과에 해당하는 값을 술어의 인수로 두고 계산한다.

처음에는 이 방식이 조금 어색했다.  
특히 `G`나 `L`처럼 결과를 저장할 변수를 인수에 같이 넣는다는 점이 일반적인 함수 호출과 다르게 느껴졌다.  
하지만 익숙해지고 나니 “두 수의 GCD가 G이다”라는 관계를 그대로 표현하는 방식이라고 이해할 수 있었다.

가장 흥미로웠던 부분은 LCM을 구할 때 이미 작성한 GCD 술어를 그대로 활용할 수 있었다는 점이다.  
`jj_lcm` 안에서 다시 `jj_gcd`를 호출하니, 두 계산이 자연스럽게 연결되었다.

또한 모듈에 대해 생각하면서, 작은 프로그램은 한 파일로 작성하는 것이 편하지만 프로그램이 커지면 기능별로 파일을 나누는 것이 더 좋겠다고 느꼈다.  
이번 과제를 통해 Prolog에서 술어를 이용해 계산을 표현하는 방법과, 모듈을 통해 코드를 분리하는 기본적인 방법을 알 수 있었다.

---

## 참고 문헌

1. Programming Languages, Spring 2026, Prof. Woo, **4 GCD and LCM**
2. SWI-Prolog Documentation
