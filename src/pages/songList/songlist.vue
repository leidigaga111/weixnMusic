<template>
  <div class="songListBox">
    <div class="songlist_top">
      <p class="mark"></p>
      <image mode='widthFix' :src='pic' class="bgPic"></image>
      <div class="top_conBox">
        <div class="top_left">
          <image mode='widthFix' :src='pic' class="tlt_pic"></image>
          <p class="play_count">
              <image mode='widthFix' src='/static/images/ting2.png' class="tlt_pic"></image>
              <span>{{playcount}}万</span>
          </p>
        </div>
        <div class="top_right">
          <p class="gd_name">{{name}}</p>
          <p class="gd_type">
            <span v-for="(item,index) in type" :key='index' v-show='3>index'>{{item}}</span>
            <!-- <span>老歌</span> -->
          </p>
          <p class="gd_order">
            <image class="gdOrder_pic" :src='singer.backgroundUrl'></image>
            <span>{{singer.nickname }}</span>
          </p>
        </div>
      </div>
      <div class="play_all">
        <div class="playAll_left">
           <image src='/static/images/songbf.png'></image>
           <span class="playAll_btn">全部播放</span>
           <span class="play_num">/{{songList.length}}首</span>
        </div>
        <div class="playAll_right">
           <image src='/static/images/xh.png'></image>
            <span class="play_sx">循环播放</span>
        </div>
      </div>
    </div>
    <div class="songList">
        <ul>
            <li v-for='(item,n) in songList' :key="item.id" @tap='toPlay(item.id)'>
                <p class="index">{{n+1}}</p>
                <div class="songlist_con">
                    <div class="songlist_con_left">
                        <p class="play_name" >{{item.name}}</p>
                        <p class="play_singer">
                            <span v-for='i in item.singerArr' :key='i.id'>{{i.name}}/</span>
                        </p>
                    </div>
                    <p v-if='item.mv' class="songlist_con_right">
                      <image src='/static/images/mvplay.png' @tap.stop='toMvPlay(item.mv)'></image>
                    </p>
                    <p class="songlist_con_right">               
                        <image src='/static/images/songbf.png'></image>                 
                    </p>
                </div>
            </li>
        </ul>
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
      isShow:true,
      songList: [],
      type:[],
      name:'',
      playcount:'',
      singer:'',
      pic:''
    };
  },
  watch: {},
  computed: {},
  methods: {
    toPlay(id){
      var arr=JSON.stringify(this.songList)
      wx.navigateTo({ url: '../play/main?id='+id +'&list='+encodeURIComponent(arr)});
    },
     toMvPlay(id){
       wx.navigateTo({ url: '../mvPlay/main?id='+id });
    },
  },
  mounted() {},
  onLoad: function(options) {
    wx.showLoading({
      title: "使出了吃奶劲~~"
    });
    var id = options.id;
    wx.request({
      url: api.songList + "?id=" + id,
      success: res => {
        // console.log(res);
        wx.hideLoading();
        this.isShow=false;
        if(res.data.playlist.tracks.length>900){
            var arr=res.data.playlist.tracks.splice(0,899)
        }else{
          var arr=res.data.playlist.tracks
        }
        
        var obj={};
        var arr2=[];
        arr.map(item=>{
            obj={
                id:item.id,
                name:item.name,
                singerArr:item.ar,
                mv:item.mv
            }
            arr2.push(obj)
        })
        // console.log(arr2);     
        this.songList = arr2 ;//歌曲列表
        this.type=res.data.playlist.tags ;//类型
        this.name=res.data.playlist.name ;//歌单名字
        this.playcount= (res.data.playlist.playCount/ 10000).toFixed(1) ;//播放数量
        this.singer=res.data.playlist.creator;//作者
        this.pic=res.data.playlist.coverImgUrl;
      }
    });
 
    
  },
   onUnload: function () {
    this.songList=[];
    this.isShow=true;
  },
};
</script>
<style lang='stylus' scoped>
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
.songlist_top {
  width: 100%;
  height: 185px;
  padding: 15px 22px 35px 18px;
  box-sizing: border-box;
  overflow: hidden;
  position: fixed;
  left:0;
  top:0;
  z-index 10;

  .mark {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    background: rgba(0, 0, 0, 0.5);
  }

  .bgPic {
    width: 100%;
    position: absolute;
    left: 0;
    top: -90px;
    z-index: -1;
  }
  .play_all{
      width:100%;
      height:35px;
      background :white;
      position :absolute;
      border-radius:10px 10px 0 0/10px 10px 0 0;
      left:0;
      bottom:0;
      display :flex;
      justify-content :space-between;
      padding:0 15px;
      box-sizing :border-box;
      .playAll_left{
          line-height 35px;
          image{
              display :inline-block;
              width:22px;
              height 22px;
              border-radius 50%;
              vertical-align :middle;
          }
          .playAll_btn{
              font-size 14px;
              color:black;
              display :inline-block;
              margin:0 3px 0 7px;
          }
          .play_num{
              font-size :10px;
              color:#999;
          }
      }
      .playAll_right{
          line-height 35px;
          image{
              width:18px;
              height :18px;
              border-radius :50%;
              display :inline-block;
              vertical-align :middle;
              margin-right :4px;
          }
          .play_sx{
              font-size :12px;
              color:#666;
          }

      }
  }

  .top_conBox {
    width: 100%;
    height: 120px;
    display: flex;
    position: relative;
    // border:1px solid white; 

    .top_left {
      width: 120px;
      margin-right: 15px;
      position: relative;

      .tlt_pic {
        width: 100%;
        border-radius: 10px;  
      }
      .play_count{
          padding:0 2px;
          height 15px;
          border-radius :3px;
          background :rgba(0,0,0,0.2);
          position :absolute;
          left:0;
          bottom:5px;
          color:rgba(255,255,255,0.6);
          image{
              width:12px;
              height 12px;
              vertical-align :2px;
              margin-right :2px;
          }
          span{
              display :inline-block;
              font-size 9px;
              vertical-align :5px;
          }
      }
    }

    .top_right {
      flex: 1;
      color: white;
      position: relative;
      padding-top: 7px;

      .gd_name {
        height: 45px;
        font-size: 16px;
        line-height: 1.4;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        overflow: hidden;
      }

      .gd_type {
        margin-top: 7px;

        span {
          display: inline-block;
          height: 20px;
          line-height: 20px;
          font-size: 12px;
          text-align: center;
          padding: 0 5px;
          margin-right: 10px;
          border-radius: 5px;
          background: #999;
        }
      }

      .gd_order {
        margin-top: 10px;
        height: 25px;

        .gdOrder_pic {
          width: 25px;
          height: 25px;
          border-radius: 50%;
          display: inline-block;
          vertical-align: middle;
          margin-right: 10px;
        }

        span {
          display: inline-block;
          line-height: 25px;
          color: rgba(255, 255, 255, 0.7);
          font-size: 14px;
          white-space: nowrap;
          text-overflow: ellipsis;
          overflow: hidden;
          vertical-align: middle;
        }
      }
    }
  }
}
.songList{
    margin-top:188px;
    width :100%;
    background:white;
    overflow hidden;
    padding:0 12px;
   box-sizing :border-box;
    ul{
       width:100%;
       padding-bottom:10px;
       border-top:1px solid #ccc;
       li{  
           width:100%;
           display :flex;
           overflow hidden;                  
           .index{
               width:40px;
               line-height 60px;
               text-align center;
               font-size :14px;
               color:#888;
           }
           .songlist_con{
               flex:1;
               height:60px;
               border-bottom :1px solid #ccc;
               display :flex;
               justify-content :space-between;
               overflow hidden;
               .songlist_con_left{
                   flex:1;
                   overflow hidden;
                   .play_name{
                       width:100%;
                       margin-top:8px;
                       height:25px;
                       line-height 25px;
                       font-size :14px;
                       color:black;
                       white-space :nowrap;
                       text-overflow :ellipsis;
                       overflow hidden;
                   }
                   .play_singer{
                       font-size :12px;
                       color:#999;
                       white-space :nowrap;
                       text-overflow :ellipsis;
                       overflow hidden;
                   }
               }
               .songlist_con_right{
                   width:30px;
                   padding-top:15px;
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
    }
}
</style>
