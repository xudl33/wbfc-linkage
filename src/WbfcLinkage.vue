<script>
  import Axios from 'axios';
  import ElCascader from 'element-ui/lib/cascader';
  import { generateId } from 'element-ui/lib/utils/util';
  delete ElCascader.props.options; // 删除options的属性 否则有必填的校验
  delete ElCascader.props.props; 
  export default {
    mixins: [ElCascader], // 合并Cascader
    name: 'WbfcLinkage',
    data() {
      return {
        id: generateId(), // id
        defaultLinkage: [], // 默认值联动临时变量
        options: [] // option列表
      };
    },
    props: {
      url: {
        type: String, // 联动URL地址
        default: '',
        required: true // 必填
      },
      props: {
        type: Object,
        default() {
          return {
            value: 'code', // 转换行政区划的格式 必须使用:props="props"
            label: 'name',
            children: 'children'
          };
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
      useCache: {
        type: Boolean, // 使用缓存项，若子列表在加载时本地已经有数据，就不再发送请求，若为false则每次都请求服务器
        default: true
      },
      beforeLinkage: Function, // 联动前拦截器
      showLog: {
        type: Boolean, // 打印debug日志
        default: false
      }
    },
    components: {
      ElCascader
    },
    watch: {
      value(val, oal) {
        // 如果新的值和旧的值不同 就触发联动(避免值手动设置value后不能联动的问题)
        if (val.join('') !== oal.join('')) {
          this.setDefaultValues();
        }
      }
    },
    methods: {
      handleChange(val) {
        var _this = this;
        var linkVal = _this.rootCode;
        var linkIndex = 0;
        if (val) {
          linkIndex = val.length;
          if (linkIndex <= 0) {
            linkVal = _this.rootCode;
          } else {
            linkVal = val[linkIndex - 1];
          }
        }
        var parentNode = _this.findTreeNode(linkVal, _this.options);
        if (parentNode) {
          var childList = parentNode[_this.props.children];
          if (_this.useCache && childList && childList.length > 0) {
            _this.writeLog('linkage ' + parentNode[_this.props.label] + ':' + linkVal + ' with Cache', childList);
            return;
          }
        }
        var linkageVal = { 'code': linkVal }; // 这个默认数据格式是联动行政区划使用的 如果需要联动其他数据 可以实现beforeLinkage方法并返回 参数值
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
          if (response && response.data && response.data.code === '000') {
            _this.writeLog('linkIndex = ' + linkIndex + ' ,linkLevel = ' + _this.linkLevel);
            // 如果达到最大联动层级 就删除子节点
            if (linkIndex + 1 >= _this.linkLevel) {
              response.data.result.forEach((n, i) => {
                n[_this.props.children] = null;
              });
            } else {
              // 一个空的子列表是为了能够动态加载下一级，否则就不显示箭头>了
              response.data.result.forEach((n, i) => {
                n[_this.props.children] = [];
              });
            }
            // 如果是一级 就直接赋值给options
            if (linkIndex <= 0) {
              _this.options = response.data.result;
            } else {
              if (parentNode) {
                parentNode[_this.props.children] = response.data.result;
              }
            }
            // 设置默认值
            _this.setDefaultValues();
          }
        });
      },
      setDefaultValues () { // 设置默认值
        var _this = this;
        var defaultValues = _this.value;
        // 如果大于联动级别，那么截取一下
        if (defaultValues.length > _this.linkLevel) {
          defaultValues = defaultValues.slice(0, _this.linkLevel);
        }
        if (defaultValues.length <= 0) {
          return;
        }
        var linkIndex = _this.defaultLinkage.length;        
        // 如果 联动时使用的是当前级别的编码 所以不用联动到最后 且 临时变量的长度比值小就进行联动
        if (linkIndex + 1 < _this.linkLevel && linkIndex < defaultValues.length) {
            _this.defaultLinkage = _this.defaultLinkage.concat(defaultValues[linkIndex]);
            _this.handleChange(_this.defaultLinkage);
        } else {
          // 联动结束后清空临时变量
          _this.defaultLinkage = [];
        }
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
      getValues() {
        return this.value;
      },
      getValue(i) {
        var index = i || 0;
        return this.value[index];
      },
      start() {
        // 加载数据
        this.handleChange();
        this.writeLog('is inited with configs', this.linkLevel);
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
      },
      listening() {
        // 监听激活item事件
        this.$on('active-item-change', function(val) {
          // 触发联动
          this.handleChange(val);
        });
      }

    },
    created() {
      // 初始化各个监听器
      this.listening();
      // 初始化联动 若需要初始化 则加载列表
      if (this.init) {
        this.start();
      }
    }
  };
</script>