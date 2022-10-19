<template>
  <div class="boxbg"  style="position: relative;">
    <div class="title"><span>三维场景</span></div>

    <div class="elem">
      <img src="~@/../static/camera/IM000001.jpg" style="width: 100%;max-height: 300px">
    </div>

    <div class="elem">
      <img src="~@/../static/camera/IM000002.jpg" style="width: 100%;max-height: 300px">
    </div>
  </div>

</template>
<script>


    var that;
    export default {
        watch:{
            data(newValue,oldValue){
            }
        },
        data () {

        },

        methods: {

        },
        mounted () {
        },
    }
</script>

<style scoped>
.elem{
  margin-top:40px;
  margin-right:20px;
  flex:1;
  background-color:#000000;
  height:300px;
}
.grid-content{width: 195px; height: 78px;  background-image: url('~@/../static/bggb.png'); margin-top: 10px; margin-left:5px; display: flex; }
</style>

