<style scoped>
    .img-wrapper{
        position: relative;
        overflow: hidden;
    }
</style>

<template>
    <div class="img-wrapper" v-html="html" ref="wrapper" :style="{width: `${width}px`, height: `${height}px` }">
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
      template:{
        type: String,
        default: '<img src="{{imgUrl}}" style="position: absolute;top: 0;left: 0;bottom:0;right:0;width:100%;height:100%;object-fit:cover;transform: translate({{translateX}}px,{{translateY}}px)"><img src="{{modalUrl}}" style="position: absolute;top: 0;left: 0;bottom:0;right:0;width:100%;height:100%">',
      },
      data: {
        type: Object,
        default: function(){
            return {
                imgUrl: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1527594868974&di=824eebc4833a2a17fcc775a52327577a&imgtype=0&src=http%3A%2F%2Fimg.taopic.com%2Fuploads%2Fallimg%2F121029%2F240425-12102920212663.jpg',
                modalUrl: 'http://oxi280pze.bkt.clouddn.com/WechatIMG392.png',
                translateX: 0,
                translateY: 0
            }
        }
      }
  },
  data(){
    return{
        selfData: {},
        html: ''
    }
  },
  mounted(){
      const $wrapper = this.$refs['wrapper']
      let isMouseDown = false
      let startX, startY
      this.selfData = Object.assign({}, this.data)

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
          this.$emit('change',this.selfData, this.template)
      })

      this.html = compile(this.template, this.selfData)
  }
};
</script>