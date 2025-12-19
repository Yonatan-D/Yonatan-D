# 解决 iOS 中文输入混乱问题：空格过滤引发的血案

接到报障反馈 APP 修改昵称不能输入中文且输入框内的拼音叠加问题，Android 端正常，iOS 端异常。

排查问题：

 - 输入框限制文本长度为 6，当输入拼音超过 6 位时选词就无法填入
 
 - 输入框 change 事件过滤空格，iOS 以空格分隔拼音，空格过滤后没办法正确分词，造成中文输入混乱


有问题的代码：

```jsx
/*react*/
<desc>

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
        value: e.target.value.replace(/\s/g, ''),
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
            maxLength={6}
            value={this.state.value}
            onChange={this.handleChange}
          />
        </div>
      );
    }
  }
</script>
```

修复后的代码：

```jsx
/*react*/
<desc>
1. 使用 onCompositionStart 和 onCompositionEnd 事件
2. 在触发 onChange 的时候，判断是否处于 composition，如果是，则不做数据处理，直接 setState
3. 在 onCompositionEnd 事件中处理字符，如：限制输入字符的最大长度
</desc>
<script>
  export default class LimitedInput extends React.Component {
    constructor(props) {
      super(props)
      this.state = {
        value: '',
        isComposition: false,
      }
    }

    handleChange = (e) => {
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
