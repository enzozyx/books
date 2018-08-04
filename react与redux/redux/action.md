# Action

[TOC]

## 1. 定义

> 把数据从应用中传递到store的有效载荷

一般通过 `store.dispatch()` 将action传到store

Action 使用一个javascript对象表述, 默认我们要求他必须要一个type属性来描述action意图



## 2. ActionType

> 一般的，我们会将Action中type 属性，统一放到一个文件中进行集中管理，而且它的值，基于可读性与可调式性考虑，推荐使用string 常量



## 3. Action创建函数

> 用来生成Action的函数，函数放回一个Action

- 传统的flux 会在创建Action函数时，自动调用dispatch
- redux 使用store.dispatch(action)方式, 来触发dispatch
- 或者是使用一个被绑定的Action创建函数

```javascript
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))

boundAddTodo(text);
boundCompleteTodo(index);
```

## connect



## bindActionCreators



