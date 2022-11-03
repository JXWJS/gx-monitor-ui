<template>
    <div class="home" v-loading="loading">


            <div class="header">
                <div class="r">
                    <div class="year">{{getNowFormatDate()}}<br>{{getWeekDay()}}</div>
                    <div class="time">{{time}}</div>
                </div>
              <div class="c"><h1 style="color: #01acff">自动化堵管在线检测系统</h1></div>
                <div class="l">
                  <el-breadcrumb separator-class="el-icon-arrow-right">
                    <el-breadcrumb-item  style="color:#029bd6" v-for="item in city" :key = "item" >{{item}}</el-breadcrumb-item>
                  </el-breadcrumb>
                </div>

            </div>




            <div class="left">

                <div  style="width:480px; height:300px; margin-top:80px">
                    <l1-com ref="l1"></l1-com >
                </div>

                 <div  style="width:480px; height:320px; margin-top:20px">
                    <l2-com ref="l2"></l2-com >
                </div>

                <div  style="width:480px; height:320px; margin-top:20px">
                    <l3-com ref="l3"></l3-com >
                </div>


            </div>

<div class="center">
  <div class="top" style="margin-top:60px">
    <map-com ref="map"></map-com>
  </div>

  <div class="bottom" style=" height:360px;margin-top:20px">
    <c1-com ref="c1"></c1-com >
  </div>
</div>



      <div class="right">

                <div  style="width:480px; height:115px; margin-top:80px">
                    <r4-com ref="r4"></r4-com >
                </div>
                <div  style="width:480px; height:180px;margin-top:5px">
                    <r1-com ref="r1"></r1-com >
                </div>

                 <div  style="width:480px; height:320px; margin-top:20px">
                    <r2-com ref="r2"></r2-com >
                </div>

                <div  style="width:480px; height:320px; margin-top:20px">
                    <r3-com ref="r3"></r3-com >
                </div>



            </div>





    </div>
</template>

<script>


import mapCom from "./light-monitor/map.vue"

import l1Com from "./light-monitor/l1.vue"
import l2Com from "./light-monitor/l2.vue"
import l3Com from "./light-monitor/l3.vue"

import c1Com from "./light-monitor/c1.vue"

import r1Com from "./light-monitor/r1.vue"
import r2Com from "./light-monitor/r2.vue"
import r3Com from "./light-monitor/r3.vue"
import r4Com from "./light-monitor/r4.vue"
import axios from "axios";

export default {
  components: {mapCom,l1Com,l2Com,l3Com,c1Com,r1Com,r2Com,r3Com,r4Com},
  data () {
    return {
        time:'',
        loading:false,
        city: [ "西安市", "阎良区", "长空路" ]
    }
  },
  mounted () {
      let that = this;
      that.loading = true;


      axios.get("http://192.168.1.99:80/MD/SUMMARY.DG?_TEMPLATE=FRS:SUMMTMPC",{})
          .then(e =>{
           if (e === undefined){
             that.$refs["r4"].data ={
               "list": [
                 {
                   "state": 2,
                   "time": new Date(),
                   "content": "数据传输关闭",
                   "equipment": "设备离线"
                 }
               ]
             };
           }else {
             that.$refs["r4"].data ={
               "list": [
                 {
                   "state": 0,
                   "time": new Date(),
                   "content": "数据传输开启",
                   "equipment": "设备在线"
                 }
               ]
             };
           }
          })


          that.$refs["l2"].data = {
            "title": "监控1",
            "src": "http://localhost:8801/live/test2.m3u8"
          };
          that.$refs["l3"].data = {
            "title": "监控2",
            "src": "http://192.168.1.50:20000/hls/test2.m3u8"
          };
          that.$refs["r2"].data = {
            "title": "监控3",
            "src": "http://192.168.1.50:20000/hls/test3.m3u8"
          };
          that.$refs["r3"].data = {
            "title": "监控4",
            "src": "http://192.168.1.50:20000/hls/test4.m3u8"
          };




          that.$refs["l1"].data = {
            "list": [
              {
                "title": "激光功率",
                "total": 2000,
                "num": 1055,
                "percentage": 20,
                "unit": "w/h"
              },
              {
                "title": "机器人速度",
                "total": 20,
                "num": 3,
                "percentage": 20,
                "unit": "m/s"
              },
              {
                "title": "粉桶1速率",
                "total": 20,
                "num": 3,
                "percentage": 20,
                "unit": "m/s"
              },
              {
                "title": "粉桶2速率",
                "total": 20,
                "num": 3,
                "percentage": 20,
                "unit": "m/s"
              }
            ]
          }

          that.$refs["r1"].data = {
            "columns": [
            "name",
            "num"
          ],
              "rows": [
            {
              "name": "A类",
              "num": 5
            },
            {
              "name": "B类",
              "num": 2
            },{
                  "name": "C类",
                  "num": 3
                }
          ]
          }




              that.loading = false;
      this.timer();
  },
  methods: {
    timer() {
  return setInterval(()=>{
        this.time = this.getNowTime()
},1000)}
  },
  destroyed() {
    clearInterval(this.timer)
  },
  getDataByPLC() {
    this.$get('mock/getDatazx',{})
        .then((response) => {

        })
  },
  index() {
    this.$router.push({ path: '/' })
  }
}
</script>

<style scoped>


.left{
 width: 500px;
 height: 1080px;
 position: absolute;
 left: 0;
 top: 0;
 /* background-color: #01067c; */
 background: linear-gradient(to right, rgba(1,6,124,1) 0%,  rgba(1,6,124,1) 70%, rgba(1,6,124,0) 100%);
 padding-left: 20px;
 padding-right: 20px;
 z-index: 1;
}

.right{
 width: 500px;
 height: 1080px;
 position: absolute;
 right: -20px;
 top: 0;
 /* background-color: #01067c; */
 background: linear-gradient(to right, rgba(1,6,124,0) 0%,  rgba(1,6,124,1) 30%, rgba(1,6,124,1) 100%);
 padding-left: 20px;
 padding-right: 20px;
 z-index: 1;
}

.bottom{
 width: 1920px;
 height: 240px;
 position: absolute;
 right: 0;
 bottom: 0;
 background: linear-gradient(to top, rgba(1,6,124,1) 0%,  rgba(1,6,124,1) 50%, rgba(1,6,124,0) 100%);
  z-index: 0; box-sizing: border-box; padding: 20px; padding-left:520px; padding-right:520px;

}

.top{
  position: absolute;
  background: linear-gradient(to top, rgba(1,6,124,1) 0%,  rgba(1,6,124,1) 50%, rgba(1,6,124,0) 100%);
  z-index: 0; box-sizing: border-box; padding: 20px; padding-left:520px; padding-right:520px;

}

</style>
