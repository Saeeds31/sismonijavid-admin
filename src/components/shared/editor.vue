<template>
  <div class="editor-container">
    <div class="html-input-section">
      <textarea
        v-model="htmlInput"
        placeholder="کد HTML خود را اینجا وارد کنید..."
        class="html-textarea"
        dir="ltr"
      ></textarea>
      <button @click="insertHtml" type="button" class="insert-btn">درج HTML</button>
    </div>
    
    <QuillEditor
      ref="quillEditorRef"
      v-model:content="editorContent"
      content-type="html"
      theme="snow"
      :options="editorOptions"
      @ready="onEditorReady"
      @update:content="onEditorChange"
      dir="rtl"
      class="quill-editor"
    />
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import ImageUploader from 'quill-image-uploader'
import BlotFormatter from 'quill-blot-formatter'
import Quill from 'quill'

// ثبت پلاگین‌ها
Quill.register('modules/imageUploader', ImageUploader)
Quill.register('modules/blotFormatter', BlotFormatter)

// props و emit برای v-model
const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
})
const emit = defineEmits(['update:modelValue'])

// رفرنس به ادیتور
const quillEditorRef = ref(null)
let quillInstance = null

// مقدار داخلی برای QuillEditor
const editorContent = ref(props.modelValue)

// ورودی HTML
const htmlInput = ref('')

// سینک کردن تغییرات با v-model
watch(editorContent, (newValue) => {
  emit('update:modelValue', newValue)
})

// سینک کردن modelValue از والد به ادیتور
watch(() => props.modelValue, (newValue) => {
  if (newValue !== editorContent.value) {
    editorContent.value = newValue
  }
})

// تنظیمات ادیتور
const editorOptions = {
  modules: {
    toolbar: [
      [{ header: [1, 2, 3, 4, 5, 6, false] }],
      [{ font: [] }],
      ['bold', 'italic', 'underline', 'strike'],
      [{ align: ['right', 'center', 'left', 'justify'] }],
      [{ list: 'ordered' }, { list: 'bullet' }],
      [{ indent: '-1' }, { indent: '+1' }],
      [{ size: ['small', false, 'large', 'huge'] }],
      [{ color: [] }, { background: [] }],
      ['link', 'image', 'video', 'blockquote', 'code-block'],
      [{ direction: 'rtl' }],
      ['clean']
    ],
    imageUploader: {
      upload: (file) => {
        return new Promise((resolve, reject) => {
          const formData = new FormData()
          formData.append('file', file)
          fetch('/upload', {
            method: 'POST',
            body: formData
          })
            .then((response) => response.json())
            .then((result) => resolve(result.url))
            .catch(() => reject('آپلود تصویر ناموفق بود'))
        })
      }
    },
    blotFormatter: {},
    clipboard: {
      matchVisual: false
    }
  },
  placeholder: 'متن خود را اینجا بنویسید...'
}

// ایونت‌ها
const onEditorReady = (quill) => {
  quillInstance = quill
  console.log('ادیتور آماده شد:', quill)
  quill.container.style.direction = 'rtl'
  quill.container.style.textAlign = 'right'
}

const onEditorChange = () => {
  console.log('محتوا تغییر کرد:', editorContent.value)
}

// تابع درج HTML
const insertHtml = () => {
  if (!quillInstance || !htmlInput.value.trim()) return
  
  try {
    // روش 1: استفاده از clipboard
    const clipboard = quillInstance.getModule('clipboard')
    const range = quillInstance.getSelection(true)
    
    if (range) {
      clipboard.dangerouslyPasteHTML(range.index, htmlInput.value)
      quillInstance.setSelection(range.index + htmlInput.value.length, 0)
    } else {
      // اگر کرسر جایی نیست، به انتها اضافه کن
      const length = quillInstance.getLength()
      clipboard.dangerouslyPasteHTML(length - 1, htmlInput.value)
    }
    
    // پاک کردن textarea بعد از درج
    htmlInput.value = ''
    
    // به‌روزرسانی editorContent
    editorContent.value = quillInstance.root.innerHTML
  } catch (error) {
    console.error('خطا در درج HTML:', error)
    alert('خطا در درج HTML. لطفاً کد را بررسی کنید.')
  }
}

// متد عمومی برای درج HTML از خارج کامپوننت
const insertHtmlProgrammatically = (html) => {
  if (!quillInstance) return
  
  try {
    const clipboard = quillInstance.getModule('clipboard')
    const range = quillInstance.getSelection(true)
    
    if (range) {
      clipboard.dangerouslyPasteHTML(range.index, html)
    } else {
      const length = quillInstance.getLength()
      clipboard.dangerouslyPasteHTML(length - 1, html)
    }
    
    editorContent.value = quillInstance.root.innerHTML
  } catch (error) {
    console.error('خطا در درج HTML:', error)
  }
}

// متد برای جایگزینی کل محتوا
const setHtmlContent = (html) => {
  if (!quillInstance) return
  
  quillInstance.root.innerHTML = html
  editorContent.value = html
}

// expose کردن متدها برای استفاده از والد
defineExpose({
  insertHtml: insertHtmlProgrammatically,
  setHtmlContent
})
</script>

<style scoped>
.editor-container {
  max-width: 800px;
  margin: 0 auto;
}

.html-input-section {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f5f5f5;
  border-radius: 8px;
  border: 1px solid #ddd;
}

.html-textarea {
  width: 100%;
  min-height: 100px;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  resize: vertical;
  direction: ltr;
  text-align: left;
}

.insert-btn {
  padding: 8px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-family: 'Vazir', 'Arial', sans-serif;
  transition: background-color 0.3s;
}

.insert-btn:hover {
  background-color: #45a049;
}

.insert-btn:active {
  background-color: #3d8b40;
}

.quill-editor {
  direction: rtl;
  text-align: right;
  font-family: 'Vazir', 'Arial', sans-serif;
}

.quill-editor :deep(.ql-editor) {
  min-height: 300px;
  font-size: 16px;
  line-height: 1.6;
}

.quill-editor :deep(.ql-toolbar) {
  direction: ltr;
}
</style>