# 关于 Update 和 FixedUpdate 的一些看法

最近在学 Bevy，系统的更新方式有 Update 和 FixedUpdate 两种（还有其他的），Update 是每帧调用一次，而 FixedUpdate 是固定时间间隔调用一次。

那么什么时候使用 Update，什么时候使用 FixedUpdate 呢？

这里给出一些个人观点（注：本文参考资料来源于 Unity，希望和 Bevy 差别不大）。

## 使用 Update

Update 是每帧调用一次，因此它的调用频率是变化的，取决于帧率。

我们可以用 Update 处理几乎任何的事情，比如：读取输入、检测碰撞、移动物体……

以下是一个伪代码：

```rust
// 每帧运行
fn update(&mut self) {
  if pressed("A") {
    // 假设平面直角坐标系
    self.x -= 1.;
  }
  //...
}
```

但设备的帧率不是常量，因此 Update 的调用频率也是变化的。所以如果你帧率不稳定，或者在使用性能比较弱的设备，你的速度可能会变化。

当然，这早就有了解决方案：

```rust
fn update(&mut self, time: Time) {
  const SPEED: f32 = 1.;
  if pressed("A") {
    // 距离上一帧运行过去了多久
    let delta = time.delta_seconds();
    // 速度 * 时间 = 路程
    self.x -= SPEED * delta;
  }
  //...
}
```

现在，即使你帧率很低，速度也不会变慢，你在相同时间内的移动距离是固定的。

当然，帧率过低你也可能出现闪现的景象。

但你还是会触碰到一些 bug，比如：[一个比较原神帧率对攻速影响的视频](https://www.bilibili.com/video/BV1fZ4y1k77i/)，使用 Update 时你应该注意这些细节。

## 使用 FixedUpdate

### 理想情况下

在理想情况下，FixedUpdate 的调用频率是固定的。

所以你当然可以这么写：

```rust
// 每固定时间运行
fn fixed_update(&mut self) {
  if pressed("A") {
    self.x -= 1.;
  }
  //...
}
```

但我们基本不会这样写，因为这是每固定时间检测一次是否正在按键，如果你在中途按键或或者是松键，有可能会漏掉一些操作。

### 并不理想的情况下

但是，FixedUpdate 的调用频率并不是固定的，它的原理是在每一次 Update 前决定是否应该调用 FixedUpdate，从而在一个比较长的时间中，调用频率是固定的。

> 例如，如果你将 FixedUpdate 设置为每秒执行 120 次，而帧率为 60 FPS，它将尝试每帧中运行两次。
>
> 如果你只有 30 FPS，它将尝试每帧中运行四次。
>
> 如果你有 120 FPS，它将尝试每帧运行一次。
>
> 如果你有 240 FPS，它将每隔一帧运行一次。

更糟糕的是，如果你的 FixedUpdate 的总运行时间过长，将会导致以下连锁反应：

1. 帧率下降，每帧间隔时间变长。
2. 每帧需要运行的 FixedUpdate 次数增加。
3. FixedUpdate 的总运行时间进一步延长。
4. 形成恶性循环，每帧渲染时间不断增加。
5. 最终可能导致游戏完全冻结。

当然，Unity 做了一些优化防止游戏冻结——但这又会导致你的 FixedUpdate 并不 Fixed（）

所以，你应该使你的帧率超过你的 FixedUpdate 的调用频率，且控制好 FixedUpdate 的调用时间。

## 所以我们应该怎么选择呢？

### Update

你可以在 Update 中做任何几乎事情，只是出于性能考虑（或者是一些其他因素），才需要将一部分事件放到 FixedUpdate 中。

### FixedUpdate

1. 更新不需要频繁更新的内容，而不是每帧都更新（如自动寻路）。
2. 检测物理事件（如碰撞检测），但要注意好频率，不然可能负载过高，或者是穿墙。
3. 网络同步事件。
4. 精确物理模拟。

等等。

## 参考资料

本文使用的参考资料都是使用 Unity 的，如果有什么错误，欢迎指出。

1. [The truth about FixedUpdate()](https://discussions.unity.com/t/the-truth-about-fixedupdate/530594/15)
2. [How to use Fixed Update in Unity](https://gamedevbeginner.com/how-to-use-fixed-update-in-unity/#how_fixed_update_works)
