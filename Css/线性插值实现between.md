# between方法

> **场景如下：**
>
> 在一块1920px大小的的屏幕中要显示一个宽度为100px的div，但是这屏幕大小的可变的，变化范围为800px -> 1920px，而在屏幕大小发生变化时，div的宽度应该进行响应式变化，变化范围为20px -> 100px
>
> 而要在屏幕在变化范围中，动态计算div的宽度，就需要用到线性插值

## 线性插值

> 数学上定义:线性插值是指插值函数为一次多项式的插值方式,其在插值节点上的插值误差为0
>
> 在图片上，我们利用线性插值的算法，可以减少图片的锯齿,模糊图片
>
> 线性插值又可以分为单线性插值、又线性插值和三线性插值，单线性插值是在一个方向上进行线性插值,比如X方向

![img](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/v2-66e61db920c59a79a8991481d7f40c3b_720w.png)

假设我们已知坐标 (x0, y0) 与 (x1, y1)，要得到 [x0, x1] 区间内某一位置 x 在直线上的值。根据图中所示，我们得到：

![img](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/v2-9e5b3837639c61d6a7989a70f7bed359_720w.webp)

由于 x 值已知，所以可以从公式得到 y 的值:

![img](https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/v2-2dbc975691a8a83ed5dc6297d235751f_720w.webp)

已知 y 求 x 的过程与以上过程相同，只是 x 与 y 要进行交换

## 代码实现

```ts
// @flow
import getValueAndUnit from '../helpers/getValueAndUnit'
import PolishedError from '../internalHelpers/_errors'

/**
 * Returns a CSS calc formula for linear interpolation of a property between two values. Accepts optional minScreen (defaults to '320px') and maxScreen (defaults to '1200px').
 *
 * @example
 * // Styles as object usage
 * const styles = {
 *   fontSize: between('20px', '100px', '400px', '1000px'),
 *   fontSize: between('20px', '100px')
 * }
 *
 * // styled-components usage
 * const div = styled.div`
 *   fontSize: ${between('20px', '100px', '400px', '1000px')};
 *   fontSize: ${between('20px', '100px')}
 * `
 *
 * // CSS as JS Output
 *
 * h1: {
 *   'fontSize': 'calc(-33.33333333333334px + 13.333333333333334vw)',
 *   'fontSize': 'calc(-9.090909090909093px + 9.090909090909092vw)'
 * }
 */
export default function between(
  fromSize: string | number,
  toSize: string | number,
  minScreen?: string = '320px',
  maxScreen?: string = '1200px',
): string {
  const [unitlessFromSize, fromSizeUnit] = getValueAndUnit(fromSize)
  const [unitlessToSize, toSizeUnit] = getValueAndUnit(toSize)
  const [unitlessMinScreen, minScreenUnit] = getValueAndUnit(minScreen)
  const [unitlessMaxScreen, maxScreenUnit] = getValueAndUnit(maxScreen)

  if (
    typeof unitlessMinScreen !== 'number'
    || typeof unitlessMaxScreen !== 'number'
    || !minScreenUnit
    || !maxScreenUnit
    || minScreenUnit !== maxScreenUnit
  ) {
    throw new PolishedError(47)
  }

  if (
    typeof unitlessFromSize !== 'number'
    || typeof unitlessToSize !== 'number'
    || fromSizeUnit !== toSizeUnit
  ) {
    throw new PolishedError(48)
  }

  if (fromSizeUnit !== minScreenUnit || toSizeUnit !== maxScreenUnit) {
    throw new PolishedError(76)
  }

  const slope = (unitlessFromSize - unitlessToSize) / (unitlessMinScreen - unitlessMaxScreen)
  const base = unitlessToSize - slope * unitlessMaxScreen
  return `calc(${base.toFixed(2)}${fromSizeUnit || ''} + ${(100 * slope).toFixed(2)}vw)`
}
```

### 解析

1. 线性插值公式: `y = y1 + (x - x1) * (y0 - y1) / (x0 - x1)`

2. 变量`slop`即为斜率`(y0 - y1) / (x0 - x1)`，这些都为已知变量，因此斜率可以轻松求得

3. 而第一点的线性插值公式可以轻松的变化为如下等式：

   `y = y1 - slope * x1 + slope * x`

4. `base`即为`y1 - slope * x1`
5. 现在如果知道自变量`x`就可以求出`y`，但是实际场景`x`其实是动态变化的，他应该代表真实的屏幕宽度(但是要在`maxScreen - minScreen`之间)，而`100vw`就是真实屏幕宽度，并且他是响应式单位，刚好符合我们的需求
