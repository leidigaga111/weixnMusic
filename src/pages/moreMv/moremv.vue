<template>
  <div class="moreMv">
    <ul class="mvBox">
      <li v-for="(item,index) in MVarr" :key="index" class="mvList" @tap='toMvPlay(item.id)'>
        <image  lazy-load="true" :src='item.cover'></image>
        <p class="hot_name1">{{item.name}}</p>
        <p class="hot_jieshao">{{item.artistName}}</p>
        <p class="playcount">
          <image src='/static/images/songbf.png'></image>
          <span class="play_num">{{item.playCount}}万</span>
        </p>
      </li>
    </ul>
    <div class="end" v-if='end'>
       小老弟，到底了~~
    </div>
      <div class="xin" v-if='isShow'>
        <image src='/static/images/xin.gif'> </image>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import api from "../../utils/api";
export default {
  components: {},
  props: [],
  data() {
    return {
      page: 0,
      MVarr: [],
      isShow:true,
      end:false,
    };
  },
  watch: {},
  computed: {},
  methods: {
    getData(n) {
      wx.showLoading({
      title: "努力加载中"
      });
      wx.request({
        url: api.newMv+"?limit=100",
        success: res => {
          // console.log(res);
          wx.hideLoading();
           this.isShow=false;
            this.end=true;
          // if(res.data.data.length==0){
          //   this.end=true;
          //   return
          // }        
          res.data.data.map(item => {
            return (item.playCount = (item.playCount / 10000).toFixed(1));
          });
          var data = this.MVarr.concat(res.data.data);
          this.MVarr = data;
        }
      });
    },
    // 跳转mv播放页面
    toMvPlay(id){
       wx.navigateTo({ url: '../mvPlay/main?id='+id });
    },
  },
  mounted() {},
  onReady: function() {
 
    this.getData(this.page);
  },
  // 下拉刷新
  onReachBottom: function() {
    // var n = this.page + 30;
    // this.page = n;
    // this.getData(n);
  },
  onUnload: function () {
    this.MVarr=[];
    this.isShow=true;
  },
};
</script>
<style lang='stylus' scoped>
.moreMv {
  padding: 10px;
}
.xin{
    width:240px;
    height:240px;
    position :absolute;
    left:50%;
    top:50%;
    transform :translate(-50%,-50%)
    image{
        width:100%;
        height:100%;
    }
}
.mvBox {
  overflow: hidden;
  margin-right: -8px;

  .mvList {
    width: 173px;
    height: 140px;
    margin-right: 8px;
    overflow: hidden;
    float: left;
    position: relative;

    image {
      width: 100%;
      height: 97px;
      border-radius: 8px;
    }

    .hot_name1 {
      font-size: 13px;
      color: #666;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      height: 17px;
    }

    .hot_jieshao {
      font-size: 11px;
      color: #999;
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      height: 15px;
    }

    .playcount {
      position: absolute;
      right: 5px;
      top: 5px;
      // width:55px;
      padding: 0 3px;
      text-align: center;
      background: rgba(0, 0, 0, 0.4);
      border-radius: 2px;

      image {
        width: 15px;
        height: 15px;
        vertical-align: -2px;
        margin-right: 5px;
      }

      .play_num {
        font-size: 12px;
        color: white;
      }
    }
  }
}
.end{
 height:30px;
 line-height 30px;
 font-size 14px;
 color:#999;
 text-align center;
 }
</style>
