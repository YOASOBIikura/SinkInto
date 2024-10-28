---
title: Css-01
date: 2024-08-30 14:49:04
categories:
  - front
tags:
  - Css / Js
---

### css开发记录

`white-space: nowrap`
当没有设置span宽度时候，使用该属性可使字体直接铺平，推着后面的元素向后移动

`cursor: pointer`
当鼠标经过某一段区域的时候，可以适时的通过该属性更改鼠标样式增加交互性

-----

### JS开发记录

获取的列表属性与实际页面的展示数据不符时，直接在方法中进行遍历修改，然后在推到reactive中进行绑定展示

```
    let input= document.createElement("input")
    input.type="file"
    input.addEventListener('change', (event)=> {
      let files = event.target.files;
      console.log("Selected file:", files[0]);
      let formData = new FormData();
      formData.append('file', files[0]);
      uploadFile(formData).then(res=>{
          console.log(res,"ssss")
          if( res.code==0){
            data.chainInfo.icon="/images/"+res.data.file.key
          }
      })
    });
```
该方法先通过创建了一个input元素，然后绑定相关事件，触发这个事件的时候会读取事件中的文件到内存，然后在初始化一个
表单数据对象，并将该文件作为参数放入到表单数据对象中，然后在调上传文件的接口方法

```
for(let i=0;i<data.contractList.length;i++){
       let item=data.contractList[i]
        if(item.label=="" || item.value==""){
            ElMessage({
                message:"有无效输入",
                type:"warning",
            })
            return
        }
        contract[item.label]=item.value
    }
```
在对数据进行转换的时候就可以顺便对数据进行一些基本校验，比如非空和非法输入，避免了在调接口的时候再进行对数据的校验，这里直接
在接收数据转换的时候就及时进行了校验并返回更加的简洁和方便

设计巧思总结:
1. 添加和修改由于逻辑和界面都很相像这就可以使添加和修改的界面和相应的响应式对象都可以复用
2. 由于其流程基本相同，而不同只是调用不同的接口，这使得调用可以通过同一个方法进行操作，只用判别响应式对象是否有Id就可以区分是添加还是修改
3. 最核心的是他们操作的是同一个响应式对象，由于我之前想的是必须要分开操作添加和修改的不同的响应式对象，这样就不至于当用户打开修改对话框之
   后不管做不做任何修改都不会影响添加对话框中的响应式对象，添加的也不会跑的修改的对话框中去。而曾哥写的是直接当打开添加对话框的方法中直接
   重置响应式对象为空，这样就避免了修改数据串过来，而在打开修改对话框时就直接通过row对象直接转换进而就区分开了