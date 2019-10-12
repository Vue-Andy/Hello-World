<template>
  <div class="ser_home">
    <!-- <div class="title"><span class='leftTime'>{{leftTime}}s</span></div> -->
    <div class="title"><span class="box"></span>x<span class='text'>{{realBoxNum ? realBoxNum : 0}}</span><span v-if='leftTime>0' class='leftTime'>{{leftTime}}s</span></div>
    <ul class="red_packet" id="red_packet">
      <template v-if='!isStarted && !isEnded'>
        <div class='tips' v-if='seconds >= 120'>活动还未开始</div>
        <div class="tips" v-if='seconds >0 && seconds < 120'>
          <div>距离活动开始还有</div>
          <div class='timeleft'>{{seconds}}s</div>
        </div>
        <!-- <div class="tips">加载中...</div> -->
      </template>
      <template v-for="(item, index) in liParams" v-show='isStarted && !isEnded'>
        <li :key="item.keyId" :style="{ left: item.left, animationDuration: item.durTime, webkitAnimationDuration: item.durTime}" 
            :class="item.cls" :data-index="index" @webkitAnimationEnd="removeDom" @touchstart='handleClick(item.couponId,$event)'>
          <div>
            <i class='redenvelope' :style="{ transform: item.transforms, webkitTransform: item.transforms}"></i>
          </div>
        </li>
      </template>
      <template v-if='isEnded'>
        <div v-if='isStarted && isEnded' class="tips" @click='checkResult'>
          <img src="./image/tips1.png" alt="">
          <div class="text">抢到了{{realBoxNum}}个红包</div>
          <button class="checkResult">查看奖品</button>
        </div>
        <div v-if='!isStarted && isEnded' class="tips" @click='scrollAround'>
          <img src="./image/tips2.png" alt="">
          <button class="checkResult">随便逛逛</button>
        </div>
        <!-- <div class="close">
          <img src="./image/close.png" alt="">
        </div> -->
      </template>
    </ul>
    <div class='count' :class='{countStart:isStarted && !isEnded}'><div>X</div><div>{{realBoxNum ? realBoxNum : 0}}</div></div>
    <div class="mask" v-if='isEnded'></div>
  </div>
