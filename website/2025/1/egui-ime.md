# 记一次 Linux 下 egui IME 问题

日期：2025.1.15

系统相关信息：Archlinux, 6.12.9-arch1-1, plasma 6.2.5

## IME 是什么？

IME 是 Input Method Editor（输入法编辑器）的缩写，它是一种软件组件，用于帮助用户输入那些无法直接通过键盘输入的字符或文字（例如中文、日文、韩文等复杂字符集）。

一句话：IME 就是输入法。

## 发生什么了？

在 egui 0.30.0 中无法通过输入法输入文字。

注：egui 默认字体不含中文，中文应该显示为方框。然而，现在的结果是只要使用输入法，你不能输入任何内容。

## 原因

### 查看 Github 的相关 Issues 和 PR

查看 Github 后，发现：

许多 Issue 中也提到了 IME 的问题，但只要我们细心一点：

[CHANGELOG.md](https://github.com/emilk/egui/blob/366900c55059e121093a61ed6184e764a68d67f3/crates/egui-winit/CHANGELOG.md?plain=1#L13-L14)

> **0.29.1 - 2024-10-01 - Fix backspace/arrow keys on X11**
>
> * Linux: Disable IME to fix backspace/arrow keys [#5188](https://github.com/emilk/egui/pull/5188) by [@emilk](https://github.com/emilk)

原来是被禁用了啊，这又是为什么呢？

继续看，发现是上游的 winit 出问题了：[Issue](https://github.com/rust-windowing/winit/issues/2498)

AI 总结了一下：X11 平台会自动发出 Ime::Disabled 事件，即使未显式启用 IME，而其他平台可能不会。

### 源码

再看一看 egui 中相关的[源代码](https://github.com/emilk/egui/blob/366900c55059e121093a61ed6184e764a68d67f3/crates/egui-winit/src/lib.rs#L335-L374)：

```rust
// 前面的代码已省略
WindowEvent::Ime(ime) => {
  // on Mac even Cmd-C is pressed during ime, a `c` is pushed to Preedit.
  // So no need to check is_mac_cmd.
  //
  // How winit produce `Ime::Enabled` and `Ime::Disabled` differs in MacOS
  // and Windows.
  //
  // - On Windows, before and after each Commit will produce an Enable/Disabled
  // event.
  // - On MacOS, only when user explicit enable/disable ime. No Disabled
  // after Commit.
  //
  // We use input_method_editor_started to manually insert CompositionStart
  // between Commits.
  match ime {
    winit::event::Ime::Enabled => {
      if cfg!(target_os = "linux") {
        // This event means different things in X11 and Wayland, but we can just
        // ignore it and enable IME on the preedit event.
        // See <https://github.com/rust-windowing/winit/issues/2498>
      } else {
        self.ime_event_enable();
      }
    }
    winit::event::Ime::Preedit(text, Some(_cursor)) => {
      self.ime_event_enable();
      self.egui_input
       .events
       .push(egui::Event::Ime(egui::ImeEvent::Preedit(text.clone())));
    }
    winit::event::Ime::Commit(text) => {
      self.egui_input
       .events
       .push(egui::Event::Ime(egui::ImeEvent::Commit(text.clone())));
      self.ime_event_disable();
    }
    winit::event::Ime::Disabled | winit::event::Ime::Preedit(_, None) => {
      self.ime_event_disable();
    }
  };
}
//后面已省略
```

可以看到，`if cfg!(target_os = "linux")` 后面是空的，即 linux 的 IME 输入被禁用了……吗？

并没有，注意到 `//ignore it and enable IME on the preedit event.`，这一行注释告诉我们，不会被禁用，preedit 会启用 IME，只有 `Ime::Enabled` 被禁用。

那为什么不可以输入中文呢？

### 本地源码

实际上，0.30.0 已经是 crate 上最新的版本了，然而，egui 作者并未发布新的修补版本。

所以我现在得手动将本地的相关源码修改，并重新编译。

修改后，问题解决。

## 后记

其实这是一个小问题，删除 IME 支持和重新启用 IME 的支持的提交都是两个星期前合并的，所以再过一段时间 egui 发布新版本后这个问题就解决了。
