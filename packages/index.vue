<template>
  <div class="form-designer">
    <el-container>
      <!-- 左侧字段 -->
      <el-aside :width="leftWidth">
        <div class="fields-list">
          <template v-if="customFields && customFields.length > 0">
            <el-link class="field-title"
                     :underline="false"
                     href="https://github.com/sscfaith/avue-form-design/blob/master/CHANGELOG.md#2020-09-22"
                     target="_blank">自定义字段 <i class="el-icon-question"></i></el-link>
            <draggable tag="ul"
                       :list="customFields"
                       :group="{ name: 'form', pull: 'clone', put: false }"
                       ghost-class="ghost"
                       :sort="false">
              <template v-for="(item, index) in customFields">
                <el-tooltip v-if="item.tips"
                            effect="dark"
                            :content="item.tips"
                            :key="index">
                  <li class="field-label"
                      :key="index">
                    <a style="padding: 0 5px;"
                       @click="handleFieldClick(item)">
                      <i :class="item.icon"></i>
                      <span style="margin-left: 5px;">{{item.title || item.label}}</span>
                    </a>
                  </li>
                </el-tooltip>
                <li v-else
                    class="field-label"
                    :key="index">
                  <a style="padding: 0 5px;"
                     @click="handleFieldClick(item)">
                    <i :class="item.icon"></i>
                    <span style="margin-left: 5px;">{{item.title || item.label}}</span>
                  </a>
                </li>
              </template>
            </draggable>
          </template>
          <div v-for="(field, index) in fields"
               :key="index">
            <template v-if="field.list.find(f => includeFields.includes(f.type))">
              <div class="field-title">{{field.title}}</div>
              <draggable tag="ul"
                         :list="field.list"
                         :group="{ name: 'form', pull: 'clone', put: false }"
                         ghost-class="ghost"
                         :sort="false">
                <template v-for="(item, index) in field.list">
                  <li class="field-label"
                      v-if="includeFields.includes(item.type)"
                      :key="index">
                    <a @click="handleFieldClick(item)">
                      <i class="icon iconfont"
                         :class="item.icon"></i>
                      <span>{{item.title || item.label}}</span>
                    </a>
                  </li>
                </template>
              </draggable>
            </template>
          </div>
        </div>
      </el-aside>
      <!-- 中间主布局 -->
      <el-container class="widget-container"
                    direction="vertical">
        <el-header class="widget-container-header">
          <div>
            <template v-if="undoRedo">
              <el-button type="text"
                         size="medium"
                         icon="el-icon-refresh-left"
                         :disabled="historySteps.index == 0"
                         @click="widgetForm = handleUndo()">撤销</el-button>
              <el-button type="text"
                         size="medium"
                         icon="el-icon-refresh-right"
                         :disabled="historySteps.index == historySteps.steps.length - 1"
                         @click="widgetForm = handleRedo()">重做</el-button>
            </template>
          </div>
          <div style="display: flex; align-items: center;">
            <iframe src="https://ghbtns.com/github-btn.html?user=sscfaith&repo=avue-form-design&type=star&count=true"
                    frameborder="0"
                    scrolling="0"
                    width="100"
                    height="20"
                    title="GitHub"
                    style="margin-left: 10px;"
                    v-if="showGithubStar"></iframe>
            <slot name="toolbar-left"></slot>
            <el-button v-if="toolbar.includes('avue-doc')"
                       type="text"
                       size="medium"
                       icon="el-icon-document"
                       @click="handleAvueDoc">Avue文档</el-button>
            <el-button v-if="toolbar.includes('import')"
                       type="text"
                       size="medium"
                       icon="el-icon-upload2"
                       @click="importJsonVisible = true">导入JSON</el-button>
            <el-button v-if="toolbar.includes('generate')"
                       type="text"
                       size="medium"
                       icon="el-icon-download"
                       @click="handleGenerateJson">生成JSON</el-button>
            <el-button v-if="toolbar.includes('preview')"
                       type="text"
                       size="medium"
                       icon="el-icon-view"
                       @click="handlePreview">预览</el-button>
            <el-button v-if="toolbar.includes('clear')"
                       class="danger"
                       type="text"
                       size="medium"
                       icon="el-icon-delete"
                       @click="handleClear">清空</el-button>
            <slot name="toolbar"></slot>
          </div>
        </el-header>
        <el-main :style="{background: widgetForm.column.length == 0 ? `url(${widgetEmpty}) no-repeat 50%`: ''}">
          <widget-form ref="widgetForm"
                       :data="widgetForm"
                       :select.sync="widgetFormSelect"
                       @change="handleHistoryChange(widgetForm)"></widget-form>
        </el-main>
      </el-container>
      <!-- 右侧配置 -->
      <el-aside class="widget-config-container"
                :width="rightWidth">
        <el-tabs v-model="configTab"
                 stretch>
          <el-tab-pane label="字段属性"
                       name="widget"
                       style="padding: 0 10px;">
            <widget-config :data="widgetFormSelect"></widget-config>
          </el-tab-pane>
          <el-tab-pane label="表单属性"
                       name="form"
                       lazy
                       style="padding: 0 10px;">
            <form-config :data="widgetForm"></form-config>
          </el-tab-pane>
        </el-tabs>
      </el-aside>
      <!-- 弹窗 -->
      <!-- 导入JSON -->
      <el-drawer title="导入JSON"
                 :visible.sync="importJsonVisible"
                 size="50%"
                 append-to-body
                 destroy-on-close>
        <monaco-editor v-model="importJson"
                       keyIndex="import"
                       height="82%"></monaco-editor>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handleImportJsonSubmit">确定</el-button>
          <el-button size="medium"
                     type="danger"
                     @click="importJsonVisible = false">取消</el-button>
        </div>
      </el-drawer>
      <!-- 生成JSON -->
      <el-drawer title="生成JSON"
                 :visible.sync="generateJsonVisible"
                 size="50%"
                 append-to-body
                 destroy-on-close>
        <monaco-editor v-model="widgetFormPreview"
                       keyIndex="generate"
                       height="82%"
                       :read-only="true"></monaco-editor>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handleGenerate">生成</el-button>
          <el-button size="medium"
                     type="primary"
                     slot="reference"
                     @click="handleCopy"
                     style="margin-left: 10px;">复制</el-button>
        </div>
      </el-drawer>
      <!-- 预览 -->
      <el-drawer title="预览"
                 :visible.sync="previewVisible"
                 size="60%"
                 append-to-body
                 :before-close="handleBeforeClose">
        <avue-form v-if="previewVisible"
                   ref="form"
                   class="preview-form"
                   :option="widgetFormPreview"
                   v-model="widgetModels"
                   @submit="handlePreviewSubmit"></avue-form>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handlePreviewSubmit">确定</el-button>
          <el-button size="medium"
                     type="danger"
                     @click="handleBeforeClose">取消</el-button>
        </div>
      </el-drawer>
    </el-container>
  </div>
