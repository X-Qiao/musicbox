# 藍芽音樂盒
### 第八組
B1042092 李心喬   B1042109   楊語涵

## 動機
現今總是隨處都可以聽到音樂，不管是咖啡廳、餐廳，還是健身房，音樂可以使人放鬆，人們對於播放音樂的需求逐漸上升，播放音樂的工具從唱片機演變成小型的MP3播放器，最後再演變至現在的智慧型手機。我們希望能透過在這堂課課程中所學的知識， 結合音樂的元素， 製作出藍牙音樂盒。
## 成果
1、LCD可以顯示目前播放的歌曲

2、蜂鳴器可播放歌曲

3、共可播放5首歌曲

4、手機連接藍芽後，可以控制播放音樂：

手機傳送1 ~ 5：可選擇播放1 ~ 5首歌

手機傳送0：暫停，再傳送一次0繼續播放

手機傳送6：終止

手機傳送8：上一首

手機傳送9：下一首
## 程式
引用函式庫、初始化 LCD 模組
```
#include <BluetoothSerial.h>        //引用藍牙函式庫
#include <LiquidCrystal.h>           //引入 LCD 函式庫
LiquidCrystal lcd(12, 14, 27, 26, 33, 32);       // 初始化 LCD 模組
```
建立藍牙物件
```
BluetoothSerial myBT;        //建立藍牙物件，名稱叫作myBT
char incomeData;                 //接收資料用的變數
```
選定PWM通道、歌曲的時基、宣告陣列note
```
const byte channel = 0;           //選定PWM通道0
const int timeBase = 60;         //以32分音符為歌曲的時基，決定曲目的速度
note_t note[]={NOTE_MAX,NOTE_C,NOTE_D,NOTE_E,NOTE_F,NOTE_G,NOTE_A,
NOTE_B,}; //以系統預定的自訂型別：note_t，宣告陣列。目的是為了從數字轉為note_t型別
```
音樂盒中歌曲的音程、自行編碼、時間倍數(共5首)
```
//新不了情
const int melody1[3][12] = {                 //SONG1的音程、自行編碼、時間倍數
  {5,   6, 5, 5, 5, 5, 6,      6, 6, 6, 5, 6,},       //音程
  {5,   3, 6, 6, 6, 7, 1,      2, 3, 2, 5, 1, },      //自行編碼
  {8,   12, 4, 8, 2.67, 2.67, 2.67,      6, 6, 2, 2, 12,} };    //時間倍數
//豆豆龍
const int melody2[3][18] = {                //SONG2的音程、自行編碼、時間倍數
  {5, 5, 5, 5, 5, 5, 5, 5, 5, 5,  5, 5, 5, 4, 5, 5, 5,  5, },   //音程18
  {1, 2, 3, 4, 5, 3, 1, 5, 4, 2,  0, 4, 2, 7, 4, 3, 1,  0, },   //自行編碼
  {4, 4, 4, 4, 8, 4, 8, 8, 8, 24, 4, 8, 4, 8, 8, 8, 24, 4, }   };    //時間倍數
//天空之城
const int melody3[3][13] = {                //SONG3的音程、自行編碼、時間倍數
  {5,  4, 5, 5,   4, 4, 4,   4, 4, 4, 5,   4,  },   //音程
  {1,  7, 1, 3,   7, 3, 3,   6, 5, 6, 1,   5,  },   //自行編碼
  {12, 4, 8, 8,  24, 4, 4,  12, 4, 8, 8,  24,  }  };    //時間倍數
//魔女宅急便
const int melody4[3][24] = {                 //SONG4的音程、自行編碼、時間倍數
  {5,5,6,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,6,5,5,5,5,5,},   //音程
  {0,3,1,0,0,3,7,0,0,3,6,5,4,5,1,2,4,6,1,7,5,3,2,3,},  //自行編碼
  {2.4,4,4,4,4,4,4,4,4,4,8,4,4,12,4,4,4,4,4,4,4,4,4,8,}  };    //時間倍數
//蝴蝶
const int melody5[3][20] = {                 //SONG5的音程、自行編碼、時間倍數
  {5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,},   //音程
  {1,1,2,3,3,2,1,2,3,1,3,3,4,5,5,4,3,4,5,3,},  //自行編碼
  {8,4,4,8,8,4,4,4,4,8,8,4,4,8,8,4,4,4,4,8,}  };    //時間倍數
```
定義藍牙的名字、設定蜂鳴器腳位、PWM 通道、LCD字幕
```
void setup() {
  Serial.begin(9600);
  myBT.begin("BT:DOOOOOOG");       //定義藍牙的名字，此設定為BT:DOOOOOOG

  ledcSetup(channel, 10000, 8); //PWM 通道 0，頻率 10kHz，解析度 8 位元
  ledcAttachPin(25, channel);   //將 PWM 通道 0，接於 GPIO 25 腳

  lcd.begin(16, 2);        // 設定LCD字幕為 16*2
}
```
使LCD可以顯示目前播放的歌曲

手機傳送1 ~ 5：可選擇播放1 ~ 5首歌

手機傳送0：暫停，再傳送一次0繼續播放

手機傳送6：終止

手機傳送8：上一首

手機傳送9：下一首
```
void loop() {
  int y=0;
  
  if (myBT.available()) {         //如果藍牙模組收到資料
    incomeData = myBT.read();     //將資料讀出
    Serial.print("從藍牙接收到：");
    Serial.println(incomeData);   //從序列埠印出手機傳來的字元
for(int j=0;j<=1;j++){
  if (incomeData == '6')
  break;
      if (incomeData == '1'||y==1){       //如果接收到字元'1'或y=1
        y=1;
        lcd.setCursor(1,0);          // LCD移動游標至第1行第0列
        lcd.print("song1");             // 顯示song1
        for (int i = 0; i <= 11; i++) {  //利用迴圈把所有的音符播放出來
          if (myBT.available()) {         //如果藍牙模組收到資料
            incomeData = myBT.read();     //將資料讀出
            Serial.print("從藍牙接收到：");
            Serial.println(incomeData);   //從序列埠印出手機傳來的字元

        
            if (incomeData == '6'||incomeData == '8'||incomeData == '9')                 
              break;                                                  //如果接收到字元'6'、'8'、'9'，停止

            if (incomeData == '0'){  //如果接收到字元'0'，暫停
              for (;;){
                delay(500);
                if (myBT.available()) {         //如果藍牙模組收到資料
                  incomeData = myBT.read();     //將資料讀出
                  Serial.print("從藍牙接收到：");
                  Serial.println(incomeData);

                  if (incomeData == '0')  //如果再次接收到字元'0'，繼續播放
                    break;
                }
              }
            }
          }
          
          else {
          int temp1 = melody1[1][i];       //把song1的自行編碼，從陣列讀出
          note_t myNote1 = note[temp1];    //讀取note[]陣列的資料，其格式為note_t，為下一行的參數
    
          ledcWriteNote(channel, myNote1, melody1[0][i]);     //發出音符
          delay(melody1[2][i] * timeBase);                   //延時：時基x倍數

          ledcWriteTone(channel, 0);     //從通道0發聲，頻率0Hz代表靜音
          delay(timeBase/3);                  //短暫的靜音，讓各音符聲音更鮮明
          }             
          }
        myBT.println("song1");           //回傳song1給手機
        if (incomeData == '9'){            //如果接收到字元'9'，播放下一首
          delay(500);
          y=2;
          incomeData ='6';
        }
        }  
      
      if (incomeData == '2'||y==2){            //如果接收到字元'2'或y=2
        y=2;
        lcd.setCursor(1,0);          // 移動游標至第1行第0列
        lcd.print("song2");             // 顯示song2
        for (int i = 0; i <= 17; i++) {  //利用迴圈把song2的音符播放出來
          if (myBT.available()) {         //如果藍牙模組收到資料
            incomeData = myBT.read();     //將資料讀出
            Serial.print("從藍牙接收到：");
            Serial.println(incomeData);   //從序列埠印出手機傳來的字元
          
            if (incomeData == '6'||incomeData == '8'||incomeData == '9')
              break;                                  //如果接收到字元'6'、'8'、'9'，停止

            if (incomeData == '0'){    //如果接收到字元'0'，暫停
              for (;;){
                delay(500);
                if (myBT.available()) {         //如果藍牙模組收到資料
                  incomeData = myBT.read();     //將資料讀出
                  Serial.print("從藍牙接收到：");
                  Serial.println(incomeData);

                  if (incomeData == '0')        //如果再次接收到字元'0'，繼續播放
                    break;
                }
              }
            }
          }

          else {
          int temp2 = melody2[1][i];       //把song2的自行編碼，從陣列讀出
          note_t myNote2 = note[temp2];    //讀取note[]陣列的資料，其格式為note_t，為下一行的參數
    
          ledcWriteNote(channel, myNote2, melody2[0][i]);     //發出音符
          delay(melody2[2][i] * timeBase);                   //延時：時基x倍數

          ledcWriteTone(channel, 0);     //從通道0發聲，頻率0Hz代表靜音
          delay(timeBase/3);                  //短暫的靜音，讓各音符聲音更鮮明
          }            
      }
      myBT.println("song2");              //回傳song2給手機
      if (incomeData == '8'){              //如果接收到字元'8'，播放上一首
        delay(500);
        y=1;
        incomeData ='0';
        }
      if (incomeData == '9'){              //如果接收到字元'9'，播放下一首
        delay(500);
        y=3;
        incomeData ='6';
        }
      } 
      
      if (incomeData == '3'||y==3){            //如果接收到字元'3'或y=3
        y=3;
        lcd.setCursor(1,0);          // 移動游標至第1行第0列
        lcd.print("song3");             // 顯示song3
        for (int i = 0; i <= 12; i++) {  //利用迴圈把所有的音符播放出來
          if (myBT.available()) {         //如果藍牙模組收到資料
            incomeData = myBT.read();     //將資料讀出
            Serial.print("從藍牙接收到：");
            Serial.println(incomeData);   //從序列埠印出手機傳來的字元
          
            if (incomeData == '6'||incomeData == '8'||incomeData == '9')
              break;                                                  //如果接收到字元'6'、'8'、'9'，停止

            if (incomeData == '0'){  //如果接收到字元'0'，暫停
              for (;;){
                delay(500);
                if (myBT.available()) {         //如果藍牙模組收到資料
                  incomeData = myBT.read();     //將資料讀出
                  Serial.print("從藍牙接收到：");
                  Serial.println(incomeData);

                  if (incomeData == '0')       //如果再次接收到字元'0'，繼續播放
                    break;
                }
              }
            }
          }

          else {
          int temp3 = melody3[1][i];       //把song3的自行編碼，從陣列讀出
          note_t myNote3 = note[temp3];    //讀取note[]陣列的資料，其格式為note_t，為下一行的參數
    
          ledcWriteNote(channel, myNote3, melody3[0][i]);     //發出音符
          delay(melody3[2][i] * timeBase);                   //延時：時基x倍數

          ledcWriteTone(channel, 0);     //從通道0發聲，頻率0Hz代表靜音
          delay(timeBase/3);                  //短暫的靜音，讓各音符聲音更鮮明
          }             
      }
      myBT.println("song3");              //回傳song3給手機
      if (incomeData == '8'){              //如果接收到字元'8'，播放上一首
        delay(500);
        y=2;
        incomeData ='0';
        }
      if (incomeData == '9'){               //如果接收到字元'9'，播放下一首
        delay(500);
        y=4;
        incomeData ='6';
        }
      }  

      if (incomeData == '4'||y==4){            //如果接收到字元'4'或y=4
        y=4;
        lcd.setCursor(1,0);          // 移動游標至第1行第0列
        lcd.print("song4");             // 顯示song4
        for (int i = 0; i <= 23; i++) {  //利用迴圈把所有的音符播放出來
          if (myBT.available()) {         //如果藍牙模組收到資料
            incomeData = myBT.read();     //將資料讀出
            Serial.print("從藍牙接收到：");
            Serial.println(incomeData);   //從序列埠印出手機傳來的字元
          
            if (incomeData == '6'||incomeData == '8'||incomeData == '9')
              break;                                                  //如果接收到字元'6'、'8'、'9'，停止

            if (incomeData == '0'){  //如果接收到字元'0'，暫停
              for (;;){
                delay(500);
                if (myBT.available()) {         //如果藍牙模組收到資料
                  incomeData = myBT.read();     //將資料讀出
                  Serial.print("從藍牙接收到：");
                  Serial.println(incomeData);

                  if (incomeData == '0')        //如果再次接收到字元'0'，繼續播放
                    break;
                }
              }
            }
          }

          else {
          int temp4 = melody4[1][i];       //把song4的自行編碼，從陣列讀出
          note_t myNote4 = note[temp4];    //讀取note[]陣列的資料，其格式為note_t，為下一行的參數
    
          ledcWriteNote(channel, myNote4, melody4[0][i]);     //發出音符
          delay(melody4[2][i] * timeBase);                   //延時：時基x倍數

          ledcWriteTone(channel, 0);     //從通道0發聲，頻率0Hz代表靜音
          delay(timeBase/3);                  //短暫的靜音，讓各音符聲音更鮮明
          }             
      }
      myBT.println("song4");              //回傳song4給手機
      if (incomeData == '8'){               //如果接收到字元'8'，播放上一首
        delay(500);
        y=3;
        incomeData ='0';
        }
      if (incomeData == '9'){              //如果接收到字元'9'，播放下一首
        delay(500);
        y=5;
        incomeData ='6';
        }
      }  

      if (incomeData == '5'||y==5){            //如果接收到字元'5'或y=5
        y=5;
        lcd.setCursor(1,0);          // 移動游標至第1行第0列
        lcd.print("song5");             // 顯示song5
        for (int i = 0; i <= 19; i++) {  //利用迴圈把所有的音符播放出來
          if (myBT.available()) {         //如果藍牙模組收到資料
            incomeData = myBT.read();     //將資料讀出
            Serial.print("從藍牙接收到：");
            Serial.println(incomeData);   //從序列埠印出手機傳來的字元
          
            if (incomeData == '6'||incomeData == '8'||incomeData == '9')
              break;                                                  //如果接收到字元'6'、'8'、'9'，停止

            if (incomeData == '0'){  //如果接收到字元'0'，暫停
              for (;;){
                delay(500);
                if (myBT.available()) {         //如果藍牙模組收到資料
                  incomeData = myBT.read();     //將資料讀出
                  Serial.print("從藍牙接收到：");
                  Serial.println(incomeData);

                  if (incomeData == '0')         //如果再次接收到字元'0'，繼續播放
                    break;
                }
              }
            }
          }

          else {
          int temp5 = melody5[1][i];       //把song5的自行編碼，從陣列讀出
          note_t myNote5 = note[temp5];    //讀取note[]陣列的資料，其格式為note_t，為下一行的參數
    
          ledcWriteNote(channel, myNote5, melody5[0][i]);     //發出音符
          delay(melody5[2][i] * timeBase);                   //延時：時基x倍數

          ledcWriteTone(channel, 0);     //從通道0發聲，頻率0Hz代表靜音
          delay(timeBase/3);                  //短暫的靜音，讓各音符聲音更鮮明
          }             
          }
      myBT.println("song5");              //回傳song5給手機
      if (incomeData == '8'){               //如果接收到字元'8'，播放上一首
        delay(500);
        y=4;
        incomeData ='0';
        }
             }  
      if (incomeData == '6'){            //如果接收到字元'6'
      myBT.println("break");           //回傳break給手機
      }
  }
}
  delay(20);
}
```
