# ck-color-picker

> 基于Vue的颜色选择器插件

[演示地址](https://zcuizhichen.github.io/colorPickerPage/)

## 安装

``` bash
$ npm install ck-color-picker -S
```
## 使用

在 `main.js` 文件中引入插件并注册

``` bash
# main.js
import ckColorPicker from 'ck-color-picker'
Vue.use(ckColorPicker)
```

在项目中使用 ck-color-picker

```js
<template>
  <ck-color-picker v-model="color" v-bind:show.sync='show'/>
</template>
<script>
  export default {
    data () {
      return {
        color: '#ff0000',
        show: true
      }
    }
  }
</script>
```

## 特点
1. 可实现拖动选择颜色、透明度，颜色实时更新
2. 只需控制show即可控制展示与消失，不依赖与其他控件
3. 可实现随机颜色，选择困难症挚爱

### 选项
<table >
    <tr>
        <th>参数</th>
        <th>说明</th>
        <th>类型</th>
        <th>可选值</th>
        <th>默认值</th>
    </tr>
    <tr>
        <td><code>v-model/value</code></td>
        <td>颜色绑定值</td>
        <td>string</td>
        <td>——</td>
        <td>——</td>
    </tr>
    <tr>
        <td><code>color-format</code></td>
        <td>输出颜色的格式</td>
        <td>string</td>
        <td>hex/rgb</td>
        <td>rgb</td>
    </tr>
    <tr>
        <td><code>show.sync</code></td>
        <td>是否显示组件</td>
        <td>boolean</td>
        <td>true/false</td>
        <td>true</td>
    </tr>
</table>

## 事件
`change`颜色值改变的时候触发

``` js
<colorPicker v-model="color" @change="headleChangeColor" />
```