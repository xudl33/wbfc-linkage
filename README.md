# wbfc-linkage

> 智源云联动器

它是一个基于`Wbfc`智源云服务和`element-ui`的联动器组件。它使用`Axios`进行网络请求，因此可以异步联动任意层级的结构化数据。

当一个有着数形或层级关系的数据集合需要异步从服务器加载时，就可以选择智源云联动器。它是`Cascader`(级联选择器)的子集，除了一些特有的属性以外，均可以参考 [Cascader(级联选择器)](http://element-cn.eleme.io/#/zh-CN/component/cascader)
### 基础用法

一般联动器在有选择行政区划的业务中最为常见

```html
<div class="wbfc-linkage block">
  <span class="demonstration">默认 初始化加载数据</span>
  <wbfc-linkage :url="action">
  </wbfc-linkage>
</div>
<div class="wbfc-linkage block">
  <span class="demonstration">默认 手动加载数据</span>
  <wbfc-linkage ref="linkage" :url="action" :init="false">
  </wbfc-linkage>
  <el-button type="primary" @click="initData">加载数据</el-button>
</div>
<script>
export default {
    data() {
      return {
        action : 'http://192.168.20.5:1106/system/area/linkageList'
      }
    },
    methods: {
      initData() {
        this.$refs.linkage.start();
      }
    }
  };
</script>
```

### 设置默认值

若联动器有需要初始化的默认值，只需要绑定`v-model`即可，联动器的value是Array型。

```html
<div class="block">
  <span class="demonstration">带有默认值的初始化</span>
  <wbfc-linkage :url="action" v-model="selectedOptions">
  </wbfc-linkage>
</div>
<script>
export default {
    data() {
      return {
        action : 'http://192.168.20.5:1106/system/area/linkageList',
        selectedOptions:['110000000000','110100000000','110101000000']
    }
  };
</script>
```

### 获取联动值

一般在包含有保存或选择联动器值的情况下，会需要得到联动器的选中值，一般有三种方法可以得到选中值。

1. 直接使用`v-model`的绑定值
2. 使用`getValues`函数
3. 使用`getValue`函数 (按索引得到某个选定值)

```html
<div class="wbfc-linkage block">
  <span class="demonstration">直接使用v-model的绑定值</span>
  <wbfc-linkage :url="action" v-model="selectedOptions2">
  </wbfc-linkage>
  <el-button type="primary" @click="getData">加载数据</el-button>
</div>
<div class="wbfc-linkage block">
  <span class="demonstration">使用getValues函数</span>
  <wbfc-linkage :url="action" ref="linkage2">
  </wbfc-linkage>
  <el-button type="primary" @click="getData">加载数据</el-button>
</div>
<div class="wbfc-linkage block">
  <span class="demonstration">使用getValue函数</span>
  <wbfc-linkage :url="action" v-model="selectedOptions3" ref="linkage3">
  </wbfc-linkage>
  <el-button type="primary" @click="getData3">获取第二级的值</el-button>
</div>
<script>
export default {
    data() {
      return {
        action : 'http://192.168.20.5:1106/system/area/linkageList',
        selectedOptions3: ['110000000000','110100000000','110101000000']
    },
    methods: {
      getData() {
        alert(this.selectedOptions2);
      },
      getData2() {
        alert(this.$refs.linkage2.getValues());
      },
      getData3() {
        alert(this.$refs.linkage3.getValue(1));
      }
    }
  };
</script>
```

### 只联动到某层级

在某些特定的场景下，联动的层级很可能不同, 默认为三级联动。

```html
<div class="block">
  <span class="demonstration">二级联动</span>
  <wbfc-linkage :url="action" :link-level="2">
  </wbfc-linkage>
</div>
<script>
export default {
    data() {
      return {
        action : 'http://192.168.20.5:1106/system/area/linkageList'
      }
    }
  };
</script>
```

### 自定义联动规则

由于业务的不同，可能会遇到数据结构虽然满足层次结构的要求，但是数据项名称不满足，此时可以手动指定联动规则以及参数值。

```html
<div class="block">
  <span class="demonstration">自定义联动规则</span>
  <wbfc-linkage :url="action" :props="props" :before-linkage="beforeLinkage">
  </wbfc-linkage>
</div>
<script>
export default {
    data() {
      return {
        action : 'http://192.168.20.5:1106/system/area/linkageList',
        props: {
            value: 'code', // 为返回值提供联动格式转换
            label: 'name',
            children: 'children'
        },
        beforeLinkage: function(val) {
          // 将联动值转换为联动url的参数
          if (!val) {
            val = "0";
          }
          return { 'code' : val };
        }
      }
    }
  };
</script>
```

### Attributes

除以下的属性外，其他属性请参考 [Cascader(级联选择器)](http://element-cn.eleme.io/#/zh-CN/component/cascader)

| 参数      | 说明    | 类型      | 可选值       | 默认值   |
|---------- |-------- |---------- |-------------  |-------- |
| url | 联动请求URL 必填 | string | — | — |
| link-level | 联动层级 | number | — | 3 |
| root-code | 根节点值(获取第一级数据的请求值)   | string | — | 0 |
| init | 是否初始化加载一级数据 | boolean | — | true |
| use-cache | 是否使用缓存项，若子列表在加载时本地已经有数据，就不再发送请求，若为false则每次都请求服务器   | boolean | —  | true |
| show-log | 是否打印debug日志 | boolean | — | false |
| before-linkage | 联动前拦截器 在自定义联动规则时，在联动请求前进行联动值的转换 | function(value) | — | — |


## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run all tests
npm test
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

## Versions
版本|更新时间|更新说明
---|---|---
1.0.9-r2 | 2019/06/28 | 修正wbfcLinkage版本没有退回的问题
1.0.9-r1 | 2019/06/25 | 修正element依赖版本(2.9之后的API重构了，如果直接install会被更新成最新版)
1.0.9 | 2018/11/01 | 修正在change-on-select="true"联动失效的问题;
1.0.8 | 2018/10/26 | 修正手动设置默认值联动不正确的问题;
1.0.7以前|2018/10/25| wbfc-linkage发布到npm server;