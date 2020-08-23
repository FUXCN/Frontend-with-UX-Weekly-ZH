# 前言
在项目进行的过程中，我们不得不去理解别人写的代码。本文作者介绍了他所认为的代码的最佳实践，减少 bug。

使用 React 的困难之处在于
+ 通常没有明确的最佳实践，到处都是相互矛盾的文章
+ 很多写法正确与否取决于你是否真正理解 JS 本身
+ 由于行业和工作需求，很多人在深入了解 JavaScript 之前就开始写代码

作者决定写一篇关于他在实际项目中看到的案例和一些重构解决方案的小指南。

下面我们开始吧~

# 1. 可选 props 参数和空对象 `{}`
组件遵循 “单一目的” 哲学。避免过于复杂的组件，并尽可能多地拆分组件。

举个例子，假设我们有一个简单的 `<UserCard />` 组件，那么该组件的唯一目的就是接收用户对象，并显示用户数据。

```jsx
import React from "react";
import PropTypes from "prop-types";

const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
    </ul>
  );
};

UserCard.propTypes = {
  user: PropTypes.object
};

UserCard.defaultProps = {
  user: {} // notice here
};

export default UserCard;
```

这个组件需要一个 prop 参数来传入用户数据，来驱动显示整个组件。但是此 prop 并未设计为**必须**的，而是设置为默认值 `{}`，以免诸如 `Cannot access property 'name' of ...` 这样的错误。

这种情况下，我们试图访问 `user` 对象属性会一直返回 `undefined`，程序会试图用空数据去渲染组件，从而引起性能问题和 UI 混乱。

如果你的页面组件没有提供任何后备默认值或者骨架屏方案 （比如 LinkedIn 和 Facebook 搞得那种），就没有任何理由在没有数据的时候去渲染这些组件 —— 不然你组件内部的计算逻辑会使用空的值，进行错误的运算。

## 那么哪些东西是必须的，哪些不是
你需要停下来问一下自己，关键点是什么。

假设我们有一个名为 `<CurrencyConverter />` 的组件来转换汇率，它有三个属性：
+ `value`: 要转换的值
+ `givenCurrency`: 给定货币汇率
+ `targetCurrency`: 目标货币汇率

我们可以给定 **givenCurrency** 的默认值为 `"CNY"`，毕竟我们人在中国。**targetCurrency** 的默认值则按国际惯例设置为美元 `"USD"`。

但是如果缺少 `value` 值的话，任何转换都是没有意义的 —— 这个时候就没有必要进行组件的渲染了。因此 `value` 属性应当被设计为 **required** 的。

我们可以用同样的方法类比问自己哪些 props 是必要的，哪些 props 不是必要的。**组件的作用是 `do something`，不要让它在根本没有 `something` 的时候白渲染**。

# 2. 父组件中的条件渲染
你觉得 `<UserCard />` 和它的父组件 `<UserContainer />` 会如何配合?

```jsx
import React, { useState } from "react";
import PropTypes from "prop-types";

export const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
    </ul>
  );
};

UserCard.propTypes = {
  user: PropTypes.object.isRequired
};

export const UserContainer = () => {
  const [user, setUser] = useState(null);
  
  // do some apiCall here
  
  return (
    <div>
      {user && <UserCard user={user} />}
    </div>
  );
};
```

把 `user` 初始化为空值 —— 这样当用户不存在的时候只需要进行简单的判断 —— 比如 `!user` 就可以简单地进行判断了。如果传入一个空对象 `{}` 则没有那么简单了，对象之间比较需要检查对象的检查。此外，我们避免了创建新的对象，节省了内存空间，并且还对渲染进行了条件空值，仅当有数据时才会显示 `<UserCard />`

这种方式对于在没有数据的情况下显示提示信息和加载动画也非常方便。

```jsx
export const UserContainer = () => {
  const [user, setUser] = useState(null);
  
  // do some apiCall here
  
  return (
    <div>
      {user ? <UserCard user={user} /> : 'No data available'}
    </div>
  );
};
```

记住，**在父组件内部执行这些操作永远比在组件本身内部执行这些操作更干净。**我记不清有多少次我看到一个组件的某些逻辑被拜拜执行，最后只显示了一个加载图标和加载消息。

