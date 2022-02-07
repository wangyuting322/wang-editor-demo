<script>
// 方法
import E from 'wangeditor'

// 获取必要的变量，这些在下文中都会用到
const { $, DropListMenu, BtnMenu } = E
// 下拉菜单 - 选择xxx
class insertMenu extends DropListMenu {
  constructor (editor) {
    // 菜单栏中，标题菜单的 DOM 元素
    // 注意，这里的 $ 不是 jQuery ，是 E.$ （wangEditor 自带的 DOM 操作工具，类似于 jQuery）
    // data-title属性表示当鼠标悬停在该按钮上时提示该按钮的功能简述
    const $elem = $('<div class="w-e-menu" data-title="插入指标,列表或图表">下拉</div>')

    // droplist 配置
    const dropListConf = {
      width: 100,
      title: '插入',
      type: 'list',
      list: [
        { $elem: $('<div>指标</div>'), value: 'zb' },
        { $elem: $('<div>列表</div>'), value: 'list' },
        { $elem: $('<div>图表</div>'), value: 'chart' }
      ],
      // droplist 每个 item 的点击事件
      clickHandler: (value) => {
        this.editor.config.showDialog(value)
        // value 参数即 dropListConf.list 中配置的 value
        // this.command(value)
      }
    }

    super($elem, editor, dropListConf)
  }

  command (value, ss) {
    // 设置标题
    this.editor.cmd.do('insertMenu', value)
  }

  // 菜单是否需要激活
  tryChangeActive () {
    const reg = /^h/i
    const cmdValue = this.editor.cmd.queryCommandValue('insertMenu')
    if (reg.test(cmdValue)) {
      // 选区处于标题内，激活菜单
      this.active()
    } else {
      // 否则，取消激活
      this.unActive()
    }
  }
}
// 按钮菜单 - 插入图片
class imgMenu extends BtnMenu {
  constructor (editor) {
    // 菜单栏中，标题菜单的 DOM 元素
    // 注意，这里的 $ 不是 jQuery ，是 E.$ （wangEditor 自带的 DOM 操作工具，类似于 jQuery）
    // data-title属性表示当鼠标悬停在该按钮上时提示该按钮的功能简述
    const $elem = $('<div class="w-e-menu" data-title="插入本地图片">图片</div>')

    super($elem, editor)
    // droplist 配置

    // droplist 每个 item 的点击事件
  }

  command (value) {
    // 设置标题
    this.editor.cmd.do('imgMenu', value)
  }

  clickHandler (value) {
    this.editor.config.uploadImg(value)
    // value 参数即 dropListConf.list 中配置的 value
    // this.command(value)
  }

  // 菜单是否需要激活
  tryChangeActive () {
    const reg = /^h/i
    const cmdValue = this.editor.cmd.queryCommandValue('imgMenu')
    if (reg.test(cmdValue)) {
    // 选区处于标题内，激活菜单
      this.active()
    } else {
    // 否则，取消激活
      this.unActive()
    }
  }
}
const menuKey = 'insertMenu'
const menuKey2 = 'imgMenu'