</template>
 
 
<script>
// https://www.jb51.net/article/143585.htm  -- 参考资料
// import  navgate  from "~/static/js/wx-mehtod.js";
export default {
  head() {
    return {
      title: "机械之家"
    };
  },
  data () {
    return {
      // 下落的红包列表
      liParams: [],
      // 请求来的优惠券ID列表
      couponIds:[1,2,3,4,5,6,7,8,9],
      // 优惠券列表长度
      couponLength:0,
      timer: null,
      // 活动持续时间
      duration: 0, // 定义红包雨持续时间
      // 活动是否已开始
      isStarted:false,
      // 活动是否已结束
      isEnded:false,
      // 活动开始时间
      startTime:1570871040000,
      // 活动结束时间
      endTime:1570871100000,
      // 判断首次是否可以开始的定时器
      firstTimer:null,
      // 距离活动开始的时间
      seconds:120,
      // 客户抢到的有效红包数量
      realBoxNum:0,
      // 剩余时间
      leftTime:0
    }
  },
  created() {
    // this.$axios.setHeader("identifyToken", this.$route.query.token);
    // this.$axios.setHeader("identifyToken", "fb0ef292ebf011e9b58300163e0c213a");
    // 一进来就先获取一次优惠券数量，防止用户退出后看不到即时数量
    // this.getCouponCount()
    // 创建完毕调取接口获取开始时间和结束时间
    this.getTime()
    // this.getCouponList()
    // 如果未开始或者已经结束了，就启动定时器，判断是否要启动活动
    if(!this.isStarted || this.isEnded){
      setInterval(()=>{
        if(new Date().getTime() >= this.endTime){
          setTimeout(()=>{
            this.isEnded = true
            // 防止中途进来的用户也能抢一分钟的红包，导致结束页已出来，红包还在下
            // this.duration = 0
          },3000)
        }else if(new Date().getTime() >= this.startTime){
          this.isStarted = true
        }else{
          this.seconds = Math.ceil((this.startTime - new Date().getTime())/1000)
        }
        // 活动一开始，便计算活动剩余时间
        if(this.isStarted && !this.isEnded){
          this.leftTime = parseInt((this.endTime - new Date().getTime())/1000)
        }
      },1000)
    }
  },
  mounted () {
    /* setInterval(()=>{
      this.startRedPacket()
    },100) */
    this.firstTimer = setInterval(()=>{
      if(this.isStarted){
        this.startRedPacket()
        clearInterval(this.firstTimer)
        this.firstTimer = null
      }
    },1000)
    // 阻止屏幕拖动事件
    document.querySelector('.ser_home').addEventListener('touchmove', function(e) {
      // console.log(e.srcElement.className)
      // let tar = e.srcElement ? e.srcElement : e.target
      /* if(tar.className == 'redenvelope'){
        console.log(tar.className)
        return
      }else {
        e.preventDefault();
      } */
      e.preventDefault();
    },false);
  },
  methods: {
    /*** 开启动画 ***/
    startRedPacket() {
      let win = document.documentElement.clientWidth || document.body.clientWidth
      let left = parseInt(Math.random() * (win - 50) + 0);
      let rotate = (parseInt(Math.random() * (45 - (-45)) - 45)) + "deg"; // 旋转角度
      let scales = (Math.random() * (12 - 8 + 1) + 8) * 0.1; // 图片尺寸
      let durTime = (Math.random() * (2.5 - 1.2 + 1) + 1.2) + 's'; // 时间 1.2和1.2这个数值保持一样--此处控制下落时间
      let keyId = (Math.random()*100).toFixed(4) + "_id"
      let num = Math.random();
      let couponId = Math.floor(Math.random()*this.couponLength)  // 此处随机生成一个 介于 0 - this.couponLength 的数，用于取优惠券ID
      if(num > 0.4){  // 此处控制红包的真是概率，5个里面一个真的
        this.liParams.push({left: left+'px', cls: 'move_1', transforms: 'rotate('+ rotate +') scale('+ scales +')', durTime: durTime,keyId:keyId,couponId: null})
      }else{
        this.liParams.push({left: left+'px', cls: 'move_1', transforms: 'rotate('+ rotate +') scale('+ scales +')', durTime: durTime,keyId:keyId,couponId: this.couponIds[couponId]})
      }
      setTimeout( () => {  // 多少时间结束
        clearTimeout(this.timer)
        return;
      }, this.duration)
 
      this.timer = setTimeout( () => {
        this.startRedPacket()
      },100)  
    },
    /** 回收dom节点 **/
    removeDom (e) {
      let target = e.currentTarget;
      document.querySelector('#red_packet').removeChild(target)
    },
    // 获取开始时间和结束时间
    getTime() {
      /* this.$axios.get("/core/mina/redenvelope/getActivityTime/1").then(res => {
        console.log(res);
        let stTime = res.data.startTime, enTime = res.data.endTime;
        this.startTime = res.data.startTime
        this.endTime = res.data.endTime
        this.seconds = Math.round(res.data.timeInterval/1000)
        setInterval(()=>{
          this.seconds--
        },1000)
        var now = new Date().getTime()
        // 用户进来的时候，如果已经开始了，则持续时间截止到结束时间，如果未开始，则持续时间为结束时间-开始时间
        this.duration = this.startTime > now ? Math.round((this.endTime - this.startTime)/1000)*1000 : Math.round((this.endTime - now)/1000)*1000
      }); */
      var now = new Date().getTime()
      this.duration = this.startTime > now ? Math.round((this.endTime - this.startTime)/1000)*1000 : Math.round((this.endTime - now)/1000)*1000

    },
    // 获取优惠券列表
    getCouponList() {
      this.$axios.post("/core/mina/redenvelope/getCouponList").then(res => {
        this.couponIds = res.data.data.couponIds
        this.couponLength = this.couponIds.length
      });
    },
    // 获取用户已领取的优惠券数量
    getCouponCount() {
      this.$axios.post('/core/mina/redenvelope/queryReceivedRedEnvelopCount',{}).then(res=>{
        console.log('getCouponCount',res)
        this.realBoxNum = res.data.data
      })
    },
    handleClick(id,e) {
      console.log(id)
      // 点击后的红包出现不同的背景区分，不管是否是有效的红包
      e.currentTarget.className = 'move_1 active'
      if(id){
        // this.$axios.post("/core/mina/redenvelope/sendRedEnvelop",{couponId:id}).then(res => {
        //   console.log(res)
          // this.getCouponCount()
          this.realBoxNum ++ 
        // });
      }
    },
    checkResult() {
      navgate.mpNavigateTo('/pages/coupon/coupon')
    },
    scrollAround() {
      navgate.mpSwitchTab('/pages/market/index')
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
html,body{
  width: 100%;
  height:100%;
}
.title{
  color:#fff;
  position:fixed;
  top:0;
  left:0;
  padding:10px;
  width:100%;
}
.title > span.box{
  display: inline-block;
  width:25px;
  height:38px;
  background-image: url(./image/redbox.png);
  background-size:contain;
  margin-right:10px;
  margin-bottom:-6px;
}
.title > span{
  color:#fff;
  font-weight: bold;
  font-size: 30px;
  line-height: 30px;
  vertical-align: baseline;
}
ul{padding:0;}
.ser_home {
  width: 100%;
  height: 100%;
  position:absolute;
  top:0;
  left:0;
  bottom:0;
  right:0;
  background-image:url('./image/bgImg.png');
  background-repeat: no-repeat;
  background-size: cover;
  overflow:hidden;
}
.tips{
  width:100%;
  color:#fff;
  font-size:1rem;
  text-align:center;
  margin-top:50%;
  position:relative;
  z-index: 200;
}
.tips > img{
  width:100%;
}
.tips > .text{
  width:15em;
  font-size: 1.5rem;
  position:absolute;
  left:50%;
  transform: translateX(-50%);
  bottom:5.2em;
}
.tips > button{
  display: block;
  width:8.4em;
  height:2.2em;
  line-height: 2.2em;
  font-size: 1.5rem;
  background:#FFF100;
  color:#323131;
  border-radius: 1.1em;
  outline: none;
  border:none;
  position:absolute;
  bottom:2em;
  left:50%;  
  transform: translateX(-50%)
}
.move_1 i{
  width: 48px;
  height: 69px;
  display: block;
  background: url('./image/redbox.png') no-repeat;
  background-size: contain;
}
.move_1.active i{
  background: url('./image/open.png') no-repeat;
  background-size: contain;
  transform:scale(2.0) !important;
}
.red_packet li{
  list-style: none;
  position: absolute;
  animation: all 3s linear;
  top:-100px;
  z-index: 10;
}
.red_packet li.move_1{
  -webkit-animation: aim_move 5s linear 1 forwards;
  animation: aim_move 5s linear 1 forwards;
}
.red_packet > li > div{
  display: block;
  border:none;
  outline:none;
}
@keyframes aim_move {
  0% {
  -webkit-transform: translateY(0);
  transform: translateY(0);
  }
  100% {
  -webkit-transform: translateY(120vh);
  transform: translateY(120vh);
  }
}
.close{
  font-size: 1rem;
  width:3em;
  height:3em;
  text-align: center;
  margin:0 auto;
}
.close > img{
  width:100%;
}
.timeleft{
  margin-top:20%;
  font-size: 50px;
  animation: mymove 1s infinite;
  -webkit-animation: mymove 1s infinite;
  -o-animation: mymove 1s infinite;
}
@keyframes mymove{
  0% {transform: scale(1);}
  50% {transform: scale(2);}
  100% {transform: scale(1);}
}
div.count{
  width:100%;
  text-align: center;
  color:#fff;
  position:absolute;
  bottom:80px;
  margin-top:500px;
}
div.count.countStart{
  animation:mymove 1s infinite;
  -webkit-animation:mymove 1s infinite;
  -o-animation: mymove 1s infinite;
}
div.count > div{
  font-size: 40px;
  display: inline-block;
  margin-right:5px;
}
div.count > div:first-child{
  font-size: 20px;
  padding-bottom:20px;
  transform:translateY(-15%)
}
.title > .leftTime{
  display:block;
  width:80px;
  font-size:40px;
  line-height: 38px;
  text-align: center;
  border-radius: 20px;
  background:rgba(217, 91, 223, 0.5);
  margin:0 auto;
}
div.mask{
  width:100%;
  height:100%;
  position: absolute;
  top:0;
  left:0;
  bottom:0;
  background:#333;
  opacity: .5;
  z-index:100;
}
</style>