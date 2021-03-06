---
id: learn-randomness
title: 随机性
sidebar_label: 随机性
---

权益证明(PoS)区块链的随机性对于验证人责任的公平并且不可预测的分配是非常重要。 ，因为计算机是确定性设备，所以它们在随机数方面很差(同样输入总是会产生同样输出)。大部份人通常在计算机上(例如游戏应用程序中)称随机数实际上是_伪随机_- 也就是说它们依赖用户或其他类型的_Oracle_ 提供足够随机的_种子_，例如[气象站的大气噪声](https://www.random.org/randomness/), [心跳率](https://mdpi.altmetric.com/details/47574324)或什至[熔岩灯](https://en.wikipedia.org/wiki/Lavarand)，就可以从中产生一系列看似随机的数字。但是如果给相同的种子，结果将会是一样。

这些输入将根据时间和空间而变化，但是不可能将相同的结果输入到全球特定区块链的所有节点中。如果节点获得不同的输入以创建区块，则会发生分叉。显然现实世界中的熵不适合用作区块链随机性的种子。

至今区块链有二种可用的随机性方法: RANDAO 和 VRF。 Polkadot 使用 VRF。

## VRF

可验证随机函数(VRF)是数学运算，需要一些输入并产生随机数以及该提交者生成该随机数的真实性证明。任何挑战者都可以验证该证明，以确保随机数生成有效。

Polkadot 中使用的 VRF 与 Ouroboros Praos 中的大致相同。 Ouroboros 随机性对于块生产来说是安全并对 BABE 来说效果很好。它们不同之处在于 Polkadot 的 VRF 不依赖于中央时钟(问题变了-谁的中央时钟?)，而是取决于它自己过去结果来确定现在和将来的结果，并且它使用插槽数字作为时钟模拟器估计时间。

具体操作如下:

时隙长度为 6 秒的时间单位。每个时隙可以包含一个区块，但可以不包含区块。时隙构成了时期(epochs) - 在 Kusama 中，2400时隙构成一个时期，这使一个时期长达四个小时。

每个时隙中，每个验证人 "掷骰子"。它们执行以下内容作为输入的函数(VRF):

- **"密钥"** - 专为掷骰子而制成的密钥，在每个新时隙中再生。
- **来自上一个(N-2)之前的时期中各个区块 VRF 值的哈希值**，因此过去的随机性会影响当前待处理的随机性( N)。
- **时隙号**

![](assets/VRF_babe.png)

输出为两个数值: ` RESULT `(随机值)和` PROOF `(证明随机数值已正确生成的证明)。

The `RESULT` is then compared to a _threshold_ defined in the implementation of the protocol (specifically, in the Polkadot Host). If the value is less than the threshold, then the validator who rolled this number is a viable block production candidate for that slot. The validator then attempts to create a block and submits this block into the network along with the previously obtained `PROOF` and `RESULT`.

钓鱼人 - 监视网络做坏事的收集人和验证人节点 - 验证中继链區块。 由于非法掷骰将产生非法区块，并且由于钓鱼人将在验证人产生的每个区块中访问` RESULT `和` PROOF `，因此对它们而言，很容易自动报告作弊的验证人。

总结: 在 VRF 中，每个验证人都会为自己掷出一个数字，并根据阈值对其进行检查，如果随机掷骰低于该阈值，则会生產區块。 钓鱼人监察网络并报告不良行为验证这些掷骰的有效性，并向系统报告任何作弊行为(例如尽管掷出的人数超过阈值，但有人假装成块生产區塊者)。

精明的读者会注意到，这种工作方式某些时隙可能没有验证人作为区块链生产者候选者，因为所有验证候选者的得分都太高而错过了阈值值。我们在[ wiki 共识页面](learn-consensus)中说明了如何解决此问题，并确保 Polkadot 出块时间保持在恒定时间。

## RANDAO

[ RANDAO ](https://github.com/randao/randao) - RANDAO 要求每个验证人通过对某些种子执行一系列哈希操作来进行准备。 验证人然后在一个回合中发布最终的哈希值，加上随机数是从每个参与者进入游戏中得出。只要有一名诚实的验证人参加，随机性就被认为是安全（在经济上进行攻击是不可行）。Polkadot 不选用 VRF 的随机性方法是因为从每个区块生产者处揭示每个时隙的哈希值需要二次带宽或至少二次计算。

RANDAO is optionally augmented with VDF.

### VDFs

[可验证延迟函数(VDF)](https://vdfresearch.org/)是即使在并行计算机上也需要花费一定时间才能完成计算。它们产生独特的输出，可以在一般配置独立和有效地对其进行验证。通过将 RANDAO 的结果输入到VDF 中引入延迟，从而使任何攻击者企图影响当前随机性的尝试都将过时。

VDFs 可能会通过需要与其他类型的节点分开运行 ASIC 设备来实现。 虽然只有一个足以保证系统的安全和它们将是开源并且几乎免费提供，但是运行它们并不便宜也没有激励，所以选择此方法的区块链用户，产生了不必要的麻烦。

## Resources

- [Polkadot's research on blockchain randomness and sortition](https://research.web3.foundation/en/latest/polkadot/BABE/Babe.html) - contains reasoning for choices made along with proofs
- [关于 Polkadot 中使用随机性的讨论](https://github.com/paritytech/ink/issues/57) - W3F 研究人员讨论了 Polkadot 中的随机性，何时用以及在哪些假设下进行。
