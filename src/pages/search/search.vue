<template>
  <div class="searchBox">
    <div class="itpwrap">
      <div class="search_btn" :style="width">
        <icon type="search" size="15" class="search_icon" />
        <input
          type="text"
          placeholder="搜索音乐/视频/歌单/歌手"
          placeholder-style="color:#999;font-size:13px;"
          confirm-type="search"
          @focus="ipt"
          @confirm="search(value)"
          v-model="value"
        />
        <icon
          type="clear"
          size="17"
          color="#ccc"
          class="remove_icon"
          v-show="isSearch"
          @tap="clear"
        />
      </div>
      <p class="close" @tap="close">取消</p>
    </div>
    <!-- 热门搜索 -->
    <div class="search_bottom" v-show="!isSearch">
      <div class="hot_srarch">
        <h2 class="hot_h2">热门搜索</h2>
        <p>
          <span
            class="hot_con"
            v-for="(item,index) in hotArr"
            :key="index"
            @tap="hotSearch(item.first)"
          >{{item.first}}</span>
        </p>
      </div>
      <div class="hot_srarch" v-show="historyArr.length>0">
        <div class="removeBox">
          <h2 class="hist_h2">历史搜索</h2>
          <img src="/static/images/remove.png" @tap="remove" />
        </div>
        <p>
          <span
            class="hot_con"
            v-for="(item,index) in historyArr"
            :key="index"
            @tap="hotSearch(item)"
            v-show="hist_num>index"
          >{{item}}</span>
        </p>
        <p class="hist_more" v-if="historyArr.length>10" @tap="getMore">点击获取更多历史记录</p>
      </div>
    </div>
    <div class="search_list" v-show="isSearch">
      <ul>
        <li v-for="(item,index) in songList" :key="index" @tap="toPlay(item.id)">
          <p class="list_pic">
            <img src="/static/images/music.png" alt />
          </p>
          <div class="list_con">
            <div class="list_left">
              <p class="list_songName">{{item.name}}</p>
              <p class="list_singerName">
                <span v-for="(i,n) in item.artists " :key="n">{{i.name}}</span>
              </p>
            </div>
             <p v-if='item.mvid' class="songlist_con_right">
                 <image src='/static/images/mvplay.png' @tap.stop='toMvPlay(item.mvid)'></image>
            </p>
             <p class="songlist_con_right">               
                   <image src='/static/images/songbf.png'></image>                 
              </p>
          </div>
        </li>
      </ul>
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
      width: "width:90%",
      isSearch: false,
      value: "",
      hotArr: [],
      songList: [],
      historyArr: [],
      hist_num: 10
    };
  },
  watch: {},
  computed: {},
  methods: {
    // 聚焦时事件
    ipt() {
      this.width = "width:80%;";
      this.isSearch = true;
    },
    //  点击×事件
    clear() {
      this.value = "";
    },
    //点击取消事件
    close(e) {
      this.value = "";
      this.isSearch = false;
      this.width = "width:90%";
      this.songList = [];
      wx.hideLoading();
    },
    //点击搜索事件
    search(value) {
      wx.showLoading({
        title: "努力加载中"
      });
      wx.request({
        url: api.search + "?keywords=" + value+"&limit=50",
        success: res => {
          wx.hideLoading();
          // console.log(res);
          this.songList = res.data.result.songs;
        }
      });

      // 设置搜索记录
      var history = wx.getStorageSync("history")
        ? wx.getStorageSync("history")
        : [];

      if (!history.some(item => item === value)) {
        history.push(value);
      }
      wx.setStorageSync("history", history);
      this.historyArr = history;
    },
    //  点击热搜内容搜索事件
    hotSearch(item) {
      this.isSearch = true;
      this.width = "width:80%";
      this.search(item);
    },
    //  点击搜索的歌曲跳转事件
    toPlay(id) {
      var arr = JSON.stringify(this.songList);
      wx.navigateTo({
        url: "../play/main?id=" + id + "&list=" + encodeURIComponent(arr)
      });
    },
    // 点击删除历史记录
    remove() {
      console.log(1);
      this.historyArr = [];
      wx.clearStorageSync("history");
    },
    // 点击获取更多历史记录
    getMore() {
      this.hist_num = this.historyArr.length;
    },
    // 跳转播放mv界面
     toMvPlay(id){
       wx.navigateTo({ url: '../mvPlay/main?id='+id });
    },
  },
  mounted() {},
  onReady: function(e) {
    wx.request({
      //最新搜索
      url: api.hotSrarch,
      success: res => {
        //   console.log(res);
        this.hotArr = res.data.result.hots;
      }
    });

    var history = wx.getStorageSync("history")
      ? wx.getStorageSync("history")
      : [];
    this.historyArr = history;
  },
  onShow: function() {}
};
</script>
<style lang='stylus' scoped>
.itpwrap {
  width: 100%;
  height: 30px;
  margin: 10px 0;
  position: relative;

  .search_btn {
    padding: 0 30px;
    line-height: 30px;
    background: #f4f4f4;
    border-radius: 30px;
    position: absolute;
    box-sizing: border-box;
    left: 5%;
    z-index: 2;

    input {
      height: 30px;
      line-height: 30px;
      font-size: 14px;
    }

    .search_icon {
      position: absolute;
      left: 8px;
      top: 8px;
    }

    .remove_icon {
      position: absolute;
      right: 8px;
      top: 7px;
    }
  }

  .close {
    line-height: 30px;
    position: absolute;
    right: 5%;
    font-size: 13px;
    color: #666;
  }
}

.search_bottom {
  padding: 10px 5%;

  .hot_srarch {
    .hot_h2 {
      font-weight: bold;
      margin-bottom: 15px;
    }

    .removeBox {
      overflow: hidden;

      .hist_h2 {
        font-weight: bold;
        margin-bottom: 15px;
        float: left;
      }

      img {
        float: right;
        width: 20px;
        height: 20px;
      }
    }

    .hot_con {
      padding: 0 7px;
      height: 25px;
      line-height: 25px;
      color: black;
      font-size: 12px;
      display: inline-block;
      background: #f4f4f4;
      border-radius: 25px;
      margin: 0 10px 20px 0;
    }

    .hist_more {
      text-align: center;
      font-size: 14px;
      color: #666;
    }
  }
}

.search_list {
  padding: 10px 5%;

  li {
    display: flex;
    margin-bottom: 10px;

    .list_pic {
      width: 40px;
      height: 40px;
      text-align: center;

      img {
        width: 25px;
        height: 25px;
        margin-top: 7px;
      }
    }

    .list_con {
      flex: 1;
      line-height: 1.6;
      padding-top: 3px;
      margin-left: 5px;
      display :flex;
      overflow: hidden;
      .list_left{
        flex:1;
         .list_songName {
          width: 100%;
          font-size: 13px;
          color: black;
          white-space: nowrap;
          text-overflow: ellipsis;
          overflow: hidden;
        }

       .list_singerName {
          width: 100%;
          font-size: 10px;
          color: #333;
          white-space: nowrap;
          text-overflow: ellipsis;
          overflow: hidden;
       }
      }
    }
   .songlist_con_right{
       width:30px;
       padding-top:5px;
      text-align center;
      image{
         width:20px;
         height 20px;
         vertical-align :middle;
         display :inline-block;    
      }
    }
  }
}
</style>
