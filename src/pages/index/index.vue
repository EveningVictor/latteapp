<template>
  <div class="wrapper">
    <div class="header-wrapper">
      <div class="header-title">
       <span>空气质量-{{airText}}</span>
       <span>{{area}}-{{city}}</span>
      </div>
      <div class="header-text">
        <span>{{airValue}}</span>
        <span>{{weather}}</span>
      </div>
      <div class="weather-advice">{{weatherAdvice}}</div>
    </div>
    <div class="body-wrapper">
      <div class="body">
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/wendu.png"/>
            <div class="data-text">
              <div class="data-title">温度</div>
              <div class="data-value">{{Temp}}℃</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/shidu.png"/>
            <div class="data-text">
              <div class="data-title">湿度</div>
              <div class="data-value">{{Hum}}%</div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/guangzhaodu.png"/>
            <div class="data-text">
              <div class="data-title">光照度</div>
              <div class="data-value">{{Light}}Lx</div>
            </div>
          </div>
          <div class="data">
            <img class="data-logo" src="/static/images/led.png"/>
            <div class="data-text">
              <div class="data-title">客厅灯</div>
              <div class="data-value">
                <switch @change="onLedChange" :checked="Led" color="#3d7ef9"/>
              </div>
            </div>
          </div>
        </div>
        <div class="data-wrapper">
          <div class="data">
            <img class="data-logo" src="/static/images/beep.png"/>
            <div class="data-text">
              <div class="data-title">报警器</div>
              <div class="data-value">
                <switch @change="onBeepChange" :checked="Beep" color="#3d7ef9"/>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    

  </div>
</template>

<script>
//import{ connect } from "mqtt";
import { connect } from 'mqtt/dist/mqtt.js'; 

const mqttUrl ='wxs://https://api.heclouds.com/devices/643509630/datastreams/dht01'; //此处填写服务器url
const wet_url ='wxs://https://api.heclouds.com/devices/643509630/datastreams/dht01';
const tem_url ='wxs://https://api.heclouds.com/devices/643509630/datastreams/dht01';



export default {
  data () {
    return {
      client:{},
      Temp:0,
      Hum:0,
      Light:0,
      Led:false,
      Beep:false,
      area:'请求中',         //地区
      city:'请求中',         //城市
      airText:'请求中',      //空气优良
      airValue:0,            //空气指数
      weather:'请求中',      //天气
      weatherAdvice:'请求中' //天气建议

    };
  },

  components: {
    
  },

  methods: {
    onLedChange(event){
      var that =this
      console.log(event.mp.detail)
      let sw = event.mp.detail.value;
      if(sw){
        that.client.publish('/mysmarthome/sub','{"LED_sw":1}',function (err) {
          if(!err){
            console.log("成功下发命令——开灯");
          }
        })
      }else{
        that.client.publish('/mysmarthome/sub','{"LED_sw":0}',function (err) {
          if(!err){
            console.log("成功下发命令——关灯");
          }
        })
      }
    },
    onBeepChange(event){
      var that =this
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      if(sw){
        that.client.publish('/mysmarthome/sub','{"BEEP_sw":1}',function (err) {
          if(!err){
            console.log("成功下发命令——警报");
          }
        })
      }else{
        that.client.publish('/mysmarthome/sub','{"BEEP_sw":0}',function (err) {
          if(!err){
            console.log("成功下发命令——关闭警报");
          }
        })
      }
    }
  },

  //created () {
    // let app = getApp()
  //}
  onShow(){
    var that =this;
    that.client = connect(mqttUrl)
    that.client.on('connect',function (){
      console.log("成功连接mqtt服务器！");
      that.client.subscribe("/mysmarthome/pub",function (err) {
        if(!err){
          console.log("成功订阅设备上行数据Topic!");
        }
      })
    })
    //以下是发送消息时触发行为
    that.client.on('message',function (topic,message) {
      console.log(topic);
      //message是十六进制的Buffer字节流
      //以下是消息解析过程
      let dataFromDev ={};
      dataFromDev = JSON.parse(message);
      console.log(dataFromDev);
      that.Temp = dataFromDev.Temp;
      that.Hum=dataFromDev.Hum;
      that.Light=dataFromDev.Light;
      that.Led=dataFromDev.Led;
      that.Beep=dataFromDev.Beep;
    });

    wx.getLocation({
    type: 'wgs84',
    success (res) {
      const latitude = res.latitude;
      const longitude = res.longitude;
      const key='d439575f4fee445b90e3cf53e4ccd4da';
      wx.request({
        url: `https://devapi.qweather.com/v7/weather/now?location=${longitude},${latitude}&key=${key}`, //获取天气数据的api接口地址
        success (res) {
          console.log(res.data);
          const { basic,now } =res.data.Heweather[0];
          that.area=basic.Location;
          that.city=basic.parent_city;
          that.weather = now.cond_txt;
        }
      });
    }
  });
}
};
</script>

<style lang="scss" scoped>
.wrapper{
  padding: 15px;
  .header-wrapper{
    background-color: #3d7ef9;
    border-radius: 20px;
    color: #fff;
    box-shadow: #d6d6d6 0px 0px 5px;
    padding: 15px 30px;
    .header-title{
      display: flex;
      justify-content: space-between;
    }
    .header-text{
      font-size: 32px;
      font-weight: 400;
      display: flex;
      justify-content: space-between;
    }
    .weather-advice{
      margin-top: 20px;
      font-size: 12px;
    }
  }
  .data-wrapper{
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    .data{
      background-color: #fff;
      width: 150px;
      height: 80px;
      border-radius: 20px;
      display: flex;
      justify-content: space-around;
      padding: 0 8px;
      box-shadow: #d6d6d6 0px 0px 5px;
      .data-logo{
        height: 36px;
        width: 36px;
        margin-top: 15px;
      }
      .data-text{
        margin-top: 15px;
        color: #7f7f7f;
        .data-title{
          text-align: right;
        }
        .data-value{
          font-size: 26px;
        }
      }

    }
  }

}
</style>
