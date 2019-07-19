<template>
  <div class="morezj">
    <ul class="hotSong">
      <li v-for="item in special" :key="item.id">
        <image mode='widthFix' lazy-load="true" :src='item.blurPicUrl'></image>
        <p class="hot_name">{{item.name}}</p>
      </li>
    </ul>
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
      special: [],
      isShow:true,
    };
  },
  watch: {},
  computed: {},
  methods: {},
  mounted() {},
  onReady: function() {
    wx.showLoading({
      title: "努力加载中"
    });
    wx.request({
      //最新专辑
      url: api.newSpecial,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        this.isShow=false;
        this.special = res.data.data;
      }
    });
  }
};
</script>
<style lang='stylus' scoped>
.morezj {
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
.hotSong {
  margin-right: -7px;
  overflow: hidden;

  li {
    width: 113px;
    margin-right: 7px;
    margin-bottom: 7px;
    overflow: hidden;
    float: left;

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
  }
}
</style>
