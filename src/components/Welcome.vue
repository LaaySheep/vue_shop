<template>
  <div class="demo-image__placeholder">
    <div class="block">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }" class="demonstration">首页</el-breadcrumb-item>
      </el-breadcrumb>
      <el-image :src="src"></el-image>
    </div>
    <div class="block">
      <span class="demonstration">自定义</span>
      <el-image :src="src">
      </el-image>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      src: 'https://image.baidu.com/search/detail?ct=503316480&z=&tn=baiduimagedetail&ipn=d&word=%E5%A3%81%E7%BA%B8&step_word=&ie=utf-8&in=&cl=2&lm=-1&st=-1&hd=undefined&latest=undefined&copyright=undefined&cs=3849111223,3170649452&os=3294848120,2909549196&simid=3849111223,3170649452&pn=0&rn=1&di=184800&ln=1960&fr=&fmq=1526269427171_R&fm=&ic=0&s=undefined&se=&sme=&tab=0&width=&height=&face=undefined&is=0,0&istype=2&ist=&jit=&bdtype=0&spn=0&pi=0&gsm=0&objurl=https%3A%2F%2Fgimg2.baidu.com%2Fimage_search%2Fsrc%3Dhttp%253A%252F%252Fup.enterdesk.com%252Fedpic%252F6e%252F7e%252Fdd%252F6e7edd30f22f2b136a696d885bc2f7bd.jpg%26refer%3Dhttp%253A%252F%252Fup.enterdesk.com%26app%3D2002%26size%3Df9999%2C10000%26q%3Da80%26n%3D0%26g%3D0n%26fmt%3Djpeg%3Fsec%3D1645172009%26t%3D5f2552aa5dba1baea949463a5b50edbb&rpstart=0&rpnum=0&adpicid=0&nojc=undefined'
    }
  },
  created() {
  },
  methods: {}
}
</script>
<style lang="less" scoped>

</style>
