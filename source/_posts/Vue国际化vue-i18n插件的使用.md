---
title: Vue.js国际化vue-i18n插件的使用
date: 2026-07-23 17:00:00
tags:
  - Vue
  - i18n
  - 国际化
categories:
  - 教程
keywords: "Vue, vue-i18n, 国际化, 多语言"
---

> 转载自：[博客园-大自然的流风](https://www.cnblogs.com/zdz8207/p/vue-i18n-js.html)

vue.js 国际化 vue-i18n 插件的使用问题，在模版文本、组件方法、jsf 方法里的使用

## 在模板文本中使用

```html
<span>{{$t("register.register")}}</span>
```

## 在组件方法中使用

```html
<md-input-item :placeholder="$t('register.enterCode')">
```

## 在JS方法中使用

```javascript
this.$i18n.t('register.imgCodeFirst')
```

## 在请求回调中使用

如果是在请求后返回的方法里使用，需要在上面先定义个变量：

```javascript
var this_ = this;
// 在回调里使用
Toast.info(this_.$i18n.t('register.getMsgCodeSucceed'));
```

## 在 main.js 中引入

```javascript
import i18n from './language/i18n'

new Vue({
  el: '#app',
  i18n, // 挂载语言包
  router: router,
  store: store,
  render: h => h(App)
})
```

## i18n.js 配置

```javascript
import Vue from 'vue'
import VueI18n from 'vue-i18n'
Vue.use(VueI18n)

// 语言包单独设置时需单独引用
const messages = {
  'zh_CN': require('./zh'),  // 中文语言包
  'en_US': require('./en'),  // 英文语言包
  'zh_TW': require('./tw')   // 繁体语言包
}

export default new VueI18n({
  locale: localStorage.getItem("language") || 'zh_CN',  // 默认显示
  messages,
  // silentTranslationWarn: true,
})
```

## 语言包示例

`zh.js`

```javascript
export default {
  register: {
    register: '注册',
    enterCode: '请输入验证码',
    imgCodeFirst: '请先输入图形验证码',
    getMsgCodeSucceed: '获取短信验证码成功'
  }
}
```

`en.js`

```javascript
export default {
  register: {
    register: 'Register',
    enterCode: 'Please enter verification code',
    imgCodeFirst: 'Please enter image verification code first',
    getMsgCodeSucceed: 'Get SMS code successfully'
  }
}
```

## 切换语言

```javascript
// 切换为英文
localStorage.setItem('language', 'en_US');
window.location.reload();
```

## 总结

vue-i18n 在 Vue 项目中的三种使用场景：

1. **模板文本**：`{{$t('key')}}`
2. **组件方法属性**：`$t('key')`
3. **JS 方法中**：`this.$i18n.t('key')`

注意在异步回调中使用时需要提前保存 `this` 引用。