</template>

<script>
import fields from './fieldsConfig.js'
import beautifier from './utils/json-beautifier'
import MonacoEditor from './utils/monaco-editor'
import widgetEmpty from './assets/widget-empty.png'
import history from './mixins/history'

import Draggable from 'vuedraggable'

import WidgetForm from './WidgetForm'
import FormConfig from './FormConfig'
import WidgetConfig from './WidgetConfig'

export default {
  name: "FormDesign",
  components: { Draggable, MonacoEditor, WidgetForm, FormConfig, WidgetConfig },
  mixins: [history],
  props: {
    options: {
      type: [Object, String],
      default: () => {
        return {
          column: []
        }
      }
    },
    storage: {
      type: Boolean,
      default: false
    },
    asideLeftWidth: {
      type: [String, Number],
      default: '270px'
    },
    asideRightWidth: {
      type: [String, Number],
      default: '380px'
    },
    showGithubStar: {
      type: Boolean,
      default: true
    },
    toolbar: {
      type: Array,
      default: () => {
        return ['import', 'generate', 'preview', 'clear']
      }
    },
    undoRedo: {
      type: Boolean,
      default: true
    },
    includeFields: {
      type: Array,
      default: () => {
        const arr = []
        fields.forEach(f => {
          f.list.forEach(c => {
            arr.push(c.type)
          })
        })
        return arr
      }
    },
    customFields: {
      type: Array,
    }
  },
  watch: {
    options: {
      handler(val) {
        let options = val
        if (typeof options == 'string') {
          try {
            options = eval('(' + options + ')')
          } catch (e) {
            console.error('非法配置')
            options = { column: [] }
          }
        }
        this.transAvueOptionsToFormDesigner(options).then(res => {
          this.widgetForm = { ...this.widgetForm, ...res }
        })
      },
      deep: true
    }
  },
  computed: {
    leftWidth() {
      if (typeof this.asideLeftWidth == 'string') {
        return this.asideLeftWidth
      } else {
        return `${this.asideLeftWidth}px`
      }
    },
    rightWidth() {
      if (typeof this.asideRightWidth == 'string') {
        return this.asideRightWidth
      } else {
        return `${this.asideRightWidth}px`
      }
    }
  },
  data() {
    return {
      widgetEmpty,
      fields,
      widgetForm: {
        column: [],
        labelPosition: 'left',
        labelSuffix: '：',
        labelWidth: 120,
        gutter: 0,
        menuBtn: true,
        submitBtn: true,
        submitText: '提交',
        emptyBtn: true,
        emptyText: '清空',
        menuPosition: 'center'
      },
      widgetFormPreview: {},
      configTab: 'widget',
      widgetFormSelect: {},
      previewVisible: false,
      generateJsonVisible: false,
      importJsonVisible: false,
      importJson: {},
      widgetModels: {},
      configOption: {},
      history: {
        index: 0, // 当前下标
        maxStep: 20, // 最大记录步数
        steps: [], // 历史步数
      }
    }
  },
  mounted() {
    this.handleLoadStorage()
    this.handleLoadCss()
  },
  methods: {
    // 组件初始化时加载本地存储中的options(需开启storage),若不存在则读取用户配置的options
    async handleLoadStorage() {
      let options = this.options
      if (typeof options == 'string') {
        try {
          options = eval('(' + options + ')')
        } catch (e) {
          console.error('非法配置')
          options = { column: [] }
        }
      }
      if (!options.column) options.column = []
      this.widgetForm = this.initHistory({
        index: 0,
        maxStep: 20,
        steps: [await this.transAvueOptionsToFormDesigner({ ...this.widgetForm, ...options })],
        storage: this.storage
      })

      if (this.undoRedo) {
        window.addEventListener('keydown', (evt) => {
          // 监听 cmd + z / ctrl + z 撤销
          if ((evt.metaKey && !evt.shiftKey && evt.keyCode == 90) || (evt.ctrlKey && !evt.shiftKey && evt.keyCode == 90)) {
            this.widgetForm = this.handleUndo()
          }

          // 监听 cmd + shift + z / ctrl + shift + z / ctrl + y 重做
          if ((evt.metaKey && evt.shiftKey && evt.keyCode == 90) || (evt.ctrlKey && evt.shiftKey && evt.keyCode == 90) || (evt.ctrlKey && evt.keyCode == 89)) {
            this.widgetForm = this.handleRedo()
          }
        }, false)
      }

    },
    // 加载icon
    handleLoadCss() {
      const head = document.getElementsByTagName('head')[0]
      const script = document.createElement('link')
      script.rel = 'stylesheet'
      script.type = 'text/css'
      script.href = 'https://at.alicdn.com/t/font_1254447_zc9iezc230c.css'
      head.appendChild(script)
      // this.loadScript('css', 'https://at.alicdn.com/t/font_1254447_zc9iezc230c.css')
    },
    // Avue文档链接
    handleAvueDoc() {
      window.open('https://avuejs.com/doc/form/form-doc', '_blank')
    },
    // 左侧字段点击
    handleFieldClick(item) {
      const activeIndex = this.widgetForm.column.findIndex(c => c.prop == this.widgetFormSelect.prop) + 1
      let newIndex = 0
      if (activeIndex == -1) {
        this.widgetForm.column.push(item)
        newIndex = this.widgetForm.column.length - 1
      } else {
        this.widgetForm.column.splice(activeIndex, 0, item)
        newIndex = activeIndex
      }

      this.$refs.widgetForm.handleWidgetAdd({ newIndex })
    },
    // 预览 - 弹窗
    handlePreview() {
      if (!this.widgetForm.column || this.widgetForm.column.length == 0) this.$message.error("没有需要展示的内容")
      else {
        this.transformToAvueOptions(this.widgetForm).then(data => {
          this.widgetFormPreview = data
          this.previewVisible = true
        })
      }
    },
    // 导入JSON - 弹窗 - 确定
    handleImportJsonSubmit() {
      try {
        this.transAvueOptionsToFormDesigner(this.importJson).then(res => {
          this.widgetForm = res
          this.importJsonVisible = false
          this.handleHistoryChange(this.widgetForm)
        })
      } catch (e) {
        this.$message.error(e.message)
      }
    },
    // 生成JSON - 弹窗
    handleGenerateJson() {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        this.widgetFormPreview = data
        this.generateJsonVisible = true
      })
    },
    // 生成JSON - 弹窗 - 确定
    handleGenerate() {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        this.$emit('submit', data)
      })
    },
    // 生成JSON - 弹窗 - 拷贝
    handleCopy() {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        this.$Clipboard({
          text: beautifier(data)
        }).then(() => {
          this.$message.success('复制成功')
        }).catch(() => {
          this.$message.error('复制失败')
        })
      })
    },
    // 预览 - 弹窗 - 确定
    handlePreviewSubmit(form, done) {
      if (done) {
        this.$alert(this.widgetModels).then(() => {
          done()
        }).catch(() => {
          done()
        })
      } else {
        this.$refs.form.validate((valid, done) => {
          if (valid) this.$alert(this.widgetModels).then(() => {
            done()
          }).catch(() => {
            done()
          })
        })
      }
    },
    // 预览 - 弹窗 - 关闭前
    handleBeforeClose() {
      this.$refs.form.resetForm()
      this.widgetModels = {}
      this.previewVisible = false
    },
    // 清空
    handleClear() {
      if (this.widgetForm && this.widgetForm.column && this.widgetForm.column.length > 0) {
        this.$confirm('确定要清空吗？', '警告', {
          type: 'warning'
        }).then(() => {
          this.$set(this.widgetForm, 'column', [])
          this.$set(this, 'widgetModels', {})
          this.$set(this, 'widgetFormSelect', {})
          this.handleHistoryChange(this.widgetForm)
        }).catch(() => {
        })
      } else this.$message.error("没有需要清空的内容")
    },
    // 表单设计器配置项 转化为 Avue配置项
    transformToAvueOptions(obj) {
      return new Promise((resolve, reject) => {
        try {
          const data = this.deepClone(obj)
          for (let i = 0; i < data.column.length; i++) {
            const col = data.column[i]
            if (col.type == 'dynamic' && col.children && col.children.column && col.children.column.length > 0) {
              const c = col.children.column;
              c.forEach(item => {
                delete item.subfield
              })
              this.transformToAvueOptions(col.children).then(res => {
                col.children = res
              })
            } else if (col.type == 'group') {
              if (!data.group) data.group = []

              const group = {
                label: col.label,
                icon: col.icon,
                prop: col.prop,
                arrow: col.arrow,
                collapse: col.collapse,
                display: col.display
              }
              this.transformToAvueOptions(col.children).then(res => {
                group.column = res.column
                data.group.push(group)
              })
              data.column.splice(i, 1)
              i--
            } else if (['checkbox', 'radio', 'tree', 'cascader', 'select'].includes(col.type)) {
              if (col.dicOption == 'static') {
                delete col.dicUrl
                delete col.dicMethod
                delete col.dicQuery
                delete col.dicQueryConfig
              } else if (col.dicOption == 'remote') {
                delete col.dicData
                if (col.dicQueryConfig && col.dicQueryConfig.length > 0) {
                  const query = {}
                  col.dicQueryConfig.forEach(q => {
                    if (q.key && q.value) query[q.key] = q.value
                  })
                  col.dicQuery = query
                  delete col.dicQueryConfig
                } else delete col.dicQueryConfig
              }
              delete col.dicOption
            } else if (['upload'].includes(col.type)) {
              if (col.headersConfig && col.headersConfig.length > 0) {
                const headers = {}
                col.headersConfig.forEach(h => {
                  if (h.key && h.value) headers[h.key] = h.value
                })
                col.headers = headers
              } else delete col.headers
              delete col.headersConfig

              if (col.dataConfig && col.dataConfig.length > 0) {
                const data = {}
                col.dataConfig.forEach(h => {
                  if (h.key && h.value) data[h.key] = h.value
                })
                col.data = data
              } else delete col.data
              delete col.dataConfig
            }
            if (col.change) col.change = eval(col.change)
            else delete col.change

            if (col.click) col.click = eval(col.click)
            else delete col.click

            if (col.focus) col.focus = eval(col.focus)
            else delete col.focus

            if (col.blur) col.blur = eval(col.blur)
            else delete col.blur
          }
          resolve(data)
        } catch (e) {
          reject(e)
        }
      })
    },
    // Avue配置项 转化为 表单设计器配置项
    transAvueOptionsToFormDesigner(obj) {
      if (typeof obj == 'string') obj = eval('(' + obj + ')')
      const data = this.deepClone(obj)
      return new Promise((resolve, reject) => {
        try {
          if (data.column && data.column.length > 0) {
            data.column.forEach(col => {
              if (col.type == 'dynamic' && col.children && col.children.column && col.children.column.length > 0) {
                const c = col.children.column;
                c.forEach(item => {
                  item.subfield = true
                })
                this.transAvueOptionsToFormDesigner(col.children).then(res => {
                  col.children = res
                })
              } else if (['checkbox', 'radio', 'tree', 'cascader', 'select'].includes(col.type)) {
                if (!col.dicData && col.dicQuery && typeof col.dicQuery == 'object') {
                  const arr = []
                  for (let key in col.dicQuery) {
                    arr.push({
                      key,
                      value: col.dicQuery[key],
                      $cellEdit: true
                    })
                  }
                  col.dicQueryConfig = arr
                }
                if (col.dicUrl) col.dicOption = 'remote'
                else col.dicOption = 'static'
                if (!col.dicData) col.dicData = []
              } else if (['upload'].includes(col.type)) {
                if (col.headers && typeof col.headers == 'object') {
                  const arr = []
                  for (let key in col.headers) {
                    arr.push({
                      key,
                      value: col.headers[key],
                      $cellEdit: true
                    })
                  }
                  col.headersConfig = arr
                } else col.headersConfig = []

                if (col.data && typeof col.data == 'object') {
                  const arr = []
                  for (let key in col.data) {
                    arr.push({
                      key,
                      value: col.data[key],
                      $cellEdit: true
                    })
                  }
                  col.dataConfig = arr
                } else col.dataConfig = []
              }
            })
          }
          if (data.group && data.group.length > 0) {
            for (let i = 0; i < data.group.length; i++) {
              if (!data.column) data.column = []
              const col = data.group[i]

              const group = {
                type: 'group',
                label: col.label,
                icon: col.icon,
                prop: col.prop,
                arrow: col.arrow,
                collapse: col.collapse,
                display: col.display
              }
              this.transAvueOptionsToFormDesigner(col).then(res => {
                group.children = res
                data.column.push(group)
              })
            }
            delete data.group
          }
          resolve(data)
        } catch (e) {
          reject(e)
        }
      })
    },
    async getData() {
      return await this.transformToAvueOptions(this.widgetForm)
    }
  }
}
</script>

<style lang="scss">
@import './styles/index.scss';
</style>
