# 基本原理

## 价格的表示

假设有资产 X 和资产 Y，现在存在一个以 X 和 Y 资产组成的交易对池子。当我们要用资产 Y来标价 X 的时候

                     $$price_{X} = Y数量 / X数量$$

## 流动性公式

v2和v3都遵循 了一个基本公式

$$x * y = k$$

x表示一种token数量，y表示另一种token的数量。这个公式就满足了下面这个[流动性曲线图](https://github.com/Dapp-Learning-DAO/Dapp-Learning/raw/main/defi/Uniswap-V3/whitepaperGuide/img/understanding-01-pricechange.webp)

![](https://blog-1251725892.cos.ap-shanghai.myqcloud.com/20220602164323.png)




可以看出，所谓的流动性 ***k***代表的实际是绿色部分的面积，曲线本身就表示x的价格轨迹，很显然，用户做市的范围理论上是$$(0,\infty)$$，而现实中的价格是在一个小的多的有限范围内波动，所以v2中的资金利用率比较低。

为了解决这个问题，v3允许用户在提供流动性的时候指定要做市的价格区间， 而且满足上面的公式。

![](https://blog-1251725892.cos.ap-shanghai.myqcloud.com/20220602164404.png)




从上图看到，假如用户流动性指定的区间是[p_upper, p_lower]，实际用户提供的流动性只是黄色部分，用x,y分别代表用户实际出资的两种token数量，要满足 x * y = k, v3中创造了 

x_virtual和y_virtual的概念，用户实际添加的流动性我们用$$\Delta x$$和$$\Delta y$$来表示，使得:

                               $$(\Delta x + x_{virtual}) * (\Delta y + y_{virtual}) = k$$

很容易可以看出 x_virtual 的长度实际是 p_upper 点的 x，而 y_virtual 的长度实际上是 p_lower 点的 y。

# v3公式推导

## xy跟价格和流动性的关系

我们根据价格公式和流动性公式可以分别推导出x,y跟流动性和价格的关系：

                    $$\left\{ x * y = L^{2}, y = p * x\right\}$$

                                  $$\Downarrow$$

                    $$\left\{ x = L / \sqrt{p}, y = L * \sqrt{p}\right\}$$

v3中把流动性k 用$$L^{2}$$表示，x的价格$$p_{x}$$用 p 表示。

那么同样两个虚拟流动性就可以写为：

```plain
x_virtual = L / √p_upper
y_virtual = L * √p_lower
```
将x_virtual 和 y_virtual 带入$$(\Delta x + x_{virtual}) * (\Delta y + y_{virtual}) = L^{2}$$,可得核心公式。

## 核心公式

                   $$(\Delta x + L / \sqrt{p_{upper}}) * (\Delta y + L * \sqrt{p_{lower}}) = L^{2}$$

公式中 `p_upper` 和 `p_lower` 都是已知的变量，也就是在 V3 中添加流动性，用户自己设置的做市价格区间

# 添加流动性

添加流动性的计算过程，是已知当前价格和输入的其中一种资产数量，计算另一种资产的数量和添加的流动性数量的过程。

因为用户分区间设置流动性，计算过程很复杂，需要分为三种情况考虑

## 当前价格在区间内

在区间内，即

###  p_lower < p < p_upper

![](https://blog-1251725892.cos.ap-shanghai.myqcloud.com/20220602164441.png)




这种情况下，橙色区域是我们实际需要添加的流动性，x和y分别是当前价格下两种资产的数量。

```plain
delta x = x - x_virtual 
delta y = y - y_virtual
```
 根据[xy和价格流动性公式](https://shimo.im/docs/vVqRVyjZjbUR9Mqy#anchor-N9LC)， 我们可以将公式写成这样 
```plain
delta x = L / √p - L / √p_upper = L * (√p_upper - √p) / (√p * √p_upper)
delta y = L * √p  - L * √p_lower = L * (√p - √p_lower)
```

再变换一下，改写成求 L（流动性数量）的等式

```plain
L = delta x * (√p * √p_upper) / (√p_upper - √p)
L = delta y / √(p - p_lower)
```

当我们添加流动性时，指定任何一种资产数量和价格区间，都能个算出流动性 `L` ,再根据另一个公式，计算出另一种资产数量。

## 当前价格在区间外

在区间外又分为两种情况

### p > p_upper

![](https://blog-1251725892.cos.ap-shanghai.myqcloud.com/20220602164525.png)


此时橙色区域已经消失，资产 X 的数量为 0，流动性全部变为资产 Y，其数量就是 `p_upper`  到  `p_lower` 的 y 轴距离，也就是图中的紫色虚线部分。我们根据 delta y 求得 L : 

```plain
L = delta y / √(p_upper - p_lower)
```
### p < p_lower

![](https://blog-1251725892.cos.ap-shanghai.myqcloud.com/20220602164539.png)


此时橙色区域已经消失，资产 Y 的数量为 0，流动性全部变为资产 X，其数量就是 `p_upper` 到 `p_lower` 的 x 轴距离，也就是图中的蓝色虚线部分。我们根据 `delta x` 求得 `L` :

```plain
L = delta x * (√p_upper * √p_lower) / (√p_upper - √p_lower)
```
## 总结

V3 的流动性计算过程是需要先确定价格范围 `p_upper` `p_lower`、当前价格 `p` 和其中一种资产的数量 `delta x`或 `delta y`，求流动性数量 `L` 和另一种资产的数量。 

# 移除流动性

移除的过程实际上是上述添加过程的逆运算，也是分三种情况。

# 交易

## 价格点Tick

V3中用户设定的价格区间不是任意的，必须处在tick上， tick的取值范围是 `int24` 有符号。每个tick和价格是一一对应的，对于索引是 `i` tick，价格与之的关系是：

```plain
p(i)=1.0001^i
```
 因为v3中在实际计算过程中使用的不是价格 p 而是根号价格 $$\sqrt{p}$$,所以公式调整为

                                    $$\sqrt{p_{i}} = \sqrt{1.0001^{i}} = 1.0001^{i/2}$$

所以v3的价格范围也是一个足够宽广的价格区间。

知道tick的索引可以计算出价 `p` ，反之知道 `p` 也可以推算出tick。

## 费率和tickspacing

不同的交易对，波动幅度不同，像稳定币的波动区间很小，而山寨币的波动幅度一般都很大，由于波动非常小，交易产生的价差也不可能太大，如果收取 0.3% 或更高的手续费是不合适的，不利于市场的流动性，而两个山寨币的交易对，由于波动幅度非常大，其产生的价差也可能非常大，即使是 1% 的手续费也有很多人愿意执行交易。所以v3中，在添加流动性的时候可以选择费率，目前有三档

另外一点是tick的可选择范围是很大的，而且tick价差可能很小，过密的价差对那些波动大的交易对是完全没必要的，所V3中引入了 `tickspacing` 的概念，它表示tick的疏密程度，在两个tickspacing之间的tick可以忽略，不参与计算。

v3将费率和tickspacing进行结合，做出如下的绑定关系：

|费率|tickspacing|建议的使用范围|
|:----|:----|:----|
|0.05%|10|稳定币交易对|
|0.3%|60|适用大多数交易对|
|1%|200|波动极大的交易对|

## tick结构

```plain
pub struct TickState {
    /// Amount of net liquidity added (subtracted) when tick is crossed from left to right (right to left)
    pub liquidity_net: i64,
    /// The total position liquidity that references this tick
    pub liquidity_gross: u64,
    /// Fee growth per unit of liquidity on the _other_ side of this tick (relative to the current tick)
    /// only has relative meaning, not absolute — the value depends on when the tick is initialized
    pub fee_growth_outside_0_x32: u64,
    pub fee_growth_outside_1_x32: u64,
    ...
}
```
### liquidity_net

liquidity_net的定义是：当价格从左到右(y->x)穿过该tick时，激活的流动性需要变化的数量；如果价格穿过tick的方向是从右到左(x->y)，那么变化量就是liquidity_net取反。

规定liquidity_net的数值在下面三种情况会发生相应的变动：

1. 增加流动性，$$\Delta L_{1}（正数）$$
    * tick作为区间下限，那么 liquidity_net = liquidity_net +$$\Delta L_{1}$$（价格的运行方向是从左到右进入一个流动性区间，应该加上该区间的流动性数量）
    * tick作为区间上限，那么 liquidity_net = liquidity_net -$$\Delta L_{1}$$（价格的运行方向是从左到右离开了一个流动性区间，把离开区间的流动性减掉)
2. 减少流动性$$\Delta L_{2}（正数）$$
    * tick作为区间下限，那么 liquidity_net = liquidity_net -$$\Delta L_{2}$$（把原来加的减去一部分)
    * tick作为区间上线，那么 liquidity_net = liquidity_net +$$\Delta L_{2}$$（把原来减的加回来一部分）
3. 清空tick
    *  liquidity_net = 0
从上面可以看出增加流动性和减少流动性对liquidity_net的改变是相反的过程。最后liquidity_net的值就是不同的流动性区间引用tick时的流动性数量的净值（每个tick都可能是区间上限或者下限，或者兼具两种角色）。

那这里我们思考一个问题：假设初始有两个用户在（a,b）和（b，c）两个区间分别添加了 100的流动性，此时tick_b作为（a,b）区间的价格上限liquidity_net = -100，同时又作为（b，c）的价格下线，所以更新tick_b的liquidity_net_new = liquidity_net_old + 100 = 0，那么怎么证明b是被引用的呢？ 所以就用到了`liquidity_gross` 这个变量

### liquidity_gross

liquidity_gross表示一个tick上流动性的累加值。 引用一个tick，即把该tick作为流动性的边界，tick就加上相应的流动性；如果tick没有被引用，那么该值为0，以此作为tick是否初始化的判断条件，该值随下面三种情况变化：

* 引用该tick，增加流动，liquidity_gross变大
* 引用该tick，减少流动性，liquidity_gross变小
* 清空tick，liquidity_gross变为0
作用： 判断tick是否初始化，目前没有其他用途

## tick位图

tick 位图用于记录所有被引用的 lower/upper tick index，我们可以用过 tick 位图，从当前价格找到下一个（从左至右或者从右至左）被引用的 tick index

tick 位图有以下几个特性：

* 对于不存在的 tick，不需要初始值，因为访问 map 中不存在的 key 默认值就是 0
* 通过对位图的每个 word(uint256) 建立索引来管理位图，即访问路径为 word index -> word -> tick bit
# 手续费的计算

## 区间内手续费公式

假如交易在一个价格区间内进行，手续费会在区间内不断累加，但是区间外是一直不变的。 所以区间内的手续费，等于总的手续费减去区间外的手续费，即：

```plain
feeGrowthInside = feeGrowthGlobal - feeGrowthOutside
```
假如交易在一个 (a,b) 的价格区间内进行
|.|.|.|.|.|.|.|.|
|:----|:----|:----|:----|:----|:----|:----|:----|
|...|    |a|  i|b|    |    |...|

在(0, a) 和 (b, ∞) 区间内的手续费并没有增加，想要计算 (a,b) 区间内的手续费，其实只要用池子内所有流动性赚取的总手续费数量减去(0, a) 和 (b, ∞) 这两个区间的区手续费就行了。我们用 `freeGrowthOutside_below` 表示(0, a)区间内的手续费， `freeGrowthOutside_above` 表示(b, ∞)区间内的手续费

间两边外侧的

总结一下公式：

```plain
feeGrowthInside = feeGrowthGlobal - feeGrowthOutside_below - feeGrowthOutside_above
```
## 计算(a,b)区间外手续费

 tick 上有一个 `feeGrowthOutside` 变量，记录该 tick 作为流动性边界时，区间外的手续费总量。 这里的区间外是相对于当前价格tick（i）位置而言的。

我们规定：

* 当  `i`  在区间边界tick的左侧时，那么边界tick中`feeGrowthOutside` 表示的是它右侧的手续费
* 当  `i`  在区间边界tick的右侧时，那么边界tick中`feeGrowthOutside` 表示的是它左侧的手续费
总结起来就是：

边界tick的`feeGrowthOutside` 变量表示的是它相对当前价格tick位置的反方向的值。

要计算（a, b）区间的手续费， 分情况分析下面三种情况：

### `i < a` 

|.|.|.|.|.|.|.|.|
|:----|:----|:----|:----|:----|:----|:----|:----|
|...|i|a|  |b|    |    |...|

当前价格的tick索引 `i` 在  `a` 的左侧 ， a中`feeGrowthOutside` 表示右侧的手续费，那么

```plain
feeGrowthOutside_below = feeGrowthGlobal - feeGrowthOutside_a
```
同时，i在b点的左侧，b中`feeGrowthOutside` 表示右侧的手续费 
```plain
feeGrowthOutside_above = feeGrowthOutside_b
```
最后计算（a,b）区间内的手续费，代入[公式](https://shimo.im/docs/vVqRVyjZjbUR9Mqy#anchor-kLp3)，得到：
```plain
feeGrowthInside_ab = feeGrowthOutside_a - feeGrowthOutside_b
```

### a < i < b

|.|.|.|.|.|.|.|.|
|:----|:----|:----|:----|:----|:----|:----|:----|
|...|    |a|  i|b|    |    |...|

当前价格的tick索引 `i` 在  `a` 的右侧 ， a中`feeGrowthOutside` 表示它左侧的手续费，那么

```plain
feeGrowthOutside_below = feeGrowthOutside_a
```
同时，i在b点的左侧，b中`feeGrowthOutside` 表示它右侧的手续费 
```plain
feeGrowthOutside_above = feeGrowthOutside_b
```
最后计算（a,b）区间内的手续费，代入[公式](https://shimo.im/docs/vVqRVyjZjbUR9Mqy#anchor-kLp3)，得到：
```plain
feeGrowthInside_ab =  feeGrowthGlobal -feeGrowthOutside_b - feeGrowthOutside_a
```

### `i > b` 

|.|.|.|.|.|.|.|.|
|:----|:----|:----|:----|:----|:----|:----|:----|
|...|    |a|  |b|i|    |...|

当前价格的tick索引 `i` 在  `a` 的右侧 ， a中`feeGrowthOutside` 表示左侧的手续费，那么

```plain
feeGrowthOutside_below = feeGrowthOutside_a
```
同时，i在b点的右侧，b中`feeGrowthOutside` 表示左侧的手续费 
```plain
feeGrowthOutside_above = feeGrowthGlobal - feeGrowthOutside_b
```
最后计算（a,b）区间内的手续费，代入[公式](https://shimo.im/docs/vVqRVyjZjbUR9Mqy#anchor-kLp3)，得到：
```plain
feeGrowthInside_ab = feeGrowthOutside_b - feeGrowthOutside_a
```

从上面的三中情况可以看出： 一旦 i 穿过 a, b两个tick, 他们的feeGrowthOutside的值就要换个方向



参考： 

[uniswapV3白皮书解读](https://github.com/Dapp-Learning-DAO/Dapp-Learning/blob/main/defi/Uniswap-V3/whitepaperGuide/understandV3Witepaper.md)

 


