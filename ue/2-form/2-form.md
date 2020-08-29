# 2-form表单

> form 表单，基本上是pc后端系统中，交互展示最多的地方，也是最可能出问题的地方

## 关于验证

* 我强烈建议各位采用框架form里面自带的验证，几乎可以搞定所有的问题
* 那些非常规的表单项，比如form中新打开弹窗选择数据，也可以使用单独验证某条规则的方式来进行
  * el中为`validateField`，
* 利用表单自带的验证规则与阻断效果，可以最大限度少写代码

## 各种输入，建议都带上清空选项的配置
* el中为 `clearable`，antd中为`allowClear`，一样的效果
* 比如是下拉多选、级联多选的时候，如果选择了一堆想清空，有clear
* 比如某些非必选的select、时间选择，选择了可以清除掉

## 提交时跳转到报错项
* 在表单项比较多的时候，提交时跳转到第一个未验证通过的表单项，会大幅度的提高交互体验
* 在ant中，有自带的实现方法`scrollToFirstError`
* 在el中没有自带的方法，实现方式十分简单
``` javascript
// 自定义 scrollToFirstError 方法，利用scrollIntoView实现滚动
export const scrollToFirstError = () => {
  setTimeout(() => {
    try{
      let dom = document.querySelector('.el-form-item__error')
      dom && dom.scrollIntoView({block: 'center'})
    }catch(e){}
  }, 100)
}

// 表单提交时
submitForm(formName) {
  this.$refs[formName].validate((valid) => {
    if (valid) {
      alert('submit!');
    } else {
      scrollToFirstError() // 提交失败时调用
      console.log('error submit!!');
      return false;
    }
  });
},
```

## 关于表单项的tooltip提示
* 某些表单项，可能填写逻辑比较复杂，需要进行一些填写的提示
* input上倒是有placeholder，提示用户进行输入
  * 不过用户只要输入一个字符，就看不到占位符的提示了
* 建议放置通用样式、交互的tooltip提示，用于给用户提示
* 此处el与ant不同的地方是，antd的验证，直接与form.item绑定，直接在某个表单的input后面添加tooltip，比较麻烦
* el封装的`FormTooltip`组件，可以直接在input、select后引用
``` html
<!-- 定义组件 -->
<template>
  <el-tooltip
    v-show="content"
    effect="dark"
    :content="content"
    placement="top"
  >
    <i class="el-icon-info" style="margin-left: 10px; font-size:16px;"></i>
  </el-tooltip>
</template>

<script>
  export default {
    props: ['content']
  }
</script>

<!-- 方法调用 -->
<el-form-item label="xxx" prop="xxx">
  <el-input placeholder="请输入xxx" v-model="ruleForm.xxx"></el-input>
  <FormTooltip content="提交后不支持修改" />
</el-form-item>

```

* antd项目是需要封装form.item
``

## 上下互相影响的表单提交
* 有那些需要先选择了第一个表单项，再根据第一个表单项的值来展示第二个表单项的值的话，需要额外注意
* 如果第一个选择项内容变动了，需要将第二个表单项已选择的值清空，要不然会提交脏数据

## 多表单提交
* 如果一个界面，有N个表单，甚至是那种分tab页的表单，一次提交的话，也可以进行验证
* 推荐如下方式：
  * 将所有的表单遍历验证，如果有验证不通过的话，定位展示第一个验证不通过的表单


## 防止重复提交
* 理论上需要后端主力来做，但为了交互，前端也可以先行阻挡一把
* 在研究了统一接口请求的时候做拦截之后，觉得直接在每个form上操作，反而简单粗暴有效
* 或者直接在保存按钮上、或者直接在整个form上，加上v-loading，在接口提交的时候，展示loading图
* 这样既能展示信息提交的loading等待，又能阻止用户再次点击提交
