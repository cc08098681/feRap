<template>
    <div class='editor-wrap'>
      <div class="field">
          <label><i class="red">*</i>输入数据格式</label>{{inputModel}}
          <textarea ref="inputeditor" ></textarea>
          <div class='' v-if='inputModel.length'>
            <div class='table-tr table-head'>
              <ul class='clearfix-sp'>]
                <li class='td-key' style='text-align:center'>属性</li>
                <li class='td-datatype'>数据类型</li>
                <li class='td-remark'>含义</li>
                <li class='td-mock'>是否必须</li>
              </ul>
            </div>
            <table-item :model='model' :is-child=false :loop=1 v-for='(model, key) in inputModel' v-bind:key="key" type="input"></table-item>
          </div>
      </div>
      <div class="field output-field clearfix-sp">
          <label><i class="red">*</i>返回数据格式<i :title="isHideOutputData ? '展开' : '隐藏'" @click="isHideOutputData = !isHideOutputData" class="chevron circle icon" :class="isHideOutputData ? 'up' : 'down'"></i>
          </label>
          <div class="output-editor" :class="{'hide' : isHideOutputData}">
            <div class='input-frame'>
              <form>
                <textarea ref="outputeditor" ></textarea>
              </form>
            </div>
            <div class='save-button mini ui button' @click='revertMock'>
              mock->
            </div>
            <div class='mock-frame'>
              <textarea ref="mockeditor" ></textarea>
            </div>
          </div>
        </div>

        <div class='' v-if='outputModel.length'>
          <div class='table-tr table-head table-item-output'>
            <ul class='clearfix-sp'>]
              <li class='td-key' style='text-align:center'>属性</li>
              <li class='td-datatype'>数据类型</li>
              <li class='td-remark'>含义</li>
              <li class='td-mock'><a href="http://mockjs.com" target="_blank">mock规则</a></li>
              <li class="td-select">状态选择</li>
            </ul>
          </div>
          <table-item :model='output' :is-child=false :loop=1 v-for="(output, key) in outputModel"  v-bind:key="key" type="output"></table-item>
        </div>
    </div>
</template>

<script type='text/babel'>
import tableItem from './table-item.vue'
import util from '@/common/util.js'
require('./codemirror_alias.js')
import mock from 'mockjs'
// import { listActive } from '../vuex/getters.js'

