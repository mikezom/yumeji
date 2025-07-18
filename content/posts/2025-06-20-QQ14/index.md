---
date: '2025-06-23T19:39:50+08:00'
draft: false
title: 'QQ14经验值随笔'
---

如果要我去填QQ14的经验值曲线，我会怎么填？

## 基于玩家游戏时间的经验值设计思路

一切从玩家游戏时间开始。玩家在QQ14的游戏时间主要由副本与任务构成。对于每名玩家的第一个职业而言，玩家的前期时间完全由任务构成；中期由完成任务为主导，攻略副本为辅助；后期则由攻略副本为主导，完成任务为辅助。

我们的经验公式大致如下所示：

$$升级所需经验值(等级) = \frac{\sum{该等级主线任务经验}}{主线任务占该等级游玩体验比(等级)}$$

$$单个任务经验 = 等级 \times 任务完成时间$$

### 可能面临的问题以及其他细节

#### 1. 开风脉、开地图、击杀小怪等杂项经验如何填写？

**这些经验的填写核心与预期副职业的升级效率与第一职业的升级效率比有关。** 玩家主要会在提升副职业等级时利用到这些杂项经验，所以若我们希望玩家提升副职业等级时的效率为主职业的1/4，那我们应该把这些杂项经验的公式后乘上1/4。我们预期玩家在不采用一次性经验时提升副职业等级所花费的总时间为提升主职业时的4倍。

我们将杂项经验按照其可获取数量分为两类：

- 一次性经验（探索）
- 可重复刷取经验（战斗、fate等）

这些经验应该与预期玩家获取时的等级以及其在玩家经验构成中的占比相关，而具体的经验计算应该与预期玩家行为相关。而这些杂项经验的经验获取效率应该比玩家正常完成主线要低很多，其中可重复刷取经验会更低。

举例来说，探索经验与探索行为相关。玩家通过在大世界花费时间移动来探索，花费时间越多理应获取经验越高，所以探索经验应该与最短移动时间成正比。由于玩家可以在大地图中任意切换职业，我们希望避免玩家切换低级职业领取高级地图探索奖励的情况，所以获取的探索经验应该和玩家探索时采用的职业等级成正比。最后，我们需要乘上相比副职业的升级效率常数更高的常数（因为一次性的资源更加稀缺）。得到如下公式：

$$单区域探索经验 = 玩家目前等级 \times 最短移动时间 \times 0.5$$

#### 2. 攻略副本的经验具体如何填写？

攻略副本的经验获取效率应低于完成主线的经验获取效率。我们预期玩家提升副职业时的主要经验获取为攻略副本。以玩家游戏时间为基础，我们可以很容易的得到以下公式：

$$副本总经验 = 预期玩家等级 \times 预期完成时间 \times 0.25$$

而日常任务的获取效率会是更高的，需要将最后的常数提高。

#### 3. 主线任务占该等级游玩体验比如何设计？

我们将玩家的战斗过程分为以下几个阶段：

- 熟悉游戏阶段
- 适应游戏阶段
- (高难挑战阶段)

玩家的等级提升应该在熟悉游戏阶段到适应游戏阶段之间。所以玩家的游玩体验中，随着等级的逐渐提升，主线任务的占比应该逐渐降低。对于游戏前期而言，我们希望玩家升级的门槛尽可能低，以增加玩家留存率。所以对于1级的玩家，主线任务的占比可以被设为100%。对于最后一级而言，主线任务的占比应该根据玩家主线所有副本经验计算得到。计算的目的是玩家完成全部主线任务并第一遍通关主线副本可以获得足够的经验升至满级。而中期的经验占比可以使用线性函数获取。

#### 4. 没有主线的等级？

在QA过程中，我们需要避免以下情况：玩家在做出了必要最少程度的经验获取，之后在某一个等级无法通过主线和必要的副本获取足够量的经验接取下一级的主线。这一情况很可能会打破玩家的心流，从而降低玩家的留存概率。

#### 5. 经验值的观感？

基于前文中提到的经验值计算，我们有可能会面临经验值需求不随着等级增长的情况。当一个等级中的主线任务耗时过长，便可能会诱发这种情况。我们可以通过和策划沟通修改部分主线任务的预期等级来解决这个问题，也可以通过计算最后规范化数值来解决这个问题。前者是一个更好的解决方案，后者在实现过程中则需要注意修改其他经验获取途径的经验值获取。