`<UserCard />` 仅负责显示用户数据。`<UserContainer />` 才是获取用户属性并且进行渲染决策的地方。所以 `<UserContainer />` 是显示 fallback 值的正确位置。

# 3. 避免嵌套，提前 return
即使在普通编程语言的范畴内，试图维护嵌套也是一团乱麻 —— 更不用说JSX了 (它是JavaScript、HTML和 `{}` 的混合)，。

你可能经常看到类似的代码：

```jsx
const NestedComponent = () => {
  // ...
  
  return (
    <>
      {!isLoading ? (
        <>
          <h2>Some heading</h2>
          <p>Some description</p>
        </>
      ) : <Spinner />}
    </>
  )
}
```

这样是不是好多了：

```jsx
const NestedComponent = () => {
  // ...
  
  if (isLoading) return <Spinner />
  
  return (
    <>
      <h2>Some heading</h2>
      <p>Some description</p>
    </>
  )
}
```

如果我们要调整渲染的条件（例如是否有可用数据，是否正在加载页面），可以选择提前返回。
不要将 HTML 与 JavaScript 条件混合使用，这样一来，我们就可以避免嵌套，并且对于别的成员或没有技术背景的人也更有可读性。

> 随着我越来越老越来越越有经验，我越关心代码对于任何人都可读。 我们不仅为计算机编代码，更为其他人编码。 我们希望审阅者能够理解我们的目标，而不是担心他们破译我们的代码。我们希望我们的下级同事能够理解代码，对其进行挑选和进一步开发。 如果从长远来看可以节省我和其他人的时间，我愿意牺牲多打几句话的功夫。

# 4. 在JSX中编写尽可能少的JavaScript
正如我前面提到的，JSX是一种复杂的语言，因为它混合了多种语言。虽然作为高级开发者，理解组件内部发生了什么没有问题，但并不是每个人这样。即使是我，作为一名高级工程师，也经常发现其他人的代码的可读性巨烂。

```jsx
const CustomInput1 = ({ onChange }) => {
  return (
    <Input onChange={e => {
      const newValue = getParsedValue(e.target.value);
      onChange(newValue);
    }} />
  )
}
```

这里使用自定义 handler 来处理和解析一些外部 `<Input />` 的变更，然后调用 `onChange` 方法，这个方法是 `<CustomInput />` 组件收到的属性。尽管此示例可以工作，但是太复杂了，很容易在 `return()` 里迷路。 在实践中，有更多的元素以及更复杂的JS逻辑嵌套在一起。

```jsx
const CustomInput2 = ({ onChange }) => {
  const handleChange = (e) => {
    const newValue = getParsedValue(e.target.value);
    onChange(newValue);
  };
  
  return (
    <Input onChange={handleChange} />
  )
}
```

这个示例只长了2行，但逻辑上显然是分离的。当返回 JSX 时，我们并不一定要使用内联 JavaScript ，使逻辑变得沉重。**在 `return()` 内部应该确保 HTML 树的结构很清楚，然而使用嵌套对象和在 JSX 中创建函数会让这个目标非常困难。**

# 5. useCallback 和 useEffect 一起使用
现在 hooks 和函数式组件已经非常常用了。很多时候，我们需要在组件内部调用数据 API，因此需要使用 `useEffect`。 按照第一版的文档，提供空的依赖项数组 `[]` 使挂钩仅在组件的初始化和卸载时运行。 因此，我们创建了组件，并在 `useEffect` 中使用了 `[]`。 但是，后来出现了 `exhaustive-deps` 规则。

```jsx
import React, { useState, useEffect } from 'react'

import { fetchUserAction } from '../api/actions.js'

const UserContainer = () => {
  const [user, setUser] = useState(null);
  
  const handleUserFetch = async () => {
    const result = await fetchUserAction();
    setUser(result);
  };
  
  useEffect(() => {
    handleUserFetch();
    // eslint-disable-next-line react-hooks/exhaustive-deps 
  }, []);
  
  if (!user) return <p>No data available.</p>
  
  return <UserCard data={user} />
};
```

人们纷纷跳出来说这个规则毫无意义。我们还不习惯。但现在，一年多过去了，有些人仍然在评论这个规则，而不是试图理解它是如何运作的，这不是一个合理的借口。

