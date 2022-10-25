<template>
  <div class="boxbg"  style="position: relative;">
    <div class="title"><span>三维场景</span></div>
    <i class="i1"></i>
    <i class="i2"></i>
    <i class="i3"></i>
    <i class="i4"></i>

    <div class="elem">
      <img id="viedo-canvas1" src="~@/../static/camera/IM000001.jpg" style="width: 100%;max-height: 270px;margin-top: 5px;margin-left: 10px;">
      <span style="position: absolute; bottom: 280px; left: 330px;color: white">3D Camera</span>
    </div>

    <div class="elem">
      <img id="viedo-canvas2" src="~@/../static/camera/IM000002.jpg" style="width: 100%;max-height: 270px;margin-top: 5px;margin-left: 10px;">
      <span style="position: absolute; bottom: 280px; left: 820px;color: white">定位</span>
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
          return {

          }

        },

        methods: {
          connect() {
            // eslint-disable-next-line no-undef
            let socket = new ReconnectingWebSocket("ws://192.168.0.187:8081/websocket/img",null,{debug: true, reconnectInterval: 3000, maxReconnectAttempts: 2});
            const img1 = document.getElementById("viedo-canvas1");
            const img2 = document.getElementById("viedo-canvas2");
            socket.onerror = err => {
              console.log(err);
            }

            socket.onopen = event => {
              console.log(event);
            };
            socket.onmessage = mess => {
              console.log(mess.data);
              let address = mess.data.split(";");
              if (address[0] === "0"){
                img1.src = 'data:image/png;base64,' + address[0];
              }else if (address[0] === "1"){
                img2.src = 'data:image/png;base64,' + address[1];
              }

            };
            socket.onclose = () => {
              console.log("连接关闭");
            };

          }
        },
        mounted () {
          this.connect();
        },
    }
</script>

<style scoped>
.elem{
  margin-top:40px;
  margin-right:20px;
  flex:1;
  background: linear-gradient(to right, rgba(1,6,124,0) 0%,  rgba(0,0,0,0) 50%, rgba(0,0,0,0.5) 100%);
  height:300px;
}
.grid-content{width: 195px; height: 78px;  background-image: url('~@/../static/bggb.png'); margin-top: 10px; margin-left:5px; display: flex; }
</style>

