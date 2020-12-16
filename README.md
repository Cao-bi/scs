

（音乐播放和二维码）

#include <Arduino.h>
#include <U8g2lib.h>
#include <Wire.h>
#include "qrcode.h"

U8G2_SSD1306_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE, /* clock=*/ 4, /* data=*/ 5);   // ESP32 Thing, HW I2C with pin remapping
#define NOTE_D0 -1
#define NOTE_D1 294
#define NOTE_D2 330
#define NOTE_D3 350
#define NOTE_D4 393
#define NOTE_D5 441
#define NOTE_D6 495
#define NOTE_D7 556

#define NOTE_DL1 147
#define NOTE_DL2 165
#define NOTE_DL3 175
#define NOTE_DL4 196
#define NOTE_DL5 221
#define NOTE_DL6 248
#define NOTE_DL7 278

#define NOTE_DH1 589
#define NOTE_DH2 661
#define NOTE_DH3 700
#define NOTE_DH4 786
#define NOTE_DH5 882
#define NOTE_DH6 990
#define NOTE_DH7 112

#define WHOLE 1
#define HALF 0.5
#define QUARTER 0.25
#define EIGHTH 0.25
#define SIXTEENTH 0.625

//整首曲子的音符部分
int tune[] =
{
  NOTE_DH1, NOTE_D6, NOTE_D5, NOTE_D6, NOTE_D0,
  NOTE_DH1, NOTE_D6, NOTE_D5, NOTE_DH1, NOTE_D6, NOTE_D0, NOTE_D6,
  NOTE_D6, NOTE_D6, NOTE_D5, NOTE_D6, NOTE_D0, NOTE_D6,
  NOTE_DH1, NOTE_D6, NOTE_D5, NOTE_DH1, NOTE_D6, NOTE_D0,

  NOTE_D1, NOTE_D1, NOTE_D3,
  NOTE_D1, NOTE_D1, NOTE_D3, NOTE_D0,
  NOTE_D6, NOTE_D6, NOTE_D6, NOTE_D5, NOTE_D6,
  NOTE_D5, NOTE_D1, NOTE_D3, NOTE_D0,
  NOTE_DH1, NOTE_D6, NOTE_D6, NOTE_D5, NOTE_D6,
  NOTE_D5, NOTE_D1, NOTE_D2, NOTE_D0,
  NOTE_D7, NOTE_D7, NOTE_D5, NOTE_D3,
  NOTE_D5,
  NOTE_DH1, NOTE_D0, NOTE_D6, NOTE_D6, NOTE_D5, NOTE_D5, NOTE_D6, NOTE_D6,
  NOTE_D0, NOTE_D5, NOTE_D1, NOTE_D3, NOTE_D0,
  NOTE_DH1, NOTE_D0, NOTE_D6, NOTE_D6, NOTE_D5, NOTE_D5, NOTE_D6, NOTE_D6,
  NOTE_D0, NOTE_D5, NOTE_D1, NOTE_D2, NOTE_D0,
  NOTE_D3, NOTE_D3, NOTE_D1, NOTE_DL6,
  NOTE_D1,
  NOTE_D3, NOTE_D5, NOTE_D6, NOTE_D6,
  NOTE_D3, NOTE_D5, NOTE_D6, NOTE_D6,
  NOTE_DH1, NOTE_D0, NOTE_D7, NOTE_D5,
  NOTE_D6,
};

//曲子的节拍，即音符持续时间
float duration[] =
{
  1, 1, 0.5, 0.5, 1,
  0.5, 0.5, 0.5, 0.5, 1, 0.5, 0.5,
  0.5, 1, 0.5, 1, 0.5, 0.5,
  0.5, 0.5, 0.5, 0.5, 1, 1,

  1, 1, 1 + 1,
  0.5, 1, 1 + 0.5, 1,
  1, 1, 0.5, 0.5, 1,
  0.5, 1, 1 + 0.5, 1,
  0.5, 0.5, 0.5, 0.5, 1 + 1,
  0.5, 1, 1 + 0.5, 1,
  1 + 1, 0.5, 0.5, 1,
  1 + 1 + 1 + 1,
  0.5, 0.5, 0.5 + 0.25, 0.25, 0.5 + 0.25, 0.25, 0.5 + 0.25, 0.25,
  0.5, 1, 0.5, 1, 1,
  0.5, 0.5, 0.5 + 0.25, 0.25, 0.5 + 0.25, 0.25, 0.5 + 0.25, 0.25,
  0.5, 1, 0.5, 1, 1,
  1 + 1, 0.5, 0.5, 1,
  1 + 1 + 1 + 1,
  0.5, 1, 0.5, 1 + 1,
  0.5, 1, 0.5, 1 + 1,
  1 + 1, 0.5, 0.5, 1,
  1 + 1 + 1 + 1
};

