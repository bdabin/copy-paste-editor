<template>
  <div id="app">
    <editor-content :editor="editor" @paste="test" ref="editor" />
      <editor-menu-bar :editor="editor" v-slot="{ commands }">
        <button
          class="menubar__button"
          @click="showImagePrompt(commands.image)"
        >
          image upload
        </button>
    </editor-menu-bar>
    <div>
      <div class="link-box">
    <img :src="img" alt="" >
    <h1 ref="title">{{title}}</h1>
    <p>{{description}}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { Editor, EditorContent, EditorMenuBar } from 'tiptap'
import { getLinkPreview } from 'link-preview-js';
import {
  Link, Image, Collaboration
} from 'tiptap-extensions'

export default {
  name: 'App',
  data() {
    return {
      url: '',
      img: '',
      title: '',
      editorText: '',
      pasteLink: null,
        editorOptions: {
          hideModeSwitch: true
        },
      /* eslint-disable */
      domParExp: /(http(s)?:\/\/|www.)([a-z0-9\w]+\.*)+[a-z0-9]{2,4}([\/a-z0-9-%#?&=\w])+(\.[a-z0-9]{2,4}(\?[\/a-z0-9-%#?&=\w]+)*)*/gi,
      description: '',
       editor: new Editor({
        extensions: [
          new Link(), new Image(), new Collaboration()
        ],
        content: '<div class="test">This is just a borifsafsafsang paragraph</div>',
        onUpdate: ({getHTML}) => {
          if(this.pasteLink !== null) {
            // const htmlData = getHTML()
            // const resultHtml = htmlData.replace(this.pasteLink, `<span>${this.pasteLink}</span>`)
            // this.editor.setContent(resultHtml)
            // this.$refs.editor.setContent(resultHtml)
            // console.log(resultHtml)
            // console.log('setContent')
            // this.pateLink = null
          }
        },
        onPaste: (_ ,event) => {
          this.pasteLink = null
          event.clipboardData.getData('Text').replace(this.domParExp, (link) => {
            this.pasteLink = link
            this.getLinkData()
          })          
        }
      }),
    }
  },

  methods: {
    getLinkData() {
      if(this.pasteLink === null) {
        return false
      }
      getLinkPreview(this.pasteLink)
        .then((data) => {
          this.img = data.images[0]
          this.title = data.title
          this.description = data.description
          // const ele = `<div class="link-box"><img src="${data.images[0]}" alt="${data.title}"><h1>${data.title}</h1><p>${data.description}</p></div>`
          // const ele = `<img contenteditable="true" src="${data.images[0]}" alt="${data.title}"><h1>${data.title}</h1><p>${data.description}</p>`
          // document.querySelector('.ProseMirror').append(ele)

        // command({  src: data.images[0] })

        this.editor.commands.image({ src: data.images[0] })
        console.log(this.editor)
        const text = data.title

        const mark = this.editor.schema.nodes.doc.create()
        const from = this.editor.state.selection.from
        const transaction = this.editor.state.tr.insertText(text + ' ')
        console.log(transaction)
        // transaction.addMark(from, from + text.length, mark)
        // this.editor.view.dispatch(transaction)


        });
    },
    test() {
      console.log('hi')
    },
    showImagePrompt(command) {
      console.log(command)
      const src = prompt('Enter the url of your image here')
      if (src !== null) {
        command({ src })
      }
    },
  },
  beforeDestroy() {
      // Always destroy your editor instance when it's no longer needed
      this.editor.destroy()
    },
    mounted() {
      

  },
 components: {
   EditorMenuBar,
    EditorContent,
  },
// components: {
//   Editor,
// },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

}
img { max-width:300px;}
</style>