很多人没有意识到的是，**方法 `handleUserFetch()` 在每次渲染都会重新创建。** 这才是为什么我们需要在 `useEffect` 内调用方法时使用 `useCallback` 的原因。 这样，我们就防止了 `handleUserFetch()` 方法的重新创建（除非其依赖关系发生了变化），因此可以用作 `useEffect` 挂钩的依赖项，而不会导致无限循环。

上面的例子应当重写：

```jsx
import React, { useState, useEffect, useCalllback } from 'react'

import { fetchUserAction } from '../api/actions.js'

const UserContainer = () => {
  const [user, setUser] = useState(null);
  
  const handleUserFetch = useCalllback(async () => {
    const result = await fetchUserAction();
    setUser(result);
  }, []);
  
  useEffect(() => {
    handleUserFetch();
  }, [handleUserFetch]);
  
  if (!user) return <p>No data available.</p>
  
  return <UserCard data={user} />
};
```

我们把 `handleUserFetch` 作为 `useEffect` 的依赖项，并使用 `useCallback` 对其进行封装。如果 `handleUserFetch` 会接收一个外部参数，比如 `id` （实际情况中，比如为了要获取特定的 user，可能要这样做）, **这个参数将作为方法的依赖项在 `useCallback` 被列出**。只有在参数发生变化的时，函数才会再次执行。

如果你发现自己在循环调用 `useEffects` 和 `useCallback` ，考虑一下用 `useReducer` 来替代 `useState` ，或换一个完全不同的方法来实现。例如，如果你想设置一个状态，你可以使用前一个状态作为来函数的参数：

```jsx
setUser(prevUser => {…})
```

如果把当前 states 作为依赖，又在方法里处理状态更新的话，则将创建一个无限循环。

# 6. 将独立逻辑外置
组件有时会调用一些方法，这些方法与组件的变量一起协作并输出我们想要的结果。比如：
```jsx
const UserCard = ({ user }) => {
  const getUserRole = () => {
    const { roles } = user;
    if (roles.includes('admin')) return 'Admin';
    if (roles.includes('maintainer')) return 'Maintainer';
    return 'Developer';
  }
  
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
      <li>{getUserRole()}</li>
    </ul>
  );
}
```

尽管随着 hooks 普及，很多人放弃了类组件的写法，但是你发现自己经常会做出类似结构的组件。

上例中这样的方法会在每次渲染时重复创建，**而且这样的方法用 `useCallback` 则是杀鸡用牛刀，我们并没有在任何地方把这个东西作为*依赖*来挂载**（比如 useEffect, 或者子组件的参数）。

退一步来看，`getUserRole()` 只是一个接收用户角色参数，并返回用户最重要角色的函数。它并不需要在函数中定义（和被反复创建）。

组件内部定义的许多逻辑可以外置，因为它们不是**真正**与组件绑定的。

```jsx
const getUserRole = (roles) => {
  if (roles.includes('admin')) return 'Admin';
  if (roles.includes('maintainer')) return 'Maintainer';
  return 'Developer';
}

const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
      <li>{getUserRole(user.roles)}</li>
    </ul>
  );
}
```

通过这种方式，函数甚至可以在单独的文件中定义并在需要的地方导入。尽早将逻辑从组件中抽象出来，可以让我们拥有更干净的组件和更容易重用的实用工具函数库。

> 可能我们会想其它地方也展示用户最重要的角色，然后我们会在别在地方也写一套代码，然后写这些代码的人可能不一样，甚至实现可能都不一样。这样缺乏架构的代码在将来需要修改（比如你要改变用户的角色）时十分困难。这就是为什么我们应该尽早进行抽象，特别是当我们在一个团队中工作时，其他开发者会在我们的代码基础上进行构建，甚至可能在没有很好理解代码的情况下复制我们的代码。

# 7. 抛弃内联样式
我经常看见有人写内联样式，内联样式带来了两个问题：

+ 代码臃肿
+ 从外部只能用 `!important` 或一些 prop 下钻方法来改变 CSS

