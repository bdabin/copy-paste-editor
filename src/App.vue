<template>
  <div id="app">
    <Editor ref="tuiEditor" initialEditType="wysiwyg" :options="editorOptions" />
    <button @click="test">저장</button>
  </div>
</template>

<script>
import { getLinkPreview } from 'link-preview-js';

import '@toast-ui/editor/dist/toastui-editor.css';
import { Editor } from '@toast-ui/vue-editor';

export default {
  name: 'App',
  data() {
    return {
      // 여러번 링크 달 경우 고유값 추적 -> replace
      id: 1,
      pasteLink: null,
      editorOptions: {
        hideModeSwitch: true
      },
      /* eslint-disable */
      domParExp: /(http(s)?:\/\/|www.)([a-z0-9\w]+\.*)+[a-z0-9]{2,4}([\/a-z0-9-%#?&=\w])+(\.[a-z0-9]{2,4}(\?[\/a-z0-9-%#?&=\w]+)*)*/gi,
      legacyData: `<p>Sodod</p>
<div align="">
 <div style="">
  <img src="https://t3.gstatic.com/images?q=tbn:ANd9GcQfm_aIU3zSPk739KJA4_ECMTDTwSlvc-kh9G1kQ4_WS90SUEOD_6ksSJsrNxcr" data-name="7CA5B3C5-7367-4AB8-A229-B21740FC7F58.png" data-volume="2022" style="display:block; max-width:100%; ">
 </div>
</div>`
    }
  },
  methods: {
    test( ) {
      console.log(this.$refs.tuiEditor.editor.getHtml())
    },  
    getThumbnailByPasteLink() {
      // 에디터 붙여넣기 이벤트 등록 
      this.$refs.tuiEditor.editor.eventManager.listen('paste', (event) => {
        this.pasteLink = null
        // 붙여넣은 텍스트가 링크일 경우 링크 데이터 가져오기
        event.data.clipboardData.getData('Text')
          .replace(this.domParExp, (link) => {
            this.pasteLink = link
            getLinkPreview(link)
              .then((data) => {
                const target = document.querySelector(`.link-loading-${this.id}`)
                const ele = `<div class="link-box"><div class="link-inner-box"><div class="link-img-box"><img src="${data.images[0]}" alt="${data.title}" /></div><div class="link-text-box"><h1>${data.title}</h1><p>${data.description}</p></div></div></div>`
                target.parentElement.insertAdjacentHTML('beforeend', ele)
                this.id++
                this.pasteLink = null
              });
          })          
      });

      // 에디터 내용 변경 이벤트 등록
      this.$refs.tuiEditor.editor.eventManager.listen('changeFromWysiwyg', () => {
        // 붙여넣기한 링크가 있을 때
        if (this.pasteLink !== null) {
          const pasteLink = this.pasteLink
          this.pasteLink = null
          const html = this.$refs.tuiEditor.editor.getHtml()
          const aLink = `<a href="${pasteLink}">${pasteLink}</a>`
          const loadingItem = `<a href="${pasteLink}">${pasteLink}</a><span class="link-loading link-loading-${this.id}"></span>`
          const newHtml = html.replace(aLink, loadingItem)
          this.$refs.tuiEditor.editor.setHtml(newHtml)
        }
      })
    },
  },
  mounted() {
    this.getThumbnailByPasteLink()
    setTimeout(() => {
      this.$refs.tuiEditor.editor.setHtml(this.legacyData)
    }, 1000);
  },
  components: {
    Editor,
  },
}
</script>

<style type="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
img { max-width:300px;}

.link-box {

}
.link-inner-box {
  max-width: 600px;  
  overflow: hidden;
  border: 1px solid #eee;
  background: #fefefe;
  text-align: left;

}
.link-img-box {
  max-width: 30%;
  float: left;
  padding: 15px;
  box-sizing: border-box !important;
}
.link-text-box {
  float: right;
  width:70%;
  padding-left: 15px;
  box-sizing: border-box !important;
}
.link-text-box h1 {
  font-size: 17px;
  border-bottom: 0;
}
.link-text-box p {
  text-align: left;
}
.link-inner-box img { 
  vertical-align: middle;
  width:100%;
}
.link-inner-box h1 { }
</style>
