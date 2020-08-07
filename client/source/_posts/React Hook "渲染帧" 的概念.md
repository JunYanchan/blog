---
title: React Hook "渲染帧" 的概念
date: 2020-07-08 14:50:40
tags:
---
## function component和class component之间会存在什么差异？
我们看下面两个例子：
```js
function Example1(props) {
    const { count } = props;
    const handleClick = () => {
      setTimeout(() => {
        alert(count);
      }, 3000);
    };
    return (
      <div>
          <p>{count}</p>
          <button onClick={handleClick}>Alert Count</button>
      </div>
    );
}
```
假如外部有一个父组件包着该子组件，并且有个state count向子组件传递，初始值为0，有个定时器使得count每秒增加1，这时点击“Alert Count”按钮，将会延迟3秒弹出count的值，会发现弹出的值并不是<font color="#9966CC">最新的值</font>，而是触发按钮那个时刻的count值，为什么？请继续往下看
```javascript
class Example2 extends Component {
  handleClick = () => {
    setTimeout(() => {
      alert(this.props.count);
    }, 3000);
  };

  render() {
    return (
      <div>
        <h2>Example2</h2>
        <p>{this.props.count}</p>
        <button onClick={this.handleClick}>Alert Count</button>
      </div>
    );
  }
}
```
我们和上面一样做相同的点击操作，点击“Alert Count”按钮，延迟3秒弹出count的值，会发现弹出的值就是页面呈现<font color="#9966CC">最新的值</font>
### 如何理解两者的差异呢？
在Example2组件中，我们是通过组件<font color="#9966CC">当前实例（this）</font>获取的count，***所以不管哪个时间点变化，this的指向还是当前***，所以延时器3秒后更新触发渲染，this.props也会跟着更新，所以count拿到的就是最新的值。

在Example1组件中，由于props是**函数作用域下**的变量，每次触发渲染调用函数时，都会产生一个**新的props变量**，意思就是变量声明时便确认了里面的值，他们之间相互不影响，所以你点击触发时的count就是在那个瞬间的count的变量值，尽管延时多少秒最后弹出的值，还是你触发按钮点击时的count值。

### 理解上面的例子后，我们再来看下React Hook中的例子
```js
function Example2() {
    const [count, setCount] = useState(0);
    const handleClick = () => {
      setTimeout(() => {
        setCount(count + 1);
      }, 3000);
    };

    return (
      <div>
        <p>{count}</p>
        <button onClick={() => setCount(count + 1)}>
          setCount
        </button>
        <button onClick={handleClick}>
          Delay setCount
        </button>
      </div>
    );
}
```
我们现在做一个点击操作：首先点击“Delay setCount”按钮，count依然为0,随后在3秒内连续点击“setCount“按钮两次，会发现count依次变为了1和2，但是3秒过后，count重新变为了1，如果你理解了上面的例子，那么对这个例子产生的结果就不会产生疑问

### 获取过去或者未来的值
说回上面的例子，如果我们希望setTimeout回调触发时，返回的count是当前最新的值，该如何去做呢？
#### 方法一:
利用<font color="#9966CC">传入函数作为参数</font>
```js
setTimeout(() => {
  setCount(prevCount => prevCount + 1); // 函数的参数prevCount即是前一个状态值
}, 3000);
```
#### 方法二：
利用<font color="#9966CC">useRef</font> hook去保存最新的count值
```js
function Example() {
  const [count, setCount] = useState(0);
  const currentCount = useRef(count);
  currentCount.current = count;
  const handleClick = () => {
      setTimeout(() => {
        setCount(currentCount.current + 1);
      }, 3000);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>
        setCount
      </button>
      <button onClick={handleClick}>
        Delay setCount
      </button>
    </div>
  );
}
```
每次组件渲染，把最新的count赋值给currentCount.current，useRef可以产生一个当前组件无论渲染多少遍都可共享的变量，<font color="#9966CC">就类似于class component的this追加属性</font>

> **每次组件函数执行渲染（render）可以理解为每一帧，这样可能比较好理解**

### 每一帧也有独立的Effects
```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
这个例子中假如我依次快速点击三次，延时后便会依次打印了1，2，3，都是count递增的效果，这是因为每次render相对独立并且获取了count递增更新后的值

> **React hook的生命周期和class component生命周期会有所差异，应当抛开在 class 组件中关于生命周期的思维**