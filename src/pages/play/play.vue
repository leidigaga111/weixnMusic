<template>
  <div class="playBox" @tap="hideList">
       <!-- <button @click='aa' class="btn">开始</button> -->
      <!-- 背景样式 -->
    <div class="backg">
      <image :src='pic'  class="bgpic" mode='widthFix'></image>
      <p class="mark"></p>
      <image src='/static/images/zhen.png' class="zhenpic" :style="isrotate_zhen"></image>
      <p class="logo">
          <image src='/static/images/logo2.png' class="logopic" mode='widthFix'></image>
          <span>网易云音乐</span>
      </p>
    </div>
    <!-- 旋转的头像 -->
    <div class="liushengji" :style='isrotate'>
      <image src='/static/images/lsj.png' class="lsjpic"></image>
      <img :src="pic" alt="" class="singerPic">
      <image src='/static/images/bf2.png' class="pausePic" v-if='isPause'></image>
    </div>
    <!-- 歌词 -->
    <div class="lyricBox">      
        <p class="nameBox">
            <span class="song_name">{{name}}</span>
            <span class="singer_name" v-for='item in singer' :key='item.id'>{{item.name}} <span v-if='singer.length>1'>/</span> </span>
        </p>
        <div class="lyric">
            <div class="lyric_on" :style="lyc_top">
                <p v-for='(item,index) in lyrics' :key='index' :class="{'lyc_active':lycIndex==index}">{{item}}</p>
                <p v-if='lyrics.length==0'>暂无歌词</p>
            </div>
        </div>
    </div>
    <!-- 播放器 -->
    <div class="play_btn_box">
       <p class="jindu_box">
        <span>{{currentTime}}</span>
        <slider @change='speedChange' activeColor='#DD001B' backgroundColor='#EBEBEB' block-size="12" :value='speed'  class="jindu" />
        <span>{{duration}}</span>
       </p>
       <div class="btn_box">
         <image src='/static/images/random.png' class="play_random"></image>
         <div class="play_pause">
            <p class="prev"> <image src='/static/images/prev.png' @tap='prev'></image></p>
            <p class="play" v-if='isPause' @tap='change'><image src='/static/images/play1.png' class="pauseing"></image></p>
            <p class="play" v-if='!isPause'  @tap='change'><image src='/static/images/playing2.png' class="playing"></image></p>
            <p class="next"><image src='/static/images/next.png' @tap='next'></image></p>
         </div>
         <image src='/static/images/list.png' class="play_list" @tap.stop='list'></image>
       </div>  
       <!-- <div class="songList" v-if='isList' >
           <ul >
               <li v-for='(item,n) in songList' :key='item.id'>
                  <div class="list_left">
                      <p class="index" v-if='index!=n'>{{n+1}}</p>
                       <image src='/static/images/playing.gif' class="active" mode='widthFix' v-if='index==n'></image>
                  </div>
                  <p class="list_songName" @tap.stop='onPlay(n)'>{{item.name}}</p>
                  <p class="list_right">
                      <image src='/static/images/remove.png' class="remove" mode='widthFix' @tap.stop='remove(n)'></image>
                  </p>                  
               </li>
           </ul>
       </div> -->
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
      audio:'',
      duration:'00:00',//总时长
      currentTime:'00:00',//当前时长
      alltime:'',//没过滤的总时长
      selectTime:'',//没过滤的当前时长
      id: "",
      name: "",
      pic: "",
      singer: [],
      speed:0,
      isPause:true,
      isrotate:'animation-play-state :paused',
      isrotate_zhen:'transform: rotate(-25deg)',
      songList:[],
      index:'',
      isList:false,
      times:[],//歌词的时间数组
      lyrics:[],//歌词数组
      lycIndex:-1,//歌词的下标
      lyc_top:'top:0rpx',//歌词滚动的距离
    };
  },
  watch: {},
  computed: {},
  methods: {  
    // 显示列表页
    list(){
        this.isList=!this.isList;
    },
    hideList(){
        this.isList=false;
    },
      // 播放暂停状态事件   
      play(){
          if(!this.isPause){
            this.isrotate='animation-play-state :running';
            this.isrotate_zhen='transform: rotate(0deg)';
          }else{
            this.isrotate='animation-play-state :paused',
            this.isrotate_zhen='transform: rotate(-25deg)'
          }      
      },
    // 播放暂停事件  
      change(){    
          if(this.isPause){ 
              this.isPause=false;  
              this.play()
              this.audio.play()
            
          }else{
              this.isPause=true;
              this.play()
               this.audio.pause()
          }     
      },
    //  下一首
      next(){       
        var n=this.index;
        if(n==this.songList.length-1){
            n=-1;
        }
        n+=1;
        this.index=n;
        this.id=this.songList[this.index].id;   
        this.play_info();
        this.lyric_info();  
        this.play_url();    
        this.audioPlay() ;     
      },
    //   上一首
    prev(){
        // this.lyc_top="top:0px";
        // this.lyrics=[];
        // this.times=[];
        // this.name='';
        // this.singer=[];
        // this.lycIndex=-1;
      var n=this.index;
        if(n==0){
            n=this.songList.length;
        }
        n-=1;
        this.index=n;
        this.id=this.songList[this.index].id;      
        this.play_info();
        this.lyric_info();  
        this.play_url();
        this.audioPlay();
    
    },
    // 点击列表播放
    onPlay(n){      
        if(this.index==n){
            return
        }
         this.index=n;
         this.id=this.songList[this.index].id;            
         this.audioPlay();
         this.play_info();
         this.lyric_info();
         this.isList=false;
    },
    // 删除列表中的歌曲
    remove(n){
       this.songList.splice(n,1)
    },
    // 进度条拖动事件
      speedChange(e){       
          var value=e.target.value;
          this.speed=value;
          var alltime=this.alltime;
          var selectTime=this.selectTime;
          var currentTime=value*alltime/100;
           this.play_url();
          this.audioPlay();
          this.play()
          this.audio.seek(currentTime);
          this.audio.play();
          this.lyricSelect(this.time(selectTime));
 

    },
    //   处理时间函数
    time(time){
        var mime=Math.floor(time/60)<10?"0"+Math.floor(time/60):Math.floor(time/60);
        var second=Math.floor(time%60)<10?"0"+Math.floor(time%60):Math.floor(time%60);
        return mime+":"+second
    },
    // 歌词同步
    lyricSelect(time){
        // var n=this.lycIndex;
        // if(time>this.times[0]){
        //     n++
        //     this.times.shift()
        // }
        // this.lycIndex=n;
        // if(this.lycIndex<2){
        //     return
        // }
        
        var num=0; 
        for(var i =0;i<this.times.length;i++){
          if(this.times[i]==time){
            num=i;
            this.lycIndex=num;
            break;
          }
        }
        if(num<2){
          return 
        }
        // 28.3
        var y=-(num-2)*30;                   
        this.lyc_top=`top:${y}px`; 
        //  console.log(this.lyc_top);               
    },
    // 请求歌曲信息
    play_info(){  
     
      wx.request({
      //歌曲详情
      url: api.playInfo + "?ids=" + this.id,
      success: res => {
        // console.log(res);
          this.name=res.data.songs[0].name;
          this.pic=res.data.songs[0].al.picUrl;
          this.singer=res.data.songs[0].ar;
      }
    });
    },
    // 请求歌词信息
    lyric_info(){ 
      this.times=[];
      this.lyrics=[];
     wx.request({
      //歌曲详情
      url: api.lyric + "?id=" + this.id,
      success: res => {  
        // console.log(res);
        if(!res.data.lrc){
          return;
        }
        var lyric=res.data.lrc.lyric; 
        // // 先用换行拆分为数组
        var arr=lyric.split('\n');          
        var reg=/\[(\d*:\d*\.\d*)\]/;
        arr.map(item=>{
            // 下面是处理歌词
            var lyc=item.split(']')[1];  
            if(lyc==''||lyc==undefined){
              return true;
            }                
            this.lyrics.push(lyc)
            
            // 时间处理
            var res=reg.exec(item)         
            if(res==null){
               return true;
            }                 
            var timeStr=res[1];//00:00.92   
            var t1=timeStr.split('.')         
                               
            // var res2=timeStr.split(':');
            // var min=parseInt(res2[0])*60;
            // var sec=parseFloat(res2[1]);
            // var time=Number((min+sec).toFixed(3));
            // this.times.push(time) 
            this.times.push(t1[0]);       
        })
        //  console.log(this.times);
        //  console.log(this.lyrics);    
       }
     });
    },
     // 请求播放url
    play_url(){
      wx.request({
        url: api.playUrl+"?id="+this.id+'&br=320000', //开发者服务器接口地址",
        success: res => {
          // console.log(res); 
          this.audio.src=res.data.data[0].url;   
        },           
      });
    },
       // 创建播放器并进行播放
    audioPlay(){
        this.lyc_top="top:0px";
        this.lyrics=[];
        this.times=[];
        this.name='';
        this.singer=[];
        this.url='';
        this.lycIndex=-1;    
           
        if(this.audio){
          this.audio.destroy();
        } 
        const audio = wx.createInnerAudioContext()
        this.audio=audio;
        // this.url='https://v1.itooi.cn/netease/url?id='+this.id+'&quality=128'       
        this.audio.autoplay=true;
           
        // 监听音乐更新进度
        this.audio.onTimeUpdate(()=>{
           wx.hideLoading();
          var allTime=this.audio.duration;
          this.alltime=allTime;
          var currentTime=this.audio.currentTime
          this.selectTime=currentTime;
          this.duration=this.time(allTime)
          this.currentTime=this.time(currentTime)
          this.speed=(currentTime/allTime)*100;
          // console.log(this.currentTime);        
          this.lyricSelect(this.currentTime) 
             
       });
           // 监听音乐播放事件
        this.audio.onPlay(()=>{
            wx.hideLoading();
            // 判断是否为播放状态
            this.isPause=false;
            this.play()
        })
        // 监听音乐暂停事件
        this.audio.onPause(()=>{
            this.isPause=true;
            this.play();
        })        
        // 监听音乐播放完成事件
        this.audio.onEnded(()=>{
            this.next();
        })
    },
 },

  // 页面加载之前
  onLoad: function(options) {  
    this.id = options.id; 
    var list=decodeURIComponent(options.list);
    this.songList=JSON.parse(list);
    var index =this.songList.findIndex(item=>{
        return item.id==this.id;
    })
    this.index=index;
   
  },

  // 页面加载完成之后
  onReady: function () {
     wx.showLoading({
      title: "努力加载中"
    });
  },
