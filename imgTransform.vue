<style scoped>
    .img-wrapper{
        display: inline-block;
        overflow: hidden;
        cursor: move;
        transform-origin: 0 0 0;
    }
</style>

<template>
    <div class="img-wrapper" v-html="html" ref="wrapper" :style="{transform: `scale(${wrapperScale})`}">        
    </div>
</template>

<script>

function compile(template, data) {
    var i = 0,fragment = ''

    // 遍历数据集合里的每一个项，做相应的替换
    function replace(obj) {
        var t, key, reg

        for (key in obj) {
            reg = new RegExp('{{' + key + '}}', 'ig')
            t = (t || template).replace(reg, obj[key])
        }
        return t
    }

    return replace(data)
}

export default {
  name: 'img-transform',
  props: {
      width: {
          type: Number,
          default: 500
      },
      height: {
          type: Number,
          default: 500
      },
      move:{
        type: Boolean,
        default: false
      },
      template:{
        type: String,
        default: '<div style="position: relative; width:{{modalWidth}}px; height:{{modalHeight}}px;background: url({{modalUrl}}) center no-repeat;background-size: contain"><img src="{{imgUrl}}" style="position: absolute;z-index: -1;top: {{marginTop}}px;left: {{marginLeft}}px;bottom:0;right:0;width:{{flexImgWidth}}px;height:{{flexImgHeight}}px;transform: translate({{translateX}}px,{{translateY}}px) scale({{scale}});"></div>',
      },
      data: {
        type: Object,
        default: function(){
            return {
                imgUrl: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1528457678683&di=0cd5026ea2686d997f0e11891e763af9&imgtype=0&src=http%3A%2F%2Fimg3.redocn.com%2Ftupian%2F20151026%2Fyingerbaobaogaoqingtupian_5186403.jpg',
                imgWidth: 1000,
                imgHeight: 667,
                flexImgWidth: 610,
                flexImgHeight: 'auto',
                modalUrl: 'http://oxi280pze.bkt.clouddn.com/WechatIMG581.png',
                modalWidth: 750,
                modalHeight: 750,
                boxWidth: 610,
                boxHeight: 400,
                marginLeft: 70,
                marginTop: 52,

                translateX: 0,
                translateY: -10,
                scale: 1
            }
        }
      }
  },
  data(){
    return{
        selfData: {},
        html: '',
        wrapperScale:1
    }
  },
  mounted(){
      const $wrapper = this.$refs['wrapper']
      let isMouseDown = false
      let startX, startY
      this.selfData = Object.assign({}, this.data)
      this.wrapperScale = this.width / this.selfData.modalWidth
      
      if(this.move){
        $wrapper.addEventListener('mousedown', (e)=>{
            startX = e.x
            startY = e.y
            isMouseDown = true
            this.oldX = this.selfData.translateX
            this.oldY = this.selfData.translateY 
        })

        $wrapper.addEventListener('mousemove', (e)=>{
            if(!isMouseDown){return}
            e.preventDefault()
            const translateX = e.x - startX 
            const translateY = e.y - startY
            this.selfData.translateX = translateX + this.oldX
            this.selfData.translateY = translateY + this.oldY

            this.html = compile(this.template, this.selfData)
            return false
        })
        
        $wrapper.addEventListener('mouseup', ()=>{
            isMouseDown = false
            this.$emit('change', this.selfData, this.template)
        })
        
        $wrapper.addEventListener('mousewheel', (e)=>{
            e.preventDefault()
            const deltaY = Math.floor(e.deltaY)

            if(deltaY<0 && (this.selfData.scale + deltaY/1000) <= 1){
                return
            }
            this.selfData.scale =  Math.floor(this.selfData.scale*1000 + deltaY)/1000
            this.html = compile(this.template, this.selfData)

            this.$emit('change', this.selfData, this.template)
        })

      }
      
      this.html = compile(this.template, this.selfData)
  },
  methods:{
      formatData(){
          const translateX = this.selfData.translateX/this.width
          const translateY = this.selfData.translateY/this.height
          const newData = Object.assign({}, this.selfData, {translateX, translateY})

          this.$emit('change', newData, this.template)
      }
  }
};
</script>