export default {
  // vuex: {
  //   getters: {
  //     list_active: listActive,
  //   }
  // },
  components: {
    tableItem
  },
  props: {
    list_active: {
      type: Object,
      default () {
        return {}
      }
    },
    outputModel: {
      type: Array,
      default () {
        return []
      },
      twoWay: true
    },
    showMock: {
      type: Boolean,
      default: false
    },
    isAdd: {
      type: Boolean,
      default: true
    },
    outputJson: {
      type: Object,
      default () {
        return {
          status: 1,
          data: {
            test: ''
          }
        }
      },
      twoWay: true
    },
    inputJson: {
      type: Object,
      default () {
        return {}
      },
      twoWay: true
    },
    editorError: {
      type: Object,
      default () {
        return {
          status: 1,
          msg: '正确'
        }
      }
    },
    inputModel: {
      type: Array,
      default () {
        return []
      },
      twoWay: true
    }

  },
  data () {
    return {
      // list_active: {},

      /*
      输入数据模型
       */
      inputEditor: {},
      inputData: '',
      /**
       * outputEditor 返回数据编辑框对象
       * outputJson 返回数据编辑框取到的值
       *
       */
      outputEditor: {},
      outputData: '',
      mockEditor: {},
      testModel: {
        a: '1',
        b: {
          b1: '2',
          b2: '3'
        }
      },
      testData: '',
      mockData: 'lalla',
      parants: '',
      editorReady: false,

      isHideOutputData: false // 是否隐藏返回数据格式
    }
  },
  watch: {
    inputJson (val) {
      console.log(`inputJson` + JSON.stringify(val))
    },
    inputModel (val) {
      console.log(`inputModel` + JSON.stringify(val))
    },
    outputData (val) {
      console.log(`outputData update....`)
      const self = this
      if (val && self.isJson(val)) {
        console.log('yes, is json')
        // self.outputJson = JSON.parse(val.replace(/[\s\r\n]/, ''))
        // self.outputModel = self.revertFormat(self.outputJson, [], 'output')
        self.$emit('update:outputJson', JSON.parse(val.replace(/[\s\r\n]/, '')))
        self.$emit('update:outputModel', self.revertFormat(self.outputJson, [], 'output'))

        if (!self.isJson(self.inputData)) {
          return self.setError(2)
        }
        return self.setError(1)
      }
      console.log('no, is not json')
      return self.setError(3)
    },
    inputData (val) {
      const self = this
      if (val && self.isJson(val)) {
        console.log('yes, is json')
        // self.inputJson = JSON.parse(val.replace(/[\s\r\n]/, ''))
        // self.inputModel = self.revertFormat(self.inputJson, [], 'input')
        const inputJsonVal = JSON.parse(val.replace(/[\s\r\n]/, ''))
        const inputModelVal = self.revertFormat(inputJsonVal, [], 'input')
        self.$emit('update:inputJson', inputJsonVal)
        self.$emit('update:inputModel', inputModelVal)
        if (!self.isJson(self.outputData)) {
          return self.setError(3)
        }
        return self.setError(1)
      }
      console.log('no, is not json')
      return self.setError(2)
    },
    isAdd (v) {
      console.log(`isAdd.....`)
      const self = this
      if (v) {
        // self.inputJson = {
        //   id: 123
        // }
        // self.outputJson = {
        //   status: 1,
        //   data: {
        //     test: 'test'
        //   }
        // }
        // self.outputModel = []

        self.$emit('update:inputJson', { id: 123 })
        self.$emit('update:outputJson', {
          status: 1,
          data: {
            test: 'test'
          }
        })
        self.$emit('update:outputModel', [])
      }
      self.$emit('init-code-mirror-all')
    }
  },
  mounted () {
    this.$on('init-code-mirror-all', this.initCodeMirrorAll)
    this.$on('remove-code-mirror-all', this.removeCodeMirrorAll)
    this.$on('revertMock', this.revertMock)

    this.initCodeMirrorAll()
  },
  // 清除事件监听
  beforeDestroy: function () {
    this.$off('init-code-mirror-all', this.initCodeMirrorAll)
    this.$off('remove-code-mirror-all', this.removeCodeMirrorAll)
    this.$off('revertMock', this.revertMock)
  },
  methods: {
    initEditor (dom, readOnly) {
      console.log(`initEditor...dom=${dom}`)
      const editor = CodeMirror.fromTextArea(dom, {
        lineNumbers: true,
        mode: 'application/json',
        lint: true,
        gutters: ['CodeMirror-lint-markers'],
        readOnly: readOnly ? 'nocursor' : false,
        tabSize: 2,
        styleActiveLine: true
      })

      return editor
    },
    getEditorData (editor) {
      const self = this
      return self[`${editor}Editor`].getValue()
    },
    setEditorData (editor, value) {
      const self = this
      if (self.isJson(value)) {
        value = JSON.parse(value)
      }
      const jsonStr = JSON.stringify(value, null, 2)
      self[`${editor}Editor`].setValue(jsonStr)
    },
    /**
     * 校验输入数据是否为json
     * @param str
     * @returns {boolean}
       */
    isJson (str) {
      try {
        JSON.parse(str)
        return true
      } catch (e) {
        return false
      }
    },
    /**
     * 校验Object
     * @param val
     * @returns {Boolean}
       */
    isObject (val) {
      return _.isObject(val)
    },
    /**
     * 将outputModel的每个字段转化成api接口字段output的数组元素
     * @param outputModel
     * @returns {{key: *, comment: string, dataType: string, mock: string, children: Array}}
       */
    revertFormat (model, parents, type) {
      console.log(`revertFormat......`)
      const self = this
      const output = []
      let children = ''
      let comment = ''
      let dataType = ''
      let mockModel = ''
      let child = ''
      let require = true
      let isSelect = false
      const staticModel = type === 'input' ? self.inputModel : self.outputModel
      _.each(model, (value, key) => {
        const _dataType = util.getDataType(value)
        switch (_dataType) {
          case 'Object':
            child = self.comparison(key, parents, staticModel, 0)
            parents.push(key)
            children = self.revertFormat(value, parents, type)
            comment = child.comment
            dataType = 'Object'
            mockModel = child.mock || self.defaultMock(key, _dataType)
            require = child.require
            isSelect = child.isSelect
            parents.pop()
            break
          case 'Array':
            child = self.comparison(key, parents, staticModel, 0)
            parents.push(key)
            if (typeof value[0] === 'object') {
              children = self.revertFormat(value[0], parents, type)
            } else {
              children = null
            }
            comment = child.comment
            dataType = 'Array'
            mockModel = child.mock || self.defaultMock(key, _dataType)
            require = child.require
            isSelect = child.isSelect
            parents.pop()
            break
          default:
//            if (outputModel.length > 0) {
            child = self.comparison(key, parents, staticModel, 0)
            comment = child.comment || ''
            children = child.children || null
            mockModel = child.mock || self.defaultMock(key, _dataType)
            dataType = _dataType
            require = child.require
            isSelect = child.isSelect

//            } else {
//              comment = value
//              children = null
//              dataType = _dataType
//              mockModel = self.defaultMock(key, _dataType)
//            }

            break
        }
        const tmp = {
          key: key,
          comment: comment,
          dataType: dataType,
          mock: mockModel,
          children: children,
          require: require,
          isSelect: isSelect
        }
        if (type === 'input') {
          delete tmp.mock
          delete tmp.isSelect
        } else {
          delete tmp.require
        }
        output.push(tmp)
      })
      return output
    },
    /**
     * 校验value是否为对象或数组(是的话则为称呼为folder)
     * @param obj
       */
    isFolder (obj) {
      let returnData = ''
      if (_.isArray(obj) || _.isObject(obj)) {
        returnData = true
      } else {
        returnData = false
      }
      return returnData
    },
    revertMock () {
      const self = this
      try {
        const mockModel = util.mockTree2MockTemplate(self.outputModel)
        const mockData = mock.mock(mockModel)
        self.setEditorData('mock', mockData)
      } catch (e) {
        toastr.error('请输入正确mock规则')
      }
    },
    comparison (key, parents, output, loop) {
      const self = this
      let child = ''
      const parentKey = parents[loop]
      if (parentKey) {
        output = self.findChild(parentKey, output).children
        loop++
        child = self.comparison(key, parents, output, loop)
      } else {
        child = self.findChild(key, output)
      }
      return child
    },
    findChild (key, output) {
      let v = ''
      _.each(output, (item) => {
        if (item.key === key) {
          v = item
        }
      })
      return v
    },
    setError (status) {
      const self = this
      const editorError = self.editorError
      editorError.status = status
      switch (status) {
        case 1:
          editorError.msg = ''
          break
        case 2:
          editorError.msg = '输入数据格式必须为json'
          break
        case 3:
          editorError.msg = '返回数据格式必须为json'
          break
        default:
          editorError.msg = ''
      }
    },
    defaultMock (key, type) {
      switch (type) {
        case 'Number':
          if (key === 'status') {
            return '|1:'
          }

          if (/mobile|phone/i.test(key)) {
            return '|10000000000-19999999999:'
          }
          return '|1-100:'
        case 'String':
          // 带name的key
          if (/name/i.test(key)) {
            return ':@cname(1,6)'
          }
          // 带date的key
          if (/date/i.test(key)) {
            return ':@date("yyyy-MM-dd")'
          }
          // 带time的key
          if (/time/i.test(key)) {
            return ':@time()'
          }
          return ':@word(1,10)'
        case 'Boolean':
          return '|1:true'
        case 'Array':
          return '|1-10:'
        case 'Object':
          return '|:'
        default:
          return ':1'
      }
    },

    initCodeMirrorAll () {
      console.log(`initCodeMirrorAll....`)
      const self = this
      if (self.editorReady) {
        self.setEditorData('input', self.inputJson)
        self.setEditorData('output', self.outputJson)
        self.$emit('revertMock')
        return
      }
      console.log(`self.$refs=${JSON.stringify(self.$refs)}`)

      const inputEditorDom = self.$refs.inputeditor
      const outputEditorDom = self.$refs.outputeditor
      const mockEditorDom = self.$refs.mockeditor
      console.log(`inputEditorDom=${inputEditorDom}`)
      console.log(`outputEditorDom=${outputEditorDom}`)

      self.inputEditor = self.initEditor(inputEditorDom)
      self.inputEditor.on('change', (cm, obj) => {
        self.inputData = $.trim(self.getEditorData('input'))
      })

      self.outputEditor = self.initEditor(outputEditorDom)
      self.outputEditor.on('change', (cm, obj) => {
        self.outputData = $.trim(self.getEditorData('output'))
      })

      self.mockEditor = self.initEditor(mockEditorDom, true)
      self.editorReady = true
      if (!self.list_active.id) {
        // self.inputJson = {
        //   id: 123
        // }
        // self.outputJson = {
        //   status: 1,
        //   data: {
        //     test: 'test'
        //   }
        // }
        // self.outputModel = []
        self.$emit('update:inputJson', { id: 123 })
        self.$emit('update:outputJson', {
          status: 1,
          data: {
            test: 'test'
          }
        })
        self.$emit('update:outputModel', [])
      }
      self.setEditorData('input', self.inputJson)
      self.setEditorData('output', self.outputJson)
      self.$emit('revertMock')
    },
    removeCodeMirrorAll () {
      self.editorReady = false
    }
  }
}
</script>
<style media='screen' lang='less'>
  .editor-wrap{
    margin-bottom: 30px;
    font-size: 14px;
    li{
      list-style:none;
    }
    .output-editor{
      position:relative;
      border-top:1px solid rgb(221,221,221);
      margin-bottom:30px;
    }
    .input-frame{
      display:inline-block;
      width: 50%;
      min-height:300px;
      border-right: 1px solid rgb(221,221,221);
    }
    .CodeMirror-scroll{
      max-height:800px;
    }
    .mock-frame{
        /* display:inline-block; */
        float:right;
        width:50%;
        min-height:300px;
        max-height:800px;
        /* border:1px solid rgb(221,221,221) */
    }
    .save-button{
      display:block;
      position:absolute;
      width:10%;
      /* height:40px; */
      line-height:40px;
      /* background-color:#ddf0ed; */
      color:#fff;
      text-align:center;
      left:50%;
      margin-left:-5%;
      top:100px;
      z-index:99;
      border-radius:5%;
      cursor:pointer;
    }
    .table-head{
      font-weight: bold;
    }
    .icon.chevron {
      font-size: 16px;
      color: #3380b6;
      cursor: pointer;
    }
    .hide {
      display: none;
    }
  }
  .CodeMirror-gutters{
      border: 0 !important;
      background-color: #fff !important;
  }
  .CodeMirror{
    height:auto !important;
    /* max-height:300px !important; */
    overflow:scroll;
  }
</style>
