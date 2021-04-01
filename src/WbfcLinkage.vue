<script>
  import Axios from 'axios';
  import ElCascader from 'element-ui/lib/cascader';
  import { generateId } from 'element-ui/lib/utils/util';

  export default {
    mixins: [ElCascader], // 合并Cascader
    name: 'WbfcLinkage',
    data() {
      return {
        id: generateId(), // id
        initFlag: false,
        defaultLinkageExists:{}, // 默认值联动Map 用于校验去重
        defaultLinkage: [], // 默认值联动临时变量
        lazyLoadResolve: null, //动态加载回调
      };
    },
    props: {
      url: {
        type: String, // 联动URL地址
        default: '',
        required: true // 必填
      },
      cascaderProps: {
        type: Object,
        default: null,
        required: false
      },
      props: {
        type: Object,
        default() {
          var _this = this;
          return Object.assign({
            value: 'code', // 转换行政区划的格式 必须使用:props="props"
            label: 'name',
            children: 'children',
            lazy: true,
            lazyLoad (node, resolve) {
              /*resolve([{
                code: '110000000000',
                name: '北京市',
                children: [{
                  code: '110100000000',
                  name: '市辖区',
                  children: [{
                    code: '110101000000',
                    name: '东城去',
                  }]
                }]
              }])*/
              // 修正2.15.1版本升级后异步加载有进度条的问题 如果不调用resolve 就会一直转圈圈
              if(node.isLeaf){
                resolve();
                return;
              }
              // 初始化加载 直接联动的
              if(_this.init && !_this.initFlag){
                _this.start(node, resolve);
                return;
              } else {
                _this.lazyLoadResolve = resolve;
              }
              // 初始化不加载 点击才联动的 必须手动调用过start函数initFlag才会变为true
              if(_this.initFlag){
                _this.start(node, resolve);
              }
            },
          }, _this.$options.propsData.cascaderProps);
        }
      },
      linkLevel: {
        type: Number,
        default: 3 // 联动层级 默认为3
      },
      rootCode: {
        type: String,
        default: '0' // 根节点默认值，当为行政区划联动时 rootCode为0时可以获取省列表
      },
      init: {
        type: Boolean, // 初始化加载一级数据
        default: true
      },
      beforeLinkage: Function, // 联动前拦截器
      showLog: {
        type: Boolean, // 打印debug日志
        default: false
      }
    },
/*    components: {
      ElCascader
    },*/
    watch: {
      /*value(val, oal) {
        // 如果新的值和旧的值不同 就触发联动(避免值手动设置value后不能联动的问题)
        if (val.join('') !== oal.join('')) {
          //this.setDefaultValues();
        }
      }*/
      initFlag(val){
        // 初始化加载完成
        var maxLinkIndex = (this.linkLevel - 1);
        if(val == true){
          // 如果有默认值的话也需要联动
          if(this.value && this.value.length > 0){
            this.setDefaultLinkage(this.value);
            this.setDefaultValue();
          }
        }
      }
    },
    methods: {
      setDefaultLinkage(tempValue){
        var maxLinkIndex = (this.linkLevel - 1);
        for(var i in tempValue){
          var tempVal = tempValue[i];
          if(tempVal.constructor == Array){
            this.setDefaultLinkage(tempVal);
          } else {
            if(i < maxLinkIndex){
              if(!this.defaultLinkageExists[tempValue[i]]){
                this.defaultLinkageExists[tempValue[i]] = tempValue[i];
                this.defaultLinkage.push(tempValue[i]);
              }
            }
          }
        }
      },
      setDefaultValue(){
        if(this.defaultLinkage.length > 0){
          var linkNode = this.$refs.panel.getNodeByValue(this.defaultLinkage[0]);
          this.$refs.panel.lazyLoad(linkNode, () => {
            this.defaultLinkage = this.defaultLinkage.slice(1);
            this.setDefaultValue();
          });
        } else {
          // 清空联动值存在缓存
          this.defaultLinkageExists = {};
          //console.log(this.$refs.panel.store.getNodes())
          // 单选同步
          this.$refs.panel.syncActivePath();
          // 多选同步
          this.$refs.panel.syncMultiCheckState();
          // 回显文字
          this.computePresentContent();
        }
      },
      writeLog(info, obj) {
        if (this.showLog) {
          if (arguments.length > 2) {
            console.log('WbfcLinkage(id = ' + this.id + ') ' + info, Array.prototype.slice.call(arguments, 1));
            return;
          }
          if (info && obj) {
            console.log('WbfcLinkage(id = ' + this.id + ') ' + info, obj);
          } else {
            console.log('WbfcLinkage(id = ' + this.id + ') ' + info);
          }
        }
      },
      getValues() {
        return this.value;
      },
      getValue(i) {
        var index = i || 0;
        return this.value[index];
      },
      start(node, resolve){
        var _this = this;
        // 默认值
        if(!node){
          node = {level: 0,loading: true,root: true};
        }

        var linkVal = node.value || node.config?node.data[node.config.value]:null || _this.rootCode;
        // 这个默认数据格式是联动行政区划使用的 如果需要联动其他数据 
        var linkageVal = { 'code': linkVal }; 
        // 可以实现beforeLinkage方法并返回 参数值
        if (_this.beforeLinkage) {
          var tempRes = _this.beforeLinkage.call(_this, linkVal);
          if (tempRes && tempRes !== false) {
            linkageVal = tempRes;
          }
        }
        // 初始化联动 加载省级列表
        _this.linkage(_this.url, linkageVal, function(response) {
          var tep = {
            'url': _this.url,
            'data': linkageVal
          };
          _this.writeLog('axios with', tep);
          _this.writeLog('axios result', response.data);
          /*var linkChildArray = [];*/
          if (response && response.data && response.data.code === '000') {
            var linkResult = response.data.result;
            // leaf判断是否为叶子节点，leaf会不显示下级的箭头  -1是因为需要用前一级联动值判断当前联动的级别
            linkResult = linkResult.map(item => Object.assign(item, {leaf: node.level >= (_this.linkLevel - 1)}));
            /*linkChildArray.push(linkResult);*/
            if(!_this.initFlag){
              _this.initFlag = true;
            }
            if(resolve){
              resolve(linkResult);
            } else {
              _this.lazyLoadResolve(linkResult);
            }
            
          }
        });
      },
      findTreeNode(code, list) { // 查找树形结构的节点
        var _this = this;
        if (code) {
          var res = null;
          if (list) {
            for (var i in list) {
              var n = list[i];
              if (code === n[_this.props.value]) {
                res = n;
              } else {
                res = _this.findTreeNode(code, n[_this.props.children]);
              }
              if (res) {
                break;
              }
            }
          }
          return res;
        }
        return null;
      },
      linkage(url, param, callback) {
        var _this = this;
        // 行政区划数据
        Axios({
          url: url,
          contentType: 'application/json;charset=UTF-8',
          method: 'post',
          data: param
        }).then(function(response) {
          if (callback) {
            callback.call(_this, response);
          }
        }).catch(function(response) {
          _this.writeLog('axios exception', response);
        });
      }

    },
    mounted(){
      /*if(this.value && this.value.length > 0){
        for (var i in this.value) {
          var linkNode = this.panel.getNodeByValue(this.value[i]);
          this.panel.lazyLoad(linkNode);
        }
      }
      console.log();*/
    },
    created() {
      // 由于ElementUI2.9.1带有默认的动态加载API，现在新版的wbfclinage就没有那么臃肿啦
      //this.setDefaultValues();
    }
  };
</script>