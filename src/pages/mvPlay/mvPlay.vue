<template>
  <div class="wrapper">
    <!-- 视频区 -->
    <div class="videobox">
      <p class="playerror" v-if="isError">视频播放出现问题，客观先观看其他视频~~</p>
      <video
        id="video"
        :src="url"
        controls="true"
        autoplay="true"
        show-mute-btn="true"
        :title="name"
        enable-play-gesture="true"
        object-fit="fill"
        poster="http://i2.w.yun.hjfile.cn/doc/201303/78ebff0b-3b4b-4695-93b7-4b5f62312ce6_03.jpg"
        @ended="end"
        @timeupdate="update"
        @waiting="wait"
        @error="playError"
      ></video>
    </div>
    <!-- 评论，详情导航 -->
    <div class="mv_nav">
      <p :class="[isShow?'active':'']" @tap="toInfo">详情</p>
      <p :class="[!isShow?'active':'']" @tap="toPinglun">热门评论</p>
    </div>
    <!-- 详情和相似mv -->
    <div class="mv_info" v-if="isShow">
      <p class="mv_name">{{name}}</p>
      <p class="mv_singerBox">
        <span>歌手:</span>
        <span class="mv_singer_name">{{singer}}</span>
      </p>
      <p class="mv_tc">
        <span>发布时间: {{time}}</span>
        <span class="mv_playcount">{{playcount}}次播放</span>
      </p>
      <div>
        <h2 class="mv_jieshao">MV介绍</h2>
        <p class="mv_jieshao_con">
          {{info}}
          <span class="mv_con_more" @tap="infoMore" v-if="infoData.length>70">{{text}}</span>
        </p>
        <p class="mv_jieshao_no" v-if="info.length==0">暂无介绍</p>
      </div>
      <div class="simiBox">
        <h2 class="simi_mv">相似MV</h2>
        <ul class="simiList">
          <li v-for="item in simiArr" :key="item.id" @tap="toPlay(item.id)">
            <img :src="item.cover" class="simi_pic" />
            <img src="/static/images/bf2.png" class="misi_playpic" />
            <div class="simi_con">
              <p class="simi_name">{{item.name}}</p>
              <p class="simi_singer">{{item.artistName}}</p>
              <p class="simi_playcount">{{item.playCount}}次播放</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <!-- 评论区 -->
    <div v-if="!isShow" class="mvComent">
      <ul >
        <li v-for="item in pinglunArr" :key="item.id">
          <p class="mv_pl_pic">
            <img :src="item.user.avatarUrl" />
          </p>
          <div class="mv_pl_con">
            <p class="mv_pl_name">{{item.user.nickname}}</p>
            <p class="mv_pl_text">{{item.content }}</p>
            <p class="mv_pl_con_bottom">
              <span class="mv_pl_time">{{item.time}}</span>
              <span class="mv_pl_like">
                <img src="/static/images/like.png" alt />
                ({{item.likedCount}})
              </span>
            </p>
          </div>
        </li>
      </ul>
      <p class="pinglun_no" v-if="pinglunArr.length==0">少年~快去展示你的才艺吧！成为热门评论</p>
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
      id: "",
      video: "",
      url: "",
      name: "",
      singer: "",
      info: "",
      infoData: "",
      time: "",
      playcount: "",
      isShow: true,
      text: "详情",
      isMore: true,
      simiArr: [], //相似mv
      isError: false,
      pinglunArr: []
    };
  },
  watch: {},
  computed: {},
  methods: {
    toInfo() {
      this.isShow = true;
    },
    toPinglun() {
      this.isShow = false;
    },
    // 再次跳转播放页面
    toPlay(id) {
      this.video.pause();
      this.id = id;
      this.getUrl();
      this.getInfo();
      this.getSimi();
      this.getPingLun();
    },
    // 播放结束
    end() {
      wx.showLoading({
        title: "视频跳转中~~"
      });
      setTimeout(() => {
        wx.hideLoading();
        var id = this.simiArr[0].id;
        this.toPlay(id);
      }, 2000);
    },
    // 当视频播放缓冲的时候
    wait() {
      wx.showLoading({
        title: "努力加载中"
      });
    },
    // 视频播放错误
    playError() {
      this.isError = true;
    },
    // 播放更新的事件
    update() {
      wx.hideLoading();
    },
    // 点击详情事件
    infoMore() {
      if (this.isMore) {
        this.isMore = false;
        this.text = "收起";
        this.info = this.infoData.slice(0);
      } else {
        this.isMore = true;
        this.text = "详情";
        this.info = this.infoData.slice(0, 70);
      }
    },
    // 获取mv播放地址
    getUrl() {
      this.video = wx.createVideoContext("video");
      wx.showLoading({
        title: "努力加载中"
      });
      wx.request({
        url: api.mvUrl + "?id=" + this.id,
        success: res => {
          // console.log(res);
          this.url = res.data.data.url;
        }
      });
    },
    // 获取mv详情信息
    getInfo() {
      wx.request({
        url: api.mvInfo + "?id=" + this.id,
        success: res => {
          // console.log(res);
          var data = res.data.data.data;
          this.name = data.name;
          this.singer = data.artistName;
          if (data.desc) {
            this.infoData = data.desc;
            this.info = this.infoData.slice(0, 70);
          }else{
            this.info='打开方式不对,小子也么有办法了~~'
          }
          this.time = data.publishTime;
          this.playcount =
            data.playCount > 10000
              ? (data.playCount / 10000).toFixed(1) + "万"
              : data.playCount;
        }
      });
    },
    // 获取相似mv
    getSimi() {
      wx.request({
        url: api.simiMv + "?mvid=" + this.id,
        success: res => {
          // console.log(res);
          var data = res.data.mvs;
          data.map(item => {
            return (item.playCount =
              item.playCount > 10000
                ? (item.playCount / 10000).toFixed(1) + "万"
                : item.playCount);
          });
          this.simiArr = data;
        }
      });
    },
    // 获取mv评论
    getPingLun() {
      wx.request({
        url: api.MvComment + "?id=" + this.id + "&type=1",
        success: res => {
          // console.log(res);
          res.data.hotComments.map(item => {
            item.time = this.timeFullter(item.time);
          });
          this.pinglunArr = res.data.hotComments;
        }
      });
    },
    // 处理时间函数
    timeFullter(time) {
      var date = new Date(time);
      var year =
        date.getFullYear() < 10 ? "0" + date.getFullYear() : date.getFullYear();
      var month =
        date.getMonth() < 10
          ? "0" + (date.getMonth() + 1)
          : date.getMonth() + 1;
      var day = date.getDate() < 10 ? "0" + date.getDate() : date.getDate();
      var hours =
        date.getHours() < 10 ? "0" + date.getHours() : date.getHours();
      var minu =
        date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes();
      return year + "年" + month + "月" + day + "日" + " " + hours + ":" + minu;
    }
  },
  mounted() {},
  onLoad: function(options) {
    this.id = options.id;
  },
  //   监听页面显示
  onShow: function() {
    this.getUrl();
    this.getInfo();
    this.getSimi();
    this.getPingLun();
  },
  // 页面卸载之前
  onUnload: function() {
    this.url = "";
    this.isShow = true;
    this.isMore = true;
    this.isError = false;
  }
};
</script>
<style lang='stylus' scoped>
.videobox {
  width: 100%;
  height: 225px;
  position: relative;

  .playerror {
    width: 100%;
    height: 100%;
    padding-top: 100px;
    box-sizing: border-box;
    position: absolute;
    left: 0;
    top: 0;
    text-align: center;
    color: white;
    background: rgba(0, 0, 0, 0.7);
    z-index: 10;
  }

  video {
    width: 100%;
  }
}

