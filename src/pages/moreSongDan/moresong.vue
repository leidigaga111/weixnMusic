<template>
  <div class="moreSong">
    <ul class="hotSong">
      <li v-for="item in arr" :key="item.id" class="more_list" @tap='tosonglist(item.id)'>
        <image mode="widthFix" lazy-load="true" :src='item.coverImgUrl'></image>
        <p class="hot_name">{{item.name}}</p>
        <p class="playcount">
          <image src='/static/images/ting2.png'></image>
          <span class="play_num">{{item.playCount}}万</span>
        </p>
      </li>
   
    </ul>
    <div  class="end" v-if='isShow'>
          小老弟，到底了~~
    </div>
     <div class="xin" v-if='isShow1'>
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
      pageSize:100,
      arr: [],
      isShow:false,
      isShow1:true,
    };
  },
  watch: {},
  computed: {},
  methods: {
    // 跳转歌曲列表页
    tosonglist(id){
      // console.log(id)
      wx.navigateTo({ url: '../songList/main?id='+id });
    },
  getDat(n){     
      wx.showLoading({
      title: '努力加载中',
    })
     wx.request({
      // url: api.goodSong + "?cat=全部&page="+n,
      url: api.hotSong + "?limit=100",
      success: res => {
        wx.hideLoading();
        this.isShow1=false;
        // if (res.data.data.length==0) {
        //    this.isShow=true;
        //   return;
        // }
        // console.log(res);
     
        res.data.playlists.map(item => {
          return (item.playCount = (item.playCount / 10000).toFixed(1));
        });
        var arr=this.arr;
        var data=arr.concat(res.data.playlists)
        // console.log(res.data.data)
        this.arr = data;
      }
    });
      }
  },
  mounted() {},
  //  生命周期函数--监听页面初次渲染完成
  onReady: function() {
    this.getDat(this.page)
  },

//    * 页面上拉触底事件的处理函数
  onReachBottom: function () {   
    //  var n=this.page+30
    //  this.page=n;
    //  this.getDat(n)
  },
  onUnload: function () {
    this.arr=[];
    this.isShow1=true;
  },
};
</script>
<style lang='stylus' scoped>
.moreSong{
    padding:10px;
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
.hotSong {
  margin-right: -7px;
  overflow: hidden;

  .more_list {
    width: 113px;
    height: 153px;
    margin-right: 7px;
    margin-bottom: 7px;
    overflow: hidden;
    float: left;
    position: relative;

    image {
      width: 100%;
      border-radius: 8px;
    }

    .hot_name {
      font-size: 13px;
      color: #666;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
      line-height: 1.4;
      height: 36px;
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