export default {
  name: 'Home',
  components: {

  },
  data () {
    return {
      // 模板id
      tmplId: '',
      mode: 'create',
      value: null,
      // 原始模板内容 - 修改之前的模板内容
      originContent: '',
      // 当前模板内容 - 修改之后的模板内容
      tmplContent: '',
      editor: null,
      selectionTimer: null,
      // 指标的正则表达式
      zbReg: /#HC_INDEX#([\w#\u4e00-\u9fa5])*#HC_INDEX#|#HC_TABLE#([\w#\u4e00-\u9fa5])*#HC_TABLE#|#HC_IMG#([\w#\u4e00-\u9fa5])*#HC_IMG#/ig
    }
  },
  methods: {
    /**
     * 插入指标、里图标、图表的对话框显示
     */
    showDialog (menuValue) {
      console.log(menuValue)
    },
    /**
     * 插入本地图片
     */
    uploadImg () {
      this.$refs['upload-input'].click()
    },
    /**
    * input选择图片
    */
    getFile (e) {
      const file = e.target.files[0]
      // 校验文件类型
      if (!file.type.includes('image')) {
        // 清空input值
        this.clearInput()
        return false
      }
      // 创建文件读取对象
      const reader = new FileReader()
      reader.readAsDataURL(file)
      console.log(file)
      // 文件读取结束
      reader.onload = (e) => {
        console.log(e.target.result)
        console.log(reader.result)
        // 1.获取图片的base64
        const img64 = e.target.result
        // 2.上传file，获取图片在服务器上的位置
        const formdata = new FormData()
        formdata.append('file', file)
        // 3.插入图片至编辑器中
        return new Promise((resolve, reject) => {
          // 清空input值
          this.clearInput()
          this.insertHtml(`<img src="${img64}"/>`)
          resolve()
        })
      }
    },
    /**
     * 清空
     */
    clearInput () {
      if (this.$refs['upload-input']) {
        this.$refs['upload-input'].value = ''
      }
    },
    /**
     * 获取选中的文字的html
     */
    handleSelectText ({ text, html, selection }) {
      console.log(text, html)
      // 防抖
      if (this.selectionTimer) {
        clearTimeout(this.selectionTimer)
      }
      this.selectionTimer = setTimeout(() => {
        const matchzb = html.match(this.zbReg)
        // 匹配上 - 选中的文字中包含xxx
        if (matchzb) {
          console.log(`匹配上:${matchzb}`)
        }
        // 都没匹配上
        console.log('没有匹配上')
      }, 500)
    },
    /**
     * 获取模板内容
     */
    getTemplateContent () {
      let cacheContent = '<p>1131212</p>'
      // 对模板内容进行格式化 - 自定义内容格式化
      // 去除flex样式 - 防止列表的文字水平位置改变无效，且去除样式的重复
      cacheContent = cacheContent.replaceAll(/display:flex;|align-items:center;|justify-content:center;|justify-content:flex-end;/gi, '')
      // .replaceAll(/margin:auto;|padding:0;|margin-left:auto;|margin-right:0;|margin-left:auto;/g, '')

      this.tmplContent = cacheContent
      this.originContent = cacheContent
      // 重新设置编辑器内容
      this.editor.txt.html(this.tmplContent)
    },
    /**
     * 编辑器中的内容修改
     */
    handleEdit (newHtml) {
      if (this.mode !== 'query') {
        // 可编辑
        this.tmplContent = newHtml
      } else {
        // 查看模式下不允许修改
        if (newHtml !== this.originContent) {
          console.log('当前模式不允许修改模板内容')
          this.editor.txt.html(this.originContent)
        }
      }
    },
    /**
     * 插入指标
     */
    insertZb ({ form, data }) {
      const tableReg = /#HC_TABLE#([\w#\u4e00-\u9fa5])*#HC_TABLE#/ig
      const matchzb = data.match(tableReg)
      // 匹配上
      if (matchzb) {
        this.insertHtml(`<div>${data}</div>`)
      } else {
        this.insertHtml(`<span>${data}</span>`)
      }
    },
    /**
     * 插入html片段
     */
    insertHtml (htm) {
      this.originContent = `${this.originContent}${htm}`
      // 在光标处插入
      this.editor.cmd.do('insertHTML', htm)
    },
    /**
     * 显示图片配置 - 宽度配置
     */
    showImgConfig (e) {
      const value = prompt('请输入长度占比(单位：%)')
      if (Number(value) && value > 0 && value < 100) {
        console.log(value)
        e.selector.style.width = value + '%'
      } else {
        alert('请输入数值，并且大于0小于100')
      }
    },
    /**
     * 初始化编辑器
     */
    initEditor () {
      this.editor.config.zIndex = 1
      if (this.mode !== 'query') {
        // 注册菜单
        this.editor.menus.extend(menuKey, insertMenu)
        this.editor.menus.extend(menuKey2, imgMenu)
        // 也可以通过配置 menus 调整菜单的顺序，参考【配置菜单】部分的文档    editor.config.menus.push(menuKey)
        this.editor.config.menus = this.editor.config.menus.concat([menuKey, menuKey2]).filter(item => !['image', 'video'].includes(item))
      } else {
        this.editor.config.menus = []
      }
      // 配置全屏功能按钮是否展示
      this.editor.config.showFullScreen = false
      // 配置 onchange 回调函数
      this.editor.config.onchange = this.handleEdit
      // 配置触发 onchange 的时间频率，默认为 200ms
      this.editor.config.onchangeTimeout = 500 // 修改为 500ms
      // 配置 onSelectionChange 回调函数
      this.editor.config.onSelectionChange = this.handleSelectText
      // 点击插入菜单之后的动作
      this.editor.config.showDialog = this.showDialog
      // 点击插入图片之后的动作
      this.editor.config.uploadImg = this.uploadImg
      if (this.mode !== 'query') {
        // 图片的点击事件
        this.editor.txt.eventHooks.imgClickEvents.push(this.showImgConfig)
      }
      // 创建编辑器
      this.editor.create()
    },
    /**
     * 保存模板内容之前的操作
     */
    beforeSave () {
      return new Promise((resolve, reject) => {
        // 对比当前的模板内容以及最开始的模板内容
        const { tmplContent, originContent } = this
        const matchzb = tmplContent.match(this.zbReg) || []
        const matchOrigin = originContent.match(this.zbReg) || []
        // 取原模板中中不包含的现指标的部分，进行取消关联
        const needDel = matchOrigin.filter(item => !matchzb.includes(item))
        if (needDel.length) {
          setTimeout(() => {
            resolve()
          }, 300)
        } else {
          resolve()
        }
      })
    },
    /**
     * 保存
     */
    async handleSave () {
      if (this.mode !== 'query') {
        await this.beforeSave()
      }
      let { tmplContent } = this
      // 保存时，对模板的内容进行格式化
      // 添加flex样式 - 使表格居中
      // tmplContent = tmplContent.replaceAll(/text-align:center;/g, 'margin:auto;text-align:center;padding:0;')
      // tmplContent = tmplContent.replaceAll(/text-align:right;/g, 'margin-left:auto;margin-right:0;text-align:right;padding:0')
      tmplContent = tmplContent.replaceAll(/text-align:center;/g, 'display:flex;justify-content:center;align-items:center;text-align:center;')
      tmplContent = tmplContent.replaceAll(/text-align:right;/g, 'display:flex;justify-content:flex-end;align-items:center;text-align:right;')
      console.log('这是保存的模板内容：' + tmplContent)
    },
    /**
     * 取消
     */
    handleBack () {
      this.$router.go(-1)
    }
  },
  created () {
    const route = this.$route
    const { mode = 'create', id } = route.query
    this.tmplId = id
    this.mode = mode
  },
  mounted () {
    this.$nextTick(() => {
      this.editor = new E(document.getElementById('template-content-editor'))
      this.initEditor()
      if (this.mode !== 'create') {
        this.getTemplateContent()
      }
    })
  },
  beforeDestroy () {
    // 销毁编辑器
    this.editor.destroy()
    this.editor = null
  },
  watch: {

  },
  render (h) {
    return (
      <div class='template-content flex-col flex-grow'>
        <input type="file"
          onChange={this.getFile}
          ref={'upload-input'}
          accept="image/*"
          style='display:none;' />
        <div class='template-content-body'>
          <div id='template-content-editor'></div>
        </div>
        <div class='template-content-footer'>
          <div class='primary-btn' v-show={this.mode !== 'query'} onClick={this.handleSave}>保存</div>
          <div class='text-btn' v-show={this.mode !== 'query'} onClick={this.handleBack}>取消</div>
          <div class='primary-btn' v-show={this.mode === 'query'} onClick={this.handleBack}>返回</div>
        </div>
      </div>
    )
  }
}
</script>
<style lang="scss" scoped>
#template-content-editor {
  // height:300px;
}
.template-content {
  &-title {
    margin: 10px 0px;
  }
  &-body {
    flex-grow: 1;
    #template-content-editor {
      height: 100%;
      display: flex;
      flex-direction: column;
      & /deep/ .w-e-text-container {
        flex-grow: 1 !important;
        overflow-y: auto;
        display: flex;
        flex-direction: column;
      }
      &/deep/.w-e-text {
        // overflow-y: hidden;
        flex-grow: 1;
        min-height: auto !important;
        // padding:0px;
      }
    }
  }
  &-footer {
    margin: 10px 0px;
    display: flex;
    & > div:not(:first-child) {
      margin-left: 10px;
    }
  }
}
</style>
