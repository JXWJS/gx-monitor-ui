<template>
    <div class="boxbg"  style="position: relative;">
        <div class="title"><span>运维概览信息</span></div>
        <i class="i1"></i>
        <i class="i2"></i>
        <i class="i3"></i>
        <i class="i4"></i>

        <div class="c flex">
            <el-row :gutter="10" style="width:100%">
                <el-col :span="12" v-for="(item,index) in data.list" :key="index">
                    <div class="txtbg">
                        <div class="flex flex1 flexc" style="padding-left:10px;">
                            <span class="t1 errtips">{{item.title}}</span>
                            <span class="t2">{{item.num}}<i>{{item.unit}}</i></span>
                        </div>
                        <div class="flex flex1 center flexc" style="max-width:70px;">
                            <el-progress type="dashboard"  :percentage="item.percentage"  :width="60" :stroke-width="6" color="#e8f60d"></el-progress>
                            <span style="margin-top:-5px; font-size:15px;color:#2887ac">总:{{item.total}}</span>
                        </div>
                    </div>
                </el-col>
            </el-row>
        </div>

    </div>
</template>
<script>
    var that;
    export default {
        watch:{
        },
        data () {
            return {
                data:{
                    list:[]
                },
            }
        },

        methods: {

        },
        mounted () {
        },
    }
</script>

<style>
.errtips {
  color:#e40438;
  animation-name: errtips;
  animation-duration: 0.5s;
}
@keyframes errtips {
  10% {
    transform: translateY(1px);
  }
  20% {
    transform: translateY(-1px);
  }
  30% {
    transform: translateY(1px);
  }
  40% {
    transform: translateY(-1px);
  }
  50% {
    transform: translateY(1px);
  }
  60% {
    transform: translateY(-1px);
  }
  70% {
    transform: translateY(1px);
  }
  80% {
    transform: translateY(-1px);
  }
  90% {
    transform: translateY(1px);
  }
  100% {
    transform: translateY(0);
  }
}
</style>