int length;//定义一个变量用来表示共有多少个音符
int tonePin = 8; //蜂鸣器的pin
void setup() {
  {
  pinMode(tonePin, OUTPUT); //设置蜂鸣器的pin为输出模式
  length = sizeof(tune) / sizeof(tune[0]); //这里用了一个sizeof函数，查出数组里有多少个音符
}

  // put your setup code here, to run once:\
  // init u8g2
  u8g2.begin();

  // gen the QR code
  QRCode qrcode;
  uint8_t qrcodeData[qrcode_getBufferSize(3)];

  qrcode_initText(&qrcode, qrcodeData, 3 , ECC_LOW, "https://blog.craftyun.cn/post/199.html");

  // start draw
  u8g2.firstPage();
  do {
    // get the draw starting point,128 and 64 is screen size
    uint8_t x0 = (128 - qrcode.size * 2) / 2;
    uint8_t y0 = (64 - qrcode.size * 2) / 2;
    
    // get QR code pixels in a loop
    for (uint8_t y = 0; y < qrcode.size; y++) {
      for (uint8_t x = 0; x < qrcode.size; x++) {
        // Check this point is black or white
        if (qrcode_getModule(&qrcode, x, y)) {
          u8g2.setColorIndex(1);
        } else {
          u8g2.setColorIndex(0);
        }
        // Double the QR code pixels
        u8g2.drawPixel(x0 + x * 2, y0 + y * 2);
        u8g2.drawPixel(x0 + 1 + x * 2, y0 + y * 2);
        u8g2.drawPixel(x0 + x * 2, y0  + 1 + y * 2);
        u8g2.drawPixel(x0 + 1 + x * 2, y0 + 1 + y * 2);
      }
    }

  } while ( u8g2.nextPage() );
}

void loop() 
{
  for (int x = 0; x < length; x++) //循环音符的次数
  {
    tone(tonePin, tune[x]); //依次播放tune数组元素，即每个音符
    delay(400 * duration[x]); //每个音符持续的时间，即节拍duration，400是调整时间的越大，曲子速度越慢，越小曲子速度越快
    noTone(tonePin);//停止当前音符，进入下一音符
  }
  delay(5000);//等待5秒后，循环重新开始
}















（温度测量 端到端）
#include "ESP8266.h"
#include "dht11.h"
#include "SoftwareSerial.h"

//配置ESP8266WIFI设置
#define SSID "此处改为自己的WIFI名称"    //填写2.4GHz的WIFI名称，不要使用校园网
#define PASSWORD "此处改为自己的WIFI密码"//填写自己的WIFI密码
#define HOST_NAME "api.heclouds.com"  //API主机名称，连接到OneNET平台，无需修改
#define DEVICE_ID "OneNet设备ID"       //填写自己的OneNet设备ID
#define HOST_PORT (80)                //API端口，连接到OneNET平台，无需修改
String APIKey = "OneNet与设备绑定的APIKey"; //与设备绑定的APIKey

#define INTERVAL_SENSOR 5000 //定义传感器采样及发送时间间隔
  #include <Arduino.h>
#include <U8g2lib.h>
#include <Wire.h>
#include "qrcode.h"

U8G2_SSD1306_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE, /* clock=*/ 4, /* data=*/ 5);   // ESP32 Thing, HW I2C with pin remapping
#define NOTE_D0 -1
#define NOTE_D1 294
#define NOTE_D2 330
#define NOTE_D3 350
#define NOTE_D4 393
#define NOTE_D5 441
#define NOTE_D6 495
#define NOTE_D7 556

#define NOTE_DL1 147
#define NOTE_DL2 165
#define NOTE_DL3 175
#define NOTE_DL4 196
#define NOTE_DL5 221
#define NOTE_DL6 248
#define NOTE_DL7 278

#define NOTE_DH1 589
#define NOTE_DH2 661
#define NOTE_DH3 700
#define NOTE_DH4 786
#define NOTE_DH5 882
#define NOTE_DH6 990
#define NOTE_DH7 112

#define WHOLE 1
#define HALF 0.5
#define QUARTER 0.25
#define EIGHTH 0.25
#define SIXTEENTH 0.625

//创建dht11示例

dht11 DHT11;

//定义DHT11接入Arduino的管脚
#define DHT11PIN 4

//定义ESP8266所连接的软串口
/*********************
 * 该实验需要使用软串口
 * Arduino上的软串口RX定义为D3,
 * 接ESP8266上的TX口,
 * Arduino上的软串口TX定义为D2,
 * 接ESP8266上的RX口.
 * D3和D2可以自定义,
 * 但接ESP8266时必须恰好相反
 *********************/
SoftwareSerial mySerial(3, 2);
ESP8266 wifi(mySerial);

void setup()
{
  mySerial.begin(115200); //初始化软串口
  Serial.begin(9600);     //初始化串口
  Serial.print("setup begin\r\n");

  //以下为ESP8266初始化的代码
  Serial.print("FW Version: ");
  Serial.println(wifi.getVersion().c_str());

  if (wifi.setOprToStation()) {
    Serial.print("to station ok\r\n");
  } else {
    Serial.print("to station err\r\n");
  }

  //ESP8266接入WIFI
  if (wifi.joinAP(SSID, PASSWORD)) {
    Serial.print("Join AP success\r\n");
    Serial.print("IP: ");
    Serial.println(wifi.getLocalIP().c_str());
  } else {
    Serial.print("Join AP failure\r\n");
  }

  Serial.println("");
  Serial.print("DHT11 LIBRARY VERSION: ");
  Serial.println(DHT11LIB_VERSION);

  mySerial.println("AT+UART_CUR=9600,8,1,0,0");
  mySerial.begin(9600);
  Serial.println("setup end\r\n");
}

