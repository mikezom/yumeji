---
date: '2025-06-26T17:18:37+08:00'
draft: true
title: '2025 06 26 Board Game Reflection'
---

# 《游城拓荒》设计反思

## 数值设计流程

1. 确定游戏内经济模型以及玩家体验目标：我们希望通过数值设计引导玩家侧重性地执行探索以及建设操作，增加玩家的内容体验。游戏中的资源入口主要是探索以及采集，玩家通过建设将其资源转化为分数。

2. 根据设计目标，确定资源产出总量：我们需要**保证游戏内的资源产出比例符合预期**，游戏内的资源获取有初始资源、探索资源、采集资源。假设预期的资源量比例为1:3:5:7。已知有共有21个资源点，其中6个资源点T4才开放探索，我们可以算出玩家的预期资源总量。

3. 根据设计目标，确定资源消耗总量

## 落地后的反思

0. **规则书撰写：**规则书的可理解性严重不足。我们需要一个更好的验证框架，需要一些能真正进行测试的人。

1. **城市移动：**玩家的具体行为与预期玩家行为不匹配。设计时认为玩家对于干扰他人会有道德负担，但实际上玩家会经常尝试这一行为。城市移动对于其他玩家来说破坏性过于巨大。为了弥补这一点，可以给被移除的子的玩家一个一次性经济补偿（每被移除一个，需要给被移除子玩家交5块钱，拆迁款），以降低对于被干扰玩家的计划影响。

2. **部件大小：**玩家面板占地过大，在设计时疏于考虑。可以考虑把设施卡做成方形，或者令玩家面板上的设施可以相互重叠。