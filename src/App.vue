<template>
  <div id="app">
    <div class="wbfc-linkage block">
    联动url:<el-input v-model="action" placeholder="http://192.168.20.5:8893/system/area/linkageList"></el-input><el-button type="primary" @click="reInit">点击重新初始化</el-button>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">直接使用v-model的绑定值</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" ref="linkage0">
      </wbfc-linkage>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">初始化不加载</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" :init="false" ref="linkage1">
      </wbfc-linkage>
      <el-button type="primary" @click="getData">点击加载数据</el-button>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">使用getValues函数</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action"v-model="selectedOptions2" ref="linkage2">
      </wbfc-linkage>
      <el-button type="primary" @click="getData2">获取数值数据</el-button>
      <el-button type="primary" @click="getData3">获取第二级的值</el-button>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">设置默认值</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" v-model="selectedOptions3" ref="linkage3">
      </wbfc-linkage>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">只联动2级</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" ref="linkage4" :link-level="2">
      </wbfc-linkage>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">选取任意节点</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" ref="linkage5" :cascader-props="{checkStrictly: true}">
      </wbfc-linkage>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">多选节点</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" ref="linkage6" :cascader-props="{multiple: true}">
      </wbfc-linkage>
    </div>
    <div class="wbfc-linkage block">
      <span class="demonstration">多选节点默认值</span>
      <wbfc-linkage v-if="isRouterAlive" :url="action" ref="linkage6" v-model="selectedOptions4" :cascader-props="{multiple: true}">
      </wbfc-linkage>
    </div>
  </div>
</template>

<script>
import WbfcLinkage from './WbfcLinkage'
/*WbfcLinkage.methods.setProps({multiple: true, checkStrictly: true})*/

export default {
  name: 'App',
  components: {
    WbfcLinkage
  },
  computed: {
  },
  data () {
    return {
      isRouterAlive: true,
      action: 'http://192.168.20.5:8893/system/area/linkageList',
      selectedOptions2:[],
      selectedOptions3: ['110000000000','110100000000','110101000000'],
      selectedOptions4: [
        ['110000000000','110100000000','110101000000'],
        ['110000000000','110100000000','110102000000'],
      ],
      linkProps: null,
    }
  },
  methods:{
    reInit(){
      this.isRouterAlive = false;
      this.$nextTick(() => (this.isRouterAlive = true))
    },
    getData() {
      this.$refs.linkage1.start();
    },
    getData2() {
      alert(this.$refs.linkage2.getValues());
    },
    getData3() {
      alert(this.$refs.linkage2.getValue(1));
    }
  },
  mounted(){
  }
}
</script>
