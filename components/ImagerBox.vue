<template>
  <div class="imager-box">
    <div class="imager-box__view-box">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="svg"
        width="720"
        height="540"
        :viewBox="`0 0 ${svgsettings.width} ${svgsettings.height}`"
        ref="svg"
        >
      <rect x="10" y="10" width="380" height="180" rx="10" ry="10" fill="#e74c3c" />
      <text
        :x="pos.x"
        :y="pos.y"
        font-size="36"
        class="text"
        :stroke="color01"
        :fill="color02"
        stroke-width="1"
        @mousedown.stop="touchStart($event)"
        @mouseup.stop="touchEnd($event)"
      >
        {{ viewtext }}
      </text>
      </svg>
      <p>svg エリア</p>
      <div class="canvas-are">
        <canvas id="view" ref="viewing" >
        </canvas>
      </div>
      <p>canvas エリア</p>
    </div>
    <div class="imager-box__input-box">
      <input type="file" ref="file" class="file" name="name" @change="onFileChange" />
      <input type="text" class="input" v-model="viewtext" name="viewtext" />
      <input type="number" class="number" v-model="font_size" step="0.1" name="font_size" />
      <input type="color" class="color" v-model="color01" name="color01" />
      <input type="color" class="color" v-model="color02" name="color02" />
      <button @click="fileDownLoad" >合成</button>
      <button @click="canvasDownLoad" >down load</button>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed } from '@vue/composition-api'

export default defineComponent({
  name:'ImagerBox',
  data() {
    return {
      viewtext: '',
      font_size: 2,
      svgsettings:{
        width:720,
        height:540,
      },
      pos: {
        x:0,
        y:90
      },
      prevPos: {
        x:0,
        y:90
      },
      color01: '#cccccc',
      color02: '#ffffff',
      filedata:false,
      fileSrcData:null,
      image_name: '',
      image_url: '',
      svg_url: ''
    }
  },
  mounted() {
  },
  methods: {
    // setUp(props){
    //   const isMouseDown = ref(false);
    //   const pos = reactive({
    //     x:0,
    //     y:100,
    //   })
    //   const prePos = reactive({
    //     x: pos.x,
    //     y: pos.y,
    //   })
    //   const touchStart = (e) => {
    //     isMouseDown.value = true
    //     prePos.x = e.offsetX
    //     prePos.y = e.offsetY
    //   }
    //   const touchMove = (e) => {
    //     if(!isMouseDown.value)return;
    //     const movedX = e.offsetX - prevPos.x;
    //     const movedY = e.offsetX - prevPos.y;
    //     pos.x += movedX
    //     pos.y += movedY

    //     prePos.x = e.offsetX
    //     prePos.y = e.offsetY
    //   }
    //   const touchEnd = () => {
    //     isMouseDown.value = false
    //   }

    //   return {
    //     pos,
    //     touchStart,
    //     touchMove,
    //     touchEnd
    //   }
    // },
    touchStart(e){
        this.isMouseDown = true
        this.prevPos.x = e.offsetX
        this.prevPos.y = e.offsetY
        e.target.addEventListener('mousemove',this.touchMove)
    },
    touchMove(e){
        if(!this.isMouseDown)return;
        let movedX = e.offsetX - this.prevPos.x;
        let movedY = e.offsetY - this.prevPos.y;
        this.pos.x += movedX
        this.pos.y += movedY

        this.prevPos.x = e.offsetX
        this.prevPos.y = e.offsetY
    },
    touchEnd() {
      this.isMouseDown = false
    },
    onFileChange(e){
      let fr = new FileReader();
      const fileData = e.target.files[0];
      this.svg_url = window.URL.createObjectURL(fileData)
      if(!fileData.type.match('image.*')){
        alert('画像を選択してください。');
        return;
      }
      fr.onload = (evt) => {
         this.fileSrcData = evt.target.result;
         this.canvasDraw()
      }
      fr.readAsDataURL(fileData)
    },
    canvasDraw(){
      const canvas = this.$refs.viewing;
      const ctx = canvas.getContext('2d');
      canvas.width = 400;
      canvas.height = 400;
      ctx.clearRect(0,0,canvas.width,canvas.height);
      const image = new Image();
      image.src = this.fileSrcData;
      image.onload = function(){
         ctx.drawImage(image,0,0,canvas.width,canvas.height);
      }
    },
    remove(){
       this.c = false
    },
    fileDownLoad(){
       const svg = this.$refs.svg
       const canvas_img = this.$refs.viewing
       const ctx_img = canvas_img.getContext('2d')
       const svgData = new XMLSerializer().serializeToString(svg)
       const canvas = document.createElement("canvas")
       canvas.width = svg.width.baseVal.value
       canvas.height = svg.height.baseVal.value
       const ctx = canvas.getContext('2d')
       let DOMURL = self.URL || self.webkitURL || self
       const re_svg = new Blob([svgData],{type:"image/svg+xml"})
       const re_url = URL.createObjectURL(re_svg)
       const imageEl = new Image()
       imageEl.crossOrigin = 'Anonymous'
       imageEl.src = re_url;
       imageEl.onload = function(){
         ctx_img.drawImage(imageEl,0,0,canvas.width,canvas.height)
         ctx.drawImage(imageEl,0,0,canvas.width,canvas.height)
        //  const link = document.createElement("a")
        //  link.href = canvas.toDataURL("image/png")
        //  link.setAttribute("download","image.png")
        //  link.click()
       }
       imageEl.src = "data:image/svg+xml;charset=utf-8;base64," + btoa(unescape(encodeURIComponent(svgData)))
    },
    canvasDownLoad(){
      const canvas_img = this.$refs.viewing
      const ctx_img = canvas_img.getContext('2d')
      const link = document.createElement("a")
      link.href = canvas_img.toDataURL("image/png")
      link.download = "image.png"
      link.click()
    }
  },
})
</script>>

<style>
.canvas-are,
.svg {
  border: 1px solid #ccc;
}
.canvas-are {
  margin: 40px 0 0;
}

@keyframes appear {
  0% {
    opacity: 0;
  }
}
</style>