//   监听页面显示
  onShow: function () {
    this.play_info();
   this.lyric_info();
   this.play_url(); 
   this.audioPlay();
    
  },
  //   页面卸载之前
  onUnload: function () {
    this.name='';
    this.pic='';
    this.singer=[];
    this.times=[];
    this.lyrics=[];
    // this.audio.destroy();
    console.log("页面卸载了");
    
  },
};
</script>
<style lang='stylus' scoped>
.playBox {
  widht: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
  .btn{
     width:70px;
     height:30px;
     line-height 30px;
     font-size :16px;
     position :absolute;
     left:0;
     top:50px;
     z-index 1;
  }
  .backg {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    .bgpic {
      width: 120%;
      position: absolute;
      left: -20%;
      top: 0;
      transform: scale(1.8);
      filter: blur(10px);
    }

    .mark {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      background: rgba(0, 0, 0, 0.4);
    }

    .zhenpic {
      width: 80px;
      height: 110px;
      position: absolute;
      z-index: 20;
      left: 45%;
      top: 0;
      transform-origin: 0 0;
      transition :all 0.3s;
    }
    .logo{
         position :absolute;
         left:0;
         top:10px;
        .logopic{
            width:30px;
            vertical-align :middle;
        }
        span{
            font-size :16px;
            color:white;
            line-height 30px;
            display :inline-block;
            vertical-align :2px;
            margin-left:5px;

        }
    }
  }

  .liushengji {
    width: 220px;
    height: 220px;
    border-radius: 50%;
    margin: 60px auto 20px;
    box-shadow: 0 0 10px black;
    position: relative;
    animation:move 5s linear infinite;
    transition :all 0.3s;
    z-index :5;
    .lsjpic {
      width: 106%;
      height: 106%;
      border-radius: 50%;
      position: absolute;
      left: -6px;
      top: -6px;
    }

    .singerPic {
      width: 63%;
      height: 63%;
      border-radius: 50%;
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
    }
    .pausePic{
        width:18%;
        height:18%;
        border-radius: 50%;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
    }
  }
  .lyricBox{
      height:215px;
      position relative;
      z-index 1;
      padding:0 15px;
      text-align center;
      overflow hidden;
      .nameBox{
          overflow hidden;
          .song_name{
              font-size 18px;
              color:#fefefe;
              display :block;
              overflow hidden;
              white-space :nowrap;
              text-overflow :ellipsis;             
          }
          .singer_name{
              display :inline-block;
              font-size :13px;
              color:rgba(255,255,255,0.6);
              overflow hidden;
              white-space :nowrap;
              text-overflow :ellipsis;
              margin-top:5px;
              margin-bottom:5px;
          }
      }
      .lyric{
          height:145px;     
          overflow auto;
          position relative;
          .lyric_on{
              width:100%;            
              text-align center;
              position :absolute;
              left:0;
              transition :all 0.3s;
              overflow hidden;
             p{
                 color:rgba(255,255,255,0.7);
                 font-size :14px;
                 height 30px;
                 line-height 30px;
                 white-space nowrap;
                 text-overflow :ellipsis;
                 overflow hidden;
             }
             .lyc_active{
                //  font-size 16rpx;
                //  color:orange;
                    background   : rgba(11, 12, 12, 0.5);
                    height 30px;
                    line-height 30px;
                    font-size    : 15px;
                    color        : white;
                    transition   : all 0.5s;
                    border-radius: 10px;
                    animation    : move2 0.5s linear forwards;
            }
         }
       
      }
  }
  .play_btn_box{
      width:90%;
      height:90px;
      position :absolute;
      left:5%;
      bottom :0;
      z-index 1;
      box-sizing :border-box;
      .jindu_box{
        position :absolute;
        left:15px;
        top:0;
        width:100%;
        .jindu{
          width:65%;
          display :inline-block;
          vertical-align :middle;
          margin:0 13px;
        }
        span{
            font-size :12px;
            display :inline-block;
            color:white;
            vertical-align :middle;
        }
      }  
      .btn_box{
          width:100%;
          display :flex;
          justify-content :space-between;
          position :absolute;
          left:0;
          top:27px;
          padding:0 10px;
          box-sizing :border-box;
          .play_random{
              width:25px;
              height:25px;
              margin-top:12px;
          }
          .play_list{
              margin-top:14px;
               width:20px;
              height:20px;
          }
          .play_pause{
              flex:1;
              display :flex;
              justify-content :space-around;
              margin:0 20px;
              p{
                  border-radius :50%;
                  background:#DD001B;
                  text-align center; 
                  padding-top:7px;
                  box-sizing :border-box; 
              }
              .prev,.next{
                  margin-top:7px;
                  width:35px;
                  height:35px;        
                    
                  image{
                      width:20px;
                      height 20px;
                  }
              }
              .play{
                  width:50px;
                  height:50px;
                  position :absolute;
                  text-align center;

                  .playing{
                      width:30px;
                      height 30px
     
                      position :absolute;
                      left:50%;
                      top:50%;
                      transform :translate(-50%,-50%)
                  }
                  .pauseing{
                     width:30px;
                      height 30px
                      margin-left:4px;
                      margin-top:1px;
                  }
              }
          }
      }
      // .songList{
      //     width:220px;
      //     height:300px;
      //     background:#f4f4f4;
      //     position :absolute;
      //     right:0;
      //     bottom:50px;
      //     border-radius :10px;
      //     z-index 100;
      //     overflow auto;
      //     transition :all 0.3s;        
      //     ul{
      //         width:100%;
      //         padding:10px;
      //         position :absolute;
      //         left:0;
      //         top:0;
      //         box-sizing :border-box;
      //         z-index 30;
      //         li{
      //             display:flex;
      //             height :30px;
      //             padding-bottom:5px;
      //             box-shadow :0 2px 5px black;
      //             margin-bottom:5px;
      //             .list_left {
      //                 width:30px;
      //                 text-align center;
      //                  line-height 30px;
      //               .index{                    
      //                 font-size :12px;
      //                 color:#888;                 
      //              }
      //             .active{
      //                 display :inline-block;
      //                 vertical-align -3px;
      //                 width:20px;
      //             }
      //           }
      //           .list_songName{
      //               flex:1;
      //               margin:0 5px;
      //               line-height 30px;
      //               font-size :14px
      //               color:black;
      //               white-space nowrap;
      //               text-overflow :ellipsis;
      //               overflow hidden;
      //           }
      //           .list_right{
      //               width:30px;
      //               text-align center;
      //               padding-top:3px;
      //               .remove{
      //                   display :inline-block;
      //                   width:20px;

      //               }
      //           }

      //         }
      //     }
      // }
  }
}
vendors = official;
@keyframes move {
  100% {
    transform: rotate(360deg);
  }
}
@keyframes move2 {
    0% {
        transform: rotateX(180deg);
    }

    100% {
        transform: rotateX(0deg);
    }
}
</style>
