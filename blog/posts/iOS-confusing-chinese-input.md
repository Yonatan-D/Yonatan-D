# React 受控组件与 iOS 输入法的冲突及解决方案

{docsify-my-updater date:2025-12-19}

## 问题描述

用户反馈在 iOS 上不能中文输入，打完拼音点候选词不能填入，且拼音自动叠加。

经排查，问题原因如下：

1. 字符叠加：iOS 原生键盘拼音输入法兼容性问题，onChange 事件拿到的是拼音，数据处理后 setState 值，导致输入不正确。如输入 "gh" 时，输入框错误地显示为 "ghgh"。

2. 长度限制：输入框最大长度为 6。iOS 原生键盘在中文输入未选词时会先将拼音填入，当拼音长度超过 6 时，选词功能失效。

## 最小可复现代码：

```jsx
/*react*/
<desc>
### 复现步骤

1. 使用 iOS 原生键盘的中文输入法

2. 受控组件的 onChange 事件中异步设置 value，并且 value 的值是过滤空格的
</desc>
<script>
  export default class LimitedInput extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
        value: '',
        changedValue: '',
      }
    }

    handleChange = (e) => {
      this.setState({
        value: e.target.value.replace(/\s/g, ''), // 过滤空格
        changedValue: e.target.value
      })
    }

    render() {
      return (
        <div>
          <p>
            changedValue: <span>{this.state.changedValue}</span>
          </p>
          <input
            type="text"
            maxLength={6} // 限制文本长度为 6
            value={this.state.value}
            onChange={this.handleChange}
          />
        </div>
      );
    }
  }
</script>
```

## 原因

受控组件对 onChange 事件的响应时机 与 [输入法 (IME)](https://developer.mozilla.org/zh-CN/docs/Glossary/Input_method_editor) 的输入行为 存在冲突。

关键触发条件：此问题仅在 `拼音输入过程会实时填充输入框` 的输入法中出现。iOS 原生键盘和 Arch Linux 下的 Fcitx5 (Rime) 均属于此类，因此都能触发 Bug（具体表现略有不同）。而搜狗等输入法在输入拼音时，候选词悬浮于输入框之上，不会触发此问题。

中文输入法在输入拼音时，会频繁触发 onChange 事件，这并非 React 特有问题（可参考社区讨论 [Change event fires extra times before IME composition ends](https://github.com/facebook/react/issues/3926)）。

iOS 原生键盘在输入拼音时，会直接将拼音字母（如 nihao）填入输入框，并用空格分隔音节。当我们的受控组件在 onChange 回调中（例如，为了过滤非法字符）处理了这些中间状态的拼音时，就破坏了输入法预期的状态，导致无法正确分词和选词。输入框内容叠加问题，也是因为对中间状态的错误处理所致。

有两种解决方案：

一是不使用受控组件。提交时再获取输入框的值做校验提示。

二是兼容这类输入法。在输入法预期状态改变时，不处理输入框内容，而是等到输入法实际完成输入时再处理。具体就是 onChange 事件中判断是否处于输入法预期状态，如果是，则不做数据处理，直接 setState，再在 onCompositionEnd 事件中对最终结果进行处理。(参考 @hec9527 的回答：https://segmentfault.com/q/1010000018185771)

为了尽量减少对现有代码的改动，我选择第二种方案。

<text class="warn">

**受控输入框与非受控输入框**

受控输入框：通过 value 属性绑定值，onChange 事件处理用户输入，setState 更新 value 属性，从而控制输入框的值

非受控输入框：不传 value，不控制输入框的值，因此不会触发这个 Bug

</text>

<text class="warn">

**CompositionEvent 组合输入事件**

> DOM 接口 **`CompositionEvent`** 表示用户间接输入文本（如使用输入法）时发生的事件。此接口的常用事件有 [compositionstart](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/compositionstart_event), [compositionupdate](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/compositionupdate_event) 和 [compositionend](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/compositionend_event)

- **compositionstart: 组合输入开始**。如：当拼音输入法开始输入时触发
 
- **compositionend: 组合输入结束**。如：当拼音输入法选择候选字后触发

其它情况：输入日文或韩文、手写识别、语音识别...

</text>

## 修复后的代码：

```jsx
/*react*/
<desc>
### 解决思路

1. 使用 onCompositionStart 和 onCompositionEnd 事件兼容 iOS 的中文输入法

2. 在触发 onChange 的时候，判断是否处于 composition，如果是，则不做数据处理，直接 setState

3. 在 onCompositionEnd 事件中处理字符，如：限制输入字符的最大长度
</desc>
<script>
  export default class LimitedInput extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
        value: '',
        isComposition: false, // 正在输入中文
      }
    }

    handleChange = (e) => {
      // 如果正在输入中文，则不过滤空格
      this.setState({ 
        value: this.state.isComposition
          ? e.target.value
          : e.target.value.replace(/\s/g, '').slice(0, 6),
      });
    }

    handleComposition = (e) => {
      if (e.type === 'compositionstart') {
        this.setState({
          isComposition: true,
        });
      }
      if (e.type === 'compositionend') {
        this.setState({
          isComposition: false,
          value: e.target.value.replace(/\s/g, '').slice(0, 6),
        });
      }
    }

    render() {
      return (
        <div>
          <input
            type="text"
            value={this.state.value}
            onChange={this.handleChange}
            onCompositionStart={this.handleComposition}
            onCompositionEnd={this.handleComposition}
          />
        </div>
      );
    }
  }
</script>
```
