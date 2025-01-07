---
title: React-11
date: 2025-01-05 15:24:20
categories:
  - React
tags: 
  - Redux-toolkit
---

## toolkit
由于传统的redux使用起来过于繁琐所以，有了toolkit来进行简化开发，现在使用redux基本上都会连带使用toolkit
```
import {configureStore} from "@reduxjs/toolkit"
import taskReducer from "./feature/taskSlice"

const store = configureStore({
  // 指定reducer
  reducer:{
    // 按模块管理各个切片
    task: taskReducer
  },
  // 使用中间件(如果不指定任何中间件，则默认集成了reduxThunk,但一旦设置，会整体替换默认值，需要手动指定
  thunk中间件)
  middleware: [.....]
})

export default store
```

### 设置模块切片
创建feature文件夹创建各个模块的切片

```
import {createSlice} from '@reduxjs/toolkit'

const taskSlice = createSlice({
  // 设置切片名字
  name: 'task',
  // 设置切片对应的reducer中的初始状态
  initialState: {
    taskList: null
    ....
  },
  reducers:{
    fn1(state,action){
      // state:redux中的公共状态信息(基于immer库管理，无需自己再克隆)
      // action:派发的行为对象，我们无需考虑行为表示的问题了，传递的其他信息，都是以action.payload传递进来的值
      ....
    }
  }
})

export default taskSlice.reducer
```

组件获取公共状态
```
let {taskList} = useSelector(state => state.task)
let dispatch = useDispatch()
```