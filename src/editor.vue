<template>
  <div class="quill-editor">
    <slot name="toolbar"></slot>
    <div ref="editor"></div>
  </div>
</template>

<script>
  // require sources
  import { ImageExtend } from './lib/ImageExtend'
  import ImageResize from './lib/ImageResize'
  import defaultOptions from './options'

  Quill.register('modules/ImageExtend', ImageExtend)
  Quill.register('modules/ImageResize', ImageResize)

  // pollfill
  if (typeof Object.assign != 'function') {
    Object.defineProperty(Object, 'assign', {
      value(target, varArgs) {
        if (target == null) {
          throw new TypeError('Cannot convert undefined or null to object')
        }
        const to = Object(target)
        for (let index = 1; index < arguments.length; index++) {
          const nextSource = arguments[index]
          if (nextSource != null) {
            for (const nextKey in nextSource) {
              if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
                to[nextKey] = nextSource[nextKey]
              }
            }
          }
        }
        return to
      },
      writable: true,
      configurable: true
    })
  }

  // export
  export default {
    name: 'quill-editor',
    data() {
      return {
        _options: {},
        _content: '',
        defaultOptions
      }
    },
    props: {
      content: String,
      value: String,
      disabled: Boolean,
      options: {
        type: Object,
        required: false,
        default: () => ({})
      },
      globalOptions: {
        type: Object,
        required: false,
        default: () => ({})
      }
    },
    mounted() {
      this.initialize()
    },
    beforeDestroy() {
      this.quill = null
      delete this.quill
    },
    methods: {
      // Init Quill instance
      initialize() {
        if (this.$el) {

          // Options
          this._options = Object.assign({}, this.defaultOptions, this.globalOptions, this.options)

          // Instance
          this.quill = new Quill(this.$refs.editor, this._options)

          // Set editor content
          if (this.value || this.content) {
            this.quill.pasteHTML(this.value || this.content)
          }

          // Disabled editor
          if (this.disabled) {
            this.quill.enable(false)
          }

          // Mark model as touched if editor lost focus
          this.quill.on('selection-change', range => {
            if (!range) {
              this.$emit('blur', this.quill)
            } else {
              this.$emit('focus', this.quill)
            }
          })

          // Update model if text changes
          this.quill.on('text-change', (delta, oldDelta, source) => {
            let html = this.$refs.editor.children[0].innerHTML
            const quill = this.quill
            const text = this.quill.getText()
            if (html === '<p><br></p>') html = ''
            this._content = html
            this.$emit('input', this._content)
            this.$emit('change', { html, text, quill })
          })

          // 处理编辑器事件
          this.$refs.editor.onbeforecopy = function(e) {
            e = e || window.event;
            e.stopPropagation();
            return true;
          };
          // this.$refs.editor.onselect = this.$refs.editor.onmouseup = function(e) {
          //   e = e || window.event;
          //   e.stopPropagation();
          // };
          this.$refs.editor.onselect = function(e) {
            e = e || window.event;
            e.stopPropagation();
          };
          this.$refs.editor.onselectstart = function(e) {
            e = e || window.event;
            e.stopPropagation();
            e.returnValue = true;
          }
          this.$refs.editor.oncopy = function(e) {
            e = e || window.event;
            e.stopPropagation();
            e.returnValue = true;
          }
          // this.$refs.editor.onkeydown = function(e) {
          //   e = e || window.event;
          //   e.stopPropagation();
          //   e.returnValue = e.keyCode != 13;
          // }

          // Emit ready event
          this.$emit('ready', this.quill)
        }
      }
    },
    watch: {
      // Watch content change
      content(newVal, oldVal) {
        if (this.quill) {
          if (newVal && newVal !== this._content) {
            this._content = newVal
            this.quill.pasteHTML(newVal)
          } else if(!newVal) {
            this.quill.setText('')
          }
        }
      },
      // Watch content change
      value(newVal, oldVal) {
        if (this.quill) {
          if (newVal && newVal !== this._content) {
            this._content = newVal
            this.quill.pasteHTML(newVal)
          } else if(!newVal) {
            this.quill.setText('')
          }
        }
      },
      // Watch disabled change
      disabled(newVal, oldVal) {
        if (this.quill) {
          this.quill.enable(!newVal)
        }
      }
    }
  }
</script>