.mv_nav {
  display: flex;
  justify-content: space-around;
  height: 40px;
  line-height: 40px;
  font-size: 14px;
  border-bottom: 1px solid #f4f4f4;

  .active {
    color: #52A2FF;
    border-bottom: 1px solid #52A2FF;
  }
}

// 详情区
.mv_info {
  padding: 0 10px 10px;

  .mv_name {
    font-size: 18px;
    color: black;
    font-weight: 600;
    line-height: 30px;
  }

  .mv_singerBox {
    font-size: 13px;
    color: #999;
    line-height: 25px;

    .mv_singer_name {
      color: #52A2FF;
      margin-left: 10px;
    }
  }

  .mv_tc {
    font-size: 13px;
    line-height: 25px;
    color: #999;

    .mv_playcount {
      margin-left: 15px;
    }
  }

  .mv_jieshao {
    font-size: 16px;
    font-weight: 600;
    color: black;
    line-height: 30px;
  }

  .mv_jieshao_con {
    font-size: 13px;
    color: rgba(0, 0, 0, 0.6);
    line-height: 1.5;

    .mv_con_more {
      color: #52A2FF;
      font-weight: bold;
    }
  }

  .mv_jieshao_no {
    text-align: center;
    font-size: 14px;
    line-height: 25px;
    color: #666;
  }

  // 相似mv列表
  .simiBox {
    margin-top: 15px;

    .simi_mv {
      font-size: 16px;
      font-weight: 600;
      color: black;
      line-height: 30px;
    }

    .simiList {
      margin-top: 10px;

      li {
        width: 100%;
        display: flex;
        box-shadow: 0 0 5px black;
        border-radius: 5px;
        overflow: hidden;
        position: relative;
        margin-bottom: 10px;

        .simi_pic {
          width: 90px;
          height: 80px;
        }

        .misi_playpic {
          width: 30px;
          height: 30px;
          position: absolute;
          left: 30px;
          top: 25px;
        }

        .simi_con {
          flex: 1;
          overflow: hidden;
          margin-left: 10px;

          .simi_name {
            font-size: 14px;
            color: black;
            font-weight: 300;
            display: -webkit-box;
            -webkit-box-orient: vertical;
            -webkit-lint-clamp: 2;
            overflow: hidden;
          }

          .simi_singer {
            font-size: 12px;
            color: #52A2FF;
            margin-top: 5px;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
          }

          .simi_playcount {
            font-size: 10px;
            color: #999;
            margin-top: 4px;
          }
        }
      }
    }
  }
}

// 评论区
.mvComent {
  padding: 10px;

  ul {
    li {
      display: flex;
      width: 100%;
      margin-bottom: 15px;

      .mv_pl_pic {
        width: 50px;
        text-align: center;

        img {
          width: 35px;
          height: 35px;
          border-radius: 50%;
        }
      }

      .mv_pl_con {
        flex: 1;
        margin-left: 10px;

        .mv_pl_name {
          font-size: 14px;
          line-height: 35px;
          color: #6246F9;
        }

        .mv_pl_text {
          font-size: 13px;
          font-family: '幼圆';
          color: #666;
          line-height: 1.4;
        }

        .mv_pl_con_bottom {
          line-height: 30px;
          padding-right: 5px;
          color: #999;
          display: flex;
          justify-content: space-between;
          font-size: 12px;

          img {
            width: 14px;
            height: 14px;
            vertical-align: -3px;
            margin-right: 4px;
          }
        }
      }
    }
  }
  .pinglun_no{
    text-align center;
    margin-top:100px;
    font-size :16px;
    color:rgba(0,0,0,0.6);
  }
}
</style>
