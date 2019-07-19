<template>
  <div class="index">
    <!-- banner图 -->
    <div class="bannerBox">
      <swiper
        indicator-dots="true"
        indicator-color="rgba(0,0,0,0.6)"
        indicator-active-color="#ffffff"
        autoplay="true"
        interval="3000"
        duration="500"
        class="swiper"
      >
        <block v-for="(item,index) in arr" :key="index">
          <swiper-item>
            <image :src="item.pic" mode="widthFix" lazy-load="true" />
          </swiper-item>
        </block>
      </swiper>
    </div>
    <!-- 歌单详情 -->
    <div class="navBox">
      <!-- 热门歌单 -->
      <div>
        <div class="nav">
          <h2>热门歌单</h2>
          <h4 @tap="toMoreSong">更多></h4>
        </div>
        <ul class="hotSong">
          <li v-for="(item,index) in hotList" :key="item.id" v-if="6>index" @tap='tosonglist(item.id)'>
            <image mode="widthFix" lazy-load="true" :src='item.coverImgUrl'></image>
            <p class="hot_name">{{item.name}}</p>
            <p class="playcount">
              <image src='/static/images/ting2.png'></image>
              <span class="play_num">{{item.playCount}}万</span>
            </p>
          </li>
        </ul>
      </div>
      <!-- 最新mv -->
      <div>
        <div class="nav">
          <h2>MV</h2>
          <h4 @tap='toMoreMv'>更多></h4>
        </div>
        <ul class="mvBox">
          <li v-for="(item,index) in MVarr" :key="index" v-if="6>index" class="mvList">
            <image  lazy-load="true" :src='item.cover' @tap='toMvPlay(item.id)'></image>
            <p class="hot_name1">{{item.name}}</p>
            <p class="hot_jieshao">{{item.artistName}}</p>
            <p class="playcount">
              <image src='/static/images/songbf.png'></image>
              <span class="play_num">{{item.playCount}}万</span>
            </p>
          </li>
        </ul>
      </div>
      <!-- 最新音乐推荐 -->
      <div>
        <div class="nav">
          <h2>新歌首发</h2>
          <h4>更多></h4>
        </div>
        <ul class="newSongBox">
          <li class="newSong_list" v-for="item in newSong" :key="item.id" @tap='toPlay(item.id)'>
            <image mode="widthFix" lazy-load="true" :src="item.song.album.blurPicUrl" class="touxiang_pic"></image>
            <div class="newsong_center">
              <p class="newsong_name">{{item.name}}</p>
              <p class="newsong_con">{{item.song.album.company}}</p>
            </div>
            <image src='/static/images/songbf.png' class="play_pic"></image>
          </li>
        </ul>
      </div>
    </div>
    <div class="inde_foot">
       <p>
          <image src='/static/images/logo.gif' mode='widthFix'></image>
          <span class="logo_title">小打小闹小音乐</span>
       </p>  
        <p class="banquan">网易公司版权所有©1997-2019   杭州乐读科技有限公司运营</p>
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
      arr: [], //banner
      hotList: [], //最新歌单
      special: [], //专辑
      MVarr: [], //mv数据
      newSong: [] //最新音乐
    };
  },
  watch: {},
  computed: {},
  methods: {
    // 跳转歌曲播放页
    toPlay(id){
       var arr=JSON.stringify(this.newSong);
       wx.navigateTo({ url: '../play/main?id='+id +'&list='+encodeURIComponent(arr)});
    },
    toMvPlay(id){
       wx.navigateTo({ url: '../mvPlay/main?id='+id });
    },
    // 跳转到更多歌单
    toMoreSong() {
      wx.navigateTo({
        url: "../moreSongDan/main"
      });
    },
    // 跳转到更多专辑页
    toMoreZj(){
      wx.navigateTo({ url: '../moreZJ/main' });
    },
    // 跳转到更多mv页
    toMoreMv(){
      wx.navigateTo({ url: '../moreMv/main' });
    },
    // 跳转歌曲列表
    tosonglist(id){
      // console.log(id)
      wx.navigateTo({ url: '../songList/main?id='+id });
    }
  },
  mounted() {},
  onReady: function() {
    wx.showLoading({
      title: "努力加载中"
    });
      //bannner图
    wx.request({ 
      url: api.banner,
      success: res => {
        this.arr = res.data.banners;
      }
    });
       //热门歌单
    wx.request({
      url: api.hotSong,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        res.data.playlists.map(item => {
          return (item.playCount = (item.playCount / 10000).toFixed(1));
        });
        this.hotList = res.data.playlists;
      }
    });
      //最新mv
    wx.request({ 
      url: api.newMv,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        res.data.data.map(item => {
          return (item.playCount = (item.playCount / 10000).toFixed(1));
        });
        this.MVarr = res.data.data;
      }
    });
    //   //最新音乐
    wx.request({
    
      url: api.newSong,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        this.newSong = res.data.result;
      }
    });
  }
};
</script>
<style lang='stylus' scoped>
.index {
  .swiper {
    height: 150px;

    image {
      width: 100%;
    }
  }

  .nav {
    display: flex;
    justify-content: space-between;
    height: 50px;
    line-height: 50px;

    h2 {
      font-size: 16px;
      font-weight: bold;
      color: #000;
    }

    h4 {
      font-size: 12px;
      color: #999;
    }
  }

  .navBox {
    padding: 0 10px;

    .hotSong {
      margin-right: -7px;
      overflow: hidden;

      li {
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

    // mv
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

    // 最新音乐
    .newSongBox {
      .newSong_list {
        display: flex;
        height: 50px;
        widht: 100%;
        margin-bottom: 10px;

        .touxiang_pic {
          width: 50px;
          height: 50px;
          border-radius: 5px;
        }

        .newsong_center {
          flex: 1;
          margin-left: 10px;
          padding-top: 5px;

          .newsong_name {
            font-size: 16px;
            color: #666;
            line-height: 1.6;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
          }

          .newsong_con {
            font-size: 12px;
            color: #999;
            line-height: 1.4;
          }
        }

        .play_pic {
          width: 30px;
          height: 30px;
          margin: 5px 10px 0 0;
        }
      }
    }
  }
  .inde_foot{
    // height:50px;
    background :#DD001B;
    text-align :center;
    padding:10px 0;
    image{
      width:40px;
      height:40px;
      border-radius :50%;
      display :inline-block;
      vertical-align :middle;
      margin-right:10px;
    }
    .logo_title{
      line-height :40px;
      display :inline-block;
      color:white;
      font-size :20px;
    }
    .banquan{
      font-size 12px;
      color:#fff;
      height 20px;
      line-height 20px;
    }
  }
}
</style>