unsigned long net_time1 = millis(); //数据上传服务器时间
void loop(){

  if (net_time1 > millis())
    net_time1 = millis(); 
    int n=analogRead(A3);
     Serial.println(n);               // 用于IDE串口观察窗
     delay(100);       //防止串口写入速度过快

 

   
  
    if (wifi.createTCP(HOST_NAME, HOST_PORT)) { //建立TCP连接，如果失败，不能发送该数据
      Serial.print("create tcp ok\r\n");
      char buf[10];
      //拼接发送data字段字符串
      String jsonToSend = "{\"顾客温度":";
       dtostrf(n, 1, 2, buf);

      jsonToSend += "\"" +  String(buf)+ "\"";
      
      
      jsonToSend += "}";

      //拼接POST请求字符串
      String postString = "POST /devices/";
      postString += DEVICE_ID;
      postString += "/datapoints?type=3 HTTP/1.1";
      postString += "\r\n";
      postString += "api-key:";
      postString += APIKey;
      postString += "\r\n";
      postString += "Host:api.heclouds.com\r\n";
      postString += "Connection:close\r\n";
      postString += "Content-Length:";
      postString += jsonToSend.length();
      postString += "\r\n";
      postString += "\r\n";
      postString += jsonToSend;
      postString += "\r\n";
      postString += "\r\n";
      postString += "\r\n";

      const char *postArray = postString.c_str(); //将str转化为char数组

      Serial.println(postArray);
      wifi.send((const uint8_t *)postArray, strlen(postArray)); //send发送命令，参数必须是这两种格式，尤其是(const uint8_t*)
      Serial.println("send success");
      if (wifi.releaseTCP()) { //释放TCP连接
        Serial.print("release tcp ok\r\n");
      } else {
        Serial.print("release tcp err\r\n");
      }
      postArray = NULL; //清空数组，等待下次传输数据
    } else {
      Serial.print("create tcp err\r\n");
    }

    Serial.println("");

    net_time1 = millis();
  }
}


（wxml与js）
const db=wx.cloud.database().collection('costumer')
Page({

  data:{
  temp:"还没有查看",
  result:"",
  temp1:"",
  opacity: 0.4,
    disabled: true,
  },
  Get_temp : function()
  { 
    const devicesid ='657106001'//你的设备id 
const datastreams = '顾客温度'//你的数据流ID 可以多个
const apikey = 'lm8Ro0=Ac=oWhC8aRQ7X0nZjpDk='//你的apikey
//获取设备数据流
    wx.request({
      url: `https://api.heclouds.com/devices/${devicesid}/datastreams?datastream_ids=${datastreams}`,
      header: {
            "api-key": `${apikey}`,
          },
      success(res){
        console.log(res.data);//请求成功返回数据
        var resdata=res.data;
    if (app.searchDataCallback) {
          app.searchDataCallback(resdata)
        }

      }, 
      fail(){//请求失败
        wx.showToast({
          title: '与服务器通信失败',
          icon: 'fail',
          duration: 2000
        })
      }
    })
     const app=getApp();
app.searchDataCallback = res => {
  this.setData({
    temp:res.data[0].current_value+"度",
    temp1:res.data[0].current_value
  }) 
  

   if(res.data[0].current_value>37)
   {
     this.setData({
       result:"对不起，您体温异常不能进店",
     })
   }
   if(res.data[0].current_value<=37){
     this.setData({
       result:"您体温正常，欢迎进店",
       opacity: 1,
       disabled: false,
     })
   }
  };

  
  },
  jumpToStore:function(){
    wx.navigateTo({
      url:'/pages/store/store',
    })
  },
  jumpToRecommend:function(){
    wx.navigateTo({
      url:'/pages/extra/recommend',
    })
  },jumpToAboutus:function(){
    wx.navigateTo({
      url:'/pages/extra/aboutus',
    })
  },
  onAdd(){
    db.get({
      success:function(res){
        console.log("查询成功",res),
        console.log(res.data.length),
        db.add({
          data:{
            num:res.data.length
          },
          success:function(res){
            console.log("添加数据成功！",res)
            },fail(res){
              console.log("添加数据失败",res) 
            },
          })
      },fail(res){
        console.log("查询失败",res)
      }
    })
    /**db.add({
      data:{
        age:20

      },
      success:function(res){
        console.log("添加数据成功！",res)
        },fail(res){
          console.log("添加数据失败",res) 
        },
      })**/
    }
  })
<view class="container">
<view class="title">我的体温</view>
<button type="primary" bindtap="Get_temp">查看</button>
<view class="content" >{{temp}}</view>
<view class="content">{{result}}</view>
<button id="btn2" type="primary" style="opacity: {{opacity}}" disabled="{{disabled}}" bindtap="jumpToStore">
<view bindtap="onAdd">进店</view></button>
</view>

