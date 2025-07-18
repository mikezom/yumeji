---
date: '2025-06-27T00:15:42+08:00'
draft: false
title: '2025 06 27 数值笔记'
---

## Markov Chain Problems

### 问题1：解谜时间

你面前有3扇门，

- 第一扇门，进入概率25%，进入后消耗 5 分钟返回。
- 第二扇门，进入概率25%，进入后消耗 10 分钟返回。
- 第三扇门，进入概率50%，进入后消耗 6 分钟后离开。

求离开的期望时间。

---

解：

进入第一第二扇门后，都会回到初始状态。也就是说返回后依然需要花费期望时间 $E$ 来离开。所以我们可以列出一下等式：

$$ E = 0.25 (5 + E) + 0.25 (10 + E) + 0.5 * 6 $$

求得

$$ E = 13.5 $$

### 问题2：暴击预期打击数

你面前有1个100血的小怪，你每次击打它时：

- 50% 造成1点伤害。
- 50% 造成2点伤害。

请问期望需要多少次击杀？

---

解：

这是一个Markov Chain的Mean Absorbing Time问题

假设我们有101个状态，其对应的预期变幻次数为 \(\mu_i\),

易得 

$$
\begin{aligned}
\mu_0 &= 0 \\
\mu_1 &= 1 \\
\mu_n &= 1 + 0.5 \mu_{n-1} + 0.5 \mu_{n-2}
\end{aligned}
$$

接下来我们需要将其转换为通项公式，首先利用待定系数法构造等差数列：

$$
(\mu_n + a \mu_{n-1} ) - b (\mu_{n-1} + a \mu_{n-2}) = 1
$$

解得

$$
\begin{aligned}
a &= 0.5 \\
b &= 1
\end{aligned}
$$

即
$$
(\mu_n + 0.5 \mu_{n-1} ) - (\mu_{n-1} + 0.5 \mu_{n-2}) = 1
$$

因为 \(\mu_1 - 0.5 \mu_0 = 1\)，所以我们有
$$
\mu_n + 0.5 \mu_{n-1}  = n
$$

接下来我们利用待定系数法构造等比数列\(g_n\)：

$$
\begin{aligned}
g_n &= \mu_n + An + B\\
g_{n-1} &= \mu_{n-1} + A(n-1) + B \\
&= -2(\mu_n - n) + A(n-1) + B\\
&=-2\mu_n + (2 + A)n + (-A +B)\\
&= -2(\mu_n + An + B) + (3A +2)n + (-A + 3B)
\end{aligned}
$$

解得
$$
\begin{aligned}
A &= - \frac{2}{3} \\
B &= - \frac{2}{9} \\
g_n &= \mu_n - \frac{2}{3}n - \frac{2}{9}
\end{aligned}
$$

由\(\mu_0 = 0\)与等比数列性质可得
$$
\begin{aligned}
\mu_n = \frac{2}{3}n + \frac{2}{9} - \frac{2}{9} (-\frac{1}{2})^n
\end{aligned}
$$

代入\(n = 100\)得
$$
\mu_{100} = 66\frac{8}{9} - \frac{2}{9}(-\frac{1}{2})^{100}
$$

---

扩展：设玩家面对一个血量为\(H\)的敌人，非暴击时造成\(1\)点伤害，暴击时造成\(k, k \in \mathbb{N}\)点伤害，暴击概率为\(p\)求期望打击数？

可以使用蒙特卡洛解决实际问题。

```python
import numpy as np
def expected_hits(enemy_hp: int, player_damage: list[int], player_damage_probability: list[int]) -> float:
    TEST_SIZE = 1E7
    hit_total = 0
    for _ in range(TEST_SIZE):
        damage_dealt = 0
        hit = 0
        while damage_dealt < enemy_hp:
            hit += 1
            damage_dealt += np.random.choice(player_damage, 1, player_damage_probability)
        hit_total += hit
    return hit_total / TEST_SIZE
```

## --