CSS 不是 JavaScript 相关语言之一。如果你是从某个 React 训练营起步的，你第一次看到别人用 CSS 是在一个使用 React 的 SPA 中，内联在 React 中可能是你对 CSS 的唯一认知。然而，CSS 是一种与 HTML 协同的语言，根本不需要 JS。使用 JS 可以动态地将一个 class 添加到 HTML 标签中，并将 CSS 应用到所述 class 中。CSS 提供了开箱即用的所有支持，而无需将其嵌入到 JavaScript 中。

```css
p { color: black }
p.accent { color: red }
```
上面的 CSS 表示每个段落的字体都是黑色。 具有`.accent` 类标记的段落则是红色字体。

```css
#banner p.accent { color: blue }
```

`#banner` 元素中的具有 `.accent` 标签的段落字体是蓝色的。了解 CSS 继承就可以知道， 如果我们内联定义 `p` 的样式，我们将很难从外部使用 CSS 选择器来获取并更改这个 `p` 标签的样式。 **我们只能创建只有文本颜色不同，其它都一样的组件。然后，当我们必须更改这些组件时，我们必须同时更改 n 个组件。**

## 臃肿的代码
在 https://jsfiddle.net/zonc3fvw/ 有一份使用内联样式的例子：
![Not too pretty, is it?]('./assets/1_-MOHX2YtVghAdKF8QYgM5g.png');

在 https://jsfiddle.net/vzcLxwny/1/ 则使用类进行了重写：
![]('./assets/1_CqgDoIR83KsCKhFw33y07Q.png')

我们看到的一个CSS类可以处理所有事情。 `<Card />` 组件在我们的应用程序中呈现的次数越多，我们节约的空间就越多。这时候我们只需要通过 HTML class 就可以更改 CSS 就可以改变样式，而不需要使用 prop 下钻, 也不需要 React.Context。

> 注意: 如果你喜欢用 CSS Modules，也是可以这样做的，只需使用 `:global` 前缀即可。比如你有一个 `icon`, 你想改变 `.icon` 在一个按钮中类的字体大小。同时想保留所有其他属性，创建一个新组件只是为了复制里面98%的代码是没有意义的。

一起有人反驳说“你可以用 props 来控制内联 CSS 啊”，你当然可以，但是，你的组件不应该用 10 个 prop 来处理 CSS。这里产生了 JS 和 CSS 的混合，在这里你可以尽情给自己挖坑，只是你会发现很难爬回去。组件将变得臃肿，难以理解，难以维护。你的 CSS 将越来越难以维护。

# 8. 写合乎规则的 HTML
我知道大多数人不喜欢 HTML 和 CSS，但是编写它们仍然是我们作为前端工程师的工作。React 很有趣，hooks 也很有趣，Context 也很棒，但最后我们要处理 HTML 渲染并使其看起来更漂亮。

如果它像一个按钮，那么它应该是一个 `button`，而不是一个可点击的 `div`。如果它不提交表单，那么它应该是 `type="button"`。如果它的文字数量不是固定的，它不应该在 CSS 里有固定的宽度。

如果间隔是 40 像素，那就不应该是 `<br /> <br /> <br /> <br />`。

如果是链接，则应为 `<a />`。

对于某些人来说，它是基础知识。 但是对其他人来说却不是。 如果你精通 React，但是是 HTML 小白，可以尝试创建一个完全不包含 JS 的网站 —— 仅包含纯 HTML。 通过 HTML 验证程序运行它。 这就是我们回到过去的方式（当时我10岁😄）。

表单是我经常看到的一个例子：

```jsx
import React, { useState } from 'react';

const Form = () => {
  const [name, setName] = useState('');
  
  const handleChange = e => {
    setName(e.target.value);
  }
  
  const handleSubmit = () => {
    // api call here
  }
  
  return (
    <div>
      <input type="text" onChange={handleChange} value={name} />
      <button type="button" onClick={handleSubmit}>
        Submit
      </button>
    </div>
  )
}
```
这个例子也能跑..能跑。这也是我经常在在客户端中看到的，每当我看到这种作品，我都在心里死了一小会儿。

现在，此示例所做的是，在 `<input />` 里的 **name** 更改，并单击 **Submit** `<button />` 时更新名称，并通过手动调用 `handleSubmit` 提交数据，**通过点击**。

这对于一般用户来说是可以的，但是对于使用 Enter 键提交表单的用户就不行了。HTML 无法识别这种写法，因为它实际上并不知道一个表单正在被提交。不要设置奇怪的输入监听器来实现原生 HTML & JS 支持的功能：

```jsx
<form onsubmit="myFunction()">
  Enter name: <input type="text">
  <input type="submit">
</form>
```

在 React 中使用 `onSubmit` 差不多也是这样：
```jsx

import React, { useState } from 'react';

const Form = () => {
  const [name, setName] = useState('');
  
  const handleChange = e => {
    setName(e.target.value);
  }
  
  const handleSubmit = e => {
    e.preventDefault();
    // api call here
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" onChange={handleChange} value={name} />
      <button type="submit">
        Submit
      </button>
    </form>
  )
}
```

与上一个示例相比，列出一点需要更改的内容：

+ 输入需要包装在 `<form />` 元素中
+ 按钮的类型需要是 `submit`, 它会查找最接近的父 `<form />` 并调用 submit 操作
+ 表单有一个 `onSubmit` 属性用于提交操作（`handleSubmit`）
+ submit 操作（`handleSubmit`）需要在处理事件时调用 `preventDefault()`。 否则页面将刷新 —— 就像传统的表单提交一样

现在表单可以提交了。它是一个合乎规则的表单，适用于任何提交触发器，而不仅仅是直接单击按钮。

# 9. 不要滥用 Context
在国外现在有两个阵营，Redux 阵营和 Context 阵营。大多数教程都是为 Redux 编写的，这是初学者会找到和学习的第一个资源 —— 对于他们的第一个项目来说，Redux 通常都算是滥用了。滥用 Redux 会伤到你的，很多人都被咬了，然后就转到第二阵营 —— Context 一切。

中庸是一切问题的答案，这个例子也不例外。

我曾见过很多应用程序以错误的方式使用 **Context**。应用被包装在 15 个不同的 Provider 中，分别存储状态和 setter。

假设你有两个组件用了`SnackbarContext`。其中一个负责显示 Snackbar 和里面的消息，另一个是用来触发来自外部的 Snackbar 消息。

**现在，每次带有消息的 Snackbar 渲染时，我们的第二个组件（就是那个只用于设置新的 Snackbar 消息的组件）也会重新渲染** —— 因为它的存储消息和访问器和第一个组件在同一个上下文。现在我们还没有接触到设置器，但是我们已经更新了消息，在 Provider 中设置了新值，并且与它关联的所有内容都重新呈现。 将组件包装在 `memo()` 也不起作用 —— Provider 的值不一样了。

假设你的应用中有 100 个或 1000 个这种东西。你性能就炸了。

有时候使用 Redux 是有意义的，特别是对于那些在整个应用程序中都需要，并且显示在不同屏幕上而不需要在所有地方重新获取的数据。**这里说的 Redux 是指其用做状态管理 —— 至于我本人，自从我发现了 mobx-state-tree 以后，我就再也回不去了。**

这是另一个复杂的主题，如果你对上下文管理感兴趣，可以在本文的底部找到一个神秘链接。

# 结语
如果你看到了这里，谢谢你的阅读。我希望我能帮助你们中的一些人弄清楚什么是什么。我只有一个建议，质疑你自己的知识、你学到的方法和你的代码。想想是否能做得更好。跳出 React 和 JavaScript 的框框来思考，将项目作为一个整体来看待，并扪心自问，从架构的角度来看，什么才是最合理的。


最后我想链接一些资源

+ https://slides.com/djanoskova/react-context-api-create-a-reusable-snackbar#/ — a presentation for Context API which I created to demonstrate how to use Context efficiently
+ https://medium.com/@ftangastani/mobx-state-tree-a-step-by-step-guide-for-react-apps-e65716a219d2 — a great article that shows mobx-state-tree basics in a very practical way
+ https://github.com/facebook/react/issues/14920 — exhaustive-deps discussion

# 译者
作者：Dana Janoskova
原文地址：https://itnext.io/write-clean-er-components-jsx-1e70491baded

本文为方便理解做了一定的意译。
本文很有趣地展示了东西方互联网方向初学者 roadmap 的不同 —— 毕竟你在中国很难找到一个会写 React 但是不会 HTML 的人 —— 不过大家都写不好 HTML 倒是很全球化。