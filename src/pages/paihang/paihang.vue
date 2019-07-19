<template>
  <div class="paihangBox">
    <ul>
      <li v-for="item in arrList" :key="item.id" @tap='tosonglist(item.id)'>
        <div class="list_pic">
          <img :src="item.coverImgUrl" />
        </div>
        <div class="list_con">
          <p class="list_name">{{item.name}}</p>
          <p class="list_text">{{item.description}}</p>
          <p class="playcount">
            <img src="/static/images/ting.png" alt />
            <span class="playnum">{{item.playCount}}万</span>
          </p>
        </div>
      </li>
    </ul>

  </div>
</template>

<script type="text/ecmascript-6">
import api from "../../utils/api";
export default {
  components: {},
  props: [],
  data() {
    return {
      arrList: [],
    };
  },
  watch: {},
  computed: {},
  methods: {
    tosonglist(id) {
      // console.log(id)
      wx.navigateTo({ url: "../songList/main?id=" + id });
    },

  },
  mounted() {},
  onReady: function() {
    wx.showLoading({
      title: "努力加载中"
    });
    wx.request({
      //最新专辑
      url: api.songPaihang,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        res.data.list.map(item => {
          return (item.playCount = (item.playCount / 10000).toFixed(1));
        });
        this.arrList = res.data.list;
      }
    });
  }
};
</script>
<style lang='stylus' scoped>
input{
  border:1px solid black;
}
.paihangBox {
  padding: 10px;

  ul {
    width: 100%;

    li {
      width: 100%;
      display: flex;
      background: #f4f4f4;
      box-shadow: 0 0 3px black;
      margin-bottom: 10px;
      overflow: hidden;

      .list_pic {
        width: 90px;
        height: 90px;

        img {
          width: 90px;
          height: 90px;
        }
      }

      .list_con {
        flex: 1;
        margin-left: 10px;
        padding-top: 5px;
        position relative;
        overflow: hidden;

        .list_name {
          font-size: 16px;
          font-weight: 600;
          color: #666;
        }

        .playcount {
          padding: 0 5px;
          position :absolute;
          right:5px;
          bottom:5px;
          margin-right: 5px;
          height: 20px;

          img {
            width: 18px;
            height: 18px;
            vertical-align: middle;
            margin-right: 5px;
          }

          .playnum {
            font-size: 10px;
            color: rgba(0, 0, 0, 0.6);
            vertical-align: middle;
          }
        }

        .list_text {
          padding-right: 5px;
          margin-top: 5px;
          font-size: 12px;
          color: rgba(0, 0, 0, 0.5);
          line-height: 1.5;
          display: -webkit-box;
          -webkit-box-orient: vertical;
          -webkit-line-clamp: 2;
          overflow: hidden;
        }
      }
    }
  }
}
</style>
