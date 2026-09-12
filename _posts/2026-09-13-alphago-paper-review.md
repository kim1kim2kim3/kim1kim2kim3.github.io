---
title: "[논문리뷰] David Silver et al., “Mastering the game of Go with deep neural networks and tree search”"
categories: [논문리뷰, RL]
tags: [alphago, reinforcement-learning, deep-learning, mcts]
math: true
---

## Paper

**Mastering the Game of Go with Deep Neural Networks and Tree Search**  
David Silver et al.  
Nature, 2016

---

## 1. Motivation

바둑은 가능한 상태와 행동의 수가 매우 크기 때문에
기존의 brute-force search만으로 해결하기 어렵다.

체스에서는 평가 함수와 탐색을 결합하는 방식이 성공적이었지만,
바둑에서는 두 가지 문제가 특히 어렵다.

1. 현재 상태에서 어떤 수가 좋은지 판단하는 것
2. 현재 바둑판 상태에서 누가 이길지 평가하는 것

이 논문은 이 두 문제를 각각 **Policy Network**와 **Value Network**로 학습하고,
이를 **Monte Carlo Tree Search (MCTS)**와 결합한다.

---

## 2. Main Idea

AlphaGo의 핵심 구성 요소는 크게 세 가지다.

- Policy Network
- Value Network
- Monte Carlo Tree Search

대략적인 구조는 다음과 같다.

```text
Board State
    |
    v
Policy Network
    |
    | 좋은 수의 후보
    v
   MCTS
    ^
    |
Value Network
    |
    | 현재 상태의 승리 가능성
```

Policy Network는

> 어디에 돌을 둘 것인가?

를 판단하고,

Value Network는

> 현재 상태에서 누가 이길 가능성이 높은가?

를 판단한다.

그리고 MCTS가 두 네트워크를 이용하여 실제 다음 수를 결정한다.

---

## 3. Policy Network

Policy Network는 현재 상태 $s$가 주어졌을 때
행동 $a$를 선택할 확률을 모델링한다.

$$
p(a \mid s)
$$

즉 현재 바둑판을 입력으로 받아
각 위치에 돌을 둘 확률을 출력한다.

### 3.1 Supervised Learning Policy

먼저 프로 기사들의 기보를 이용해 학습한다.

입력:

$$
s
$$

출력:

$$
p(a \mid s)
$$

즉,

> 프로 기사가 이 상황에서 어디에 돌을 두었는가?

를 예측하도록 학습한다.

이를 통해 사람의 바둑을 모방하는 초기 Policy Network를 얻는다.

---

## 4. Reinforcement Learning Policy

Supervised Learning만 사용하면 결국
사람이 두었던 수를 따라 하는 수준에 머물 수 있다.

AlphaGo는 여기서 Self-Play를 이용한 강화학습을 추가한다.

현재 Policy Network가 자기 자신과 계속 게임을 수행하면서
승리할 확률을 높이는 방향으로 Policy를 개선한다.

이를 개념적으로 쓰면 목표는

$$
J(\theta)
=
\mathbb{E}_{\pi_\theta}[R]
$$

를 최대화하는 것이다.

여기서

- $\theta$: Policy Network의 parameter
- $R$: 게임 결과에 따른 reward

이다.

즉 단순히 사람의 행동을 따라 하는 것이 아니라,

> 실제로 게임에서 이기기 좋은 행동

을 학습하게 된다.

---

## 5. Value Network

Policy Network가 다음 행동을 결정한다면
Value Network는 현재 상태 자체를 평가한다.

Value function은 다음처럼 생각할 수 있다.

$$
V(s)
=
\mathbb{E}[z \mid s]
$$

여기서 $z$는 최종 게임 결과이다.

따라서 Value Network는 현재 바둑판 상태 $s$를 보고

> 이 상태에서 최종적으로 이길 가능성이 얼마나 되는가?

를 예측한다.

기존 Monte Carlo 방식처럼 게임을 끝까지 수없이 진행하지 않고도
중간 상태의 가치를 Neural Network로 평가할 수 있게 된다.

---

## 6. Monte Carlo Tree Search

실제 AlphaGo는 Policy Network에서 가장 높은 확률이 나온 수를
그냥 바로 선택하지 않는다.

MCTS를 사용해서 미래의 수를 탐색한다.

```text
Current State
      |
  -----------
  |    |    |
  A    B    C
 / \       / \
... ...   ... ...
```

여기서 Policy Network는

> 어떤 branch를 우선적으로 탐색할 것인가?

에 도움을 주고,

Value Network는

> 이 상태가 얼마나 유리한가?

를 평가하는 데 사용된다.

즉 Neural Network와 Search를 결합하는 것이
AlphaGo의 중요한 특징이다.

---

## 7. Training Pipeline

논문의 학습 과정을 단순화하면 다음과 같이 볼 수 있다.

### Step 1. Human Games

프로 기사들의 기보를 사용해
Supervised Learning Policy Network를 학습한다.

```text
Human Games
     ↓
SL Policy Network
```

### Step 2. Self-Play

Policy Network가 자기 자신과 게임하면서
Reinforcement Learning으로 성능을 개선한다.

```text
SL Policy
    ↓
Self-Play
    ↓
RL Policy
```

### Step 3. Value Network

Self-Play 게임에서 생성된 데이터를 이용해
각 상태에서 최종 승자를 예측하는 Value Network를 학습한다.

```text
Self-Play Games
      ↓
State → Game Result
      ↓
Value Network
```

### Step 4. Search

실제 게임에서는 학습한 Policy와 Value를 MCTS와 결합한다.

```text
Policy Network
       ↓
      MCTS
       ↑
Value Network
       ↓
   Next Move
```

---

## 8. Why is this Reinforcement Learning?

이 논문에서 흥미로운 점은
Supervised Learning과 Reinforcement Learning을 같이 사용한다는 것이다.

처음에는 인간의 데이터를 이용해

> 사람이 어떻게 두는가?

를 학습하고,

이후 Self-Play를 통해

> 어떻게 두어야 실제로 이기는가?

를 학습한다.

따라서 학습 목표가

```text
Human Move Prediction
```

에서

```text
Winning the Game
```

으로 바뀐다고 볼 수 있다.

---

## 9. Key Contribution

이 논문의 핵심은 단순히 "Deep Learning으로 바둑을 풀었다"는 것이 아니다.

개인적으로 중요한 부분은 다음 세 가지라고 생각한다.

### 1. Policy와 Value를 Neural Network로 학습

기존에 사람이 설계하던 바둑 평가 방법을
Neural Network가 데이터로부터 학습한다.

### 2. Supervised Learning + Reinforcement Learning

인간의 기보로 좋은 초기 Policy를 만들고,
Self-Play를 통해 이를 더 개선한다.

### 3. Learning + Search

Neural Network만 사용하는 것이 아니라
MCTS와 결합한다.

즉,

$$
\text{Learning} + \text{Search}
$$

라는 구조가 AlphaGo의 핵심이라고 볼 수 있다.

---

## 10. My Takeaway

이 논문을 읽으면서 가장 흥미로웠던 점은
Policy Network 자체가 AlphaGo의 전부가 아니라는 것이다.

처음에는 AlphaGo를 단순히

> 강화학습으로 바둑을 학습한 모델

정도로 생각했지만,

실제로는

$$
\text{Supervised Learning}
+
\text{Reinforcement Learning}
+
\text{Value Learning}
+
\text{Tree Search}
$$

를 결합한 시스템에 가깝다.

특히 학습된 Neural Network가 탐색 공간을 줄이고,
Search가 Neural Network의 판단을 보완한다는 점이 인상적이다.

---

## 11. Summary

AlphaGo의 전체 과정을 한 번에 정리하면 다음과 같다.

```text
Professional Go Games
         |
         v
Supervised Policy Network
         |
         v
      Self-Play
         |
         v
RL Policy Network
         |
         +--------------+
         |              |
         v              v
   Policy Network   Value Network
         |              |
         +------ MCTS ---+
                  |
                  v
              Next Move
```

한 줄로 정리하면,

> **AlphaGo는 인간의 기보에서 시작해 Self-Play로 Policy를 개선하고,
> Policy/Value Network와 MCTS를 결합하여 바둑의 다음 수를 결정한다.**

---

## Reference

Silver, D. et al.  
**Mastering the game of Go with deep neural networks and tree search.**  
Nature 529, 484–489 (2016).
