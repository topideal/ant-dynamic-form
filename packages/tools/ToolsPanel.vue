<!-- 工具箱面板 -->
<template>
  <div class="tools-panel">
    <a-button
      v-show="mode === 'edit'"
      icon="undo"
      type="link"
      size="small"
      :disabled="undoStack.size() === 0"
      @click="handleUndo">撤回</a-button>
    <a-button
      v-show="mode === 'edit'"
      icon="redo"
      type="link"
      size="small"
      :disabled="redoStack.size() === 0"
      @click="handleRedo">重做</a-button>
    <a-divider v-show="mode === 'edit'" type="vertical" />
    <a-button icon="bulb" type="link" size="small">表单提交</a-button>
    <a-button icon="search" type="link" size="small">配置查看</a-button>
    <a-divider type="vertical" />
    <a-button
      v-show="mode === 'edit'"
      icon="delete"
      type="link"
      size="small"
      @click="handleClear">清空</a-button>
    <a-button
      v-show="mode === 'edit'"
      icon="check"
      type="link"
      size="small">完成</a-button>
    <a-divider v-show="mode === 'edit'" type="vertical" />
    <a-button
      v-show="mode === 'preview'"
      icon="eye"
      type="link"
      size="small"
      @click="handleModeChange('edit')"
    >预览模式</a-button>
    <a-button
      v-show="mode === 'edit'"
      icon="edit"
      type="link"
      size="small"
      @click="handleModeChange('preview')"
    >编辑模式</a-button>
  </div>
</template>

<script>
import { cloneDeep } from '~/util'
import PanelInjectMixin from '~/common/panel-inject-mixin'
export default {
  name: 'ToolsPanel',
  mixins: [PanelInjectMixin],
  data () {
    return {
      undoStack: new MaxSizeStack(1000),
      redoStack: new MaxSizeStack(1000),
      historySettingClone: null,
      stackPushLock: false
    }
  },
  watch: {
    formSetting: {
      handler (val) {
        if (this.stackPushLock) {
          this.stackPushLock = false
        } else {
          this.undoStack.push(this.historySettingClone)
        }
        this.historySettingClone = cloneDeep(val)
      },
      deep: true
    }
  },
  methods: {
    /**
     * 处理撤回
     */
    handleUndo () {
      this.stackPushLock = true
      this.redoStack.push(this.formSetting)
      this.$emit('on-setting-change', this.undoStack.pop())
    },
    /**
     * 处理重做
     */
    handleRedo () {
      this.stackPushLock = true
      this.undoStack.push(this.formSetting)
      this.$emit('on-setting-change', this.redoStack.pop())
    },
    handleClear () {
      this.$emit('on-clear')
    },
    handleModeChange (mode) {
      this.$emit('on-mode-change', mode)
    }
  },
  created () {
    this.historySettingClone = cloneDeep(this.formSetting)
  }
}
/**
 * 限制容量的栈
 */
function MaxSizeStack (size) {
  this.maxSize = size
  this.items = []
  this.pop = function () {
    return this.items.pop()
  }
  this.push = function (item) {
    if (this.size() >= this.maxSize) {
      this.items.splice(0, 1)
    }
    this.items.push(item)
  }
  this.size = function () {
    return this.items.length
  }
  this.peek = function () {
    if (this.size() === 0) {
      return null
    }
    return this.items[this.size() - 1]
  }
}
</script>

<style lang="less" scoped>
.tools-panel {
  line-height: 45px;
  font-size: 18px;
  text-align: right;
}
</style>
