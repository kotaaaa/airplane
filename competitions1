//working comletely, comfirmed on 2015/07/30 06:47 JST
//
//I2C通信ライブラリを取り込む
#include <Wire.h>

//デジタルコンパスモジュール i2cのピン設定はデフォルト SCL:A5 SDA:A4

//赤外線距離センサ　ピン設定
#define DISTANCE_PIN A0

//モータ用PWM　ピン設定
#define MOTOR_PIN_L 3
#define MOTOR_PIN_R 9
#define MOTOR_PIN_V 10

//指令値
#define COMMAND_ANGLE  0
#define COMMAND_HEIGHT 150

//時間ステージ数
#define STAGE_NUM 3
//ステージ1:直進、ステージ2:

//指令値テーブル
int commandTime[STAGE_NUM]   = {100000,   20,  30};
int commandAngle[STAGE_NUM]  = {180,  90, 180};
int commandHeight[STAGE_NUM] = {100, 200, 100};
int stage = 0;

//デジタルコンパスモジュールのアドレス設定
int compassAddress = 0x42 >> 1; //=0x21
//読み込み値の変数を用意
int compass_Deg = 0;
//センサーの角度の差分
int sensorAngleDiff;

//モータのデューティ比設定
int motorDuty_L = 128;
int motorDuty_R = 128;
int motorDuty_V = 128;

//赤外線距離センサ
float distanceAd_Volt = 0.0;
float distance; //距離
float distanceDiff; //設定値との差分

//時間
unsigned long currentTime_Sec;

//モータのデューティ比を0-255に収める
int motorDuty(int duty){
  if(duty > 255){
    return 255;
  }else if(duty < 0){
    return 0;
  }else{
    return duty;
  }
}

void setup() {
  //デジタルコンパスモジュールの設定
  //I2C通信開始 
  Wire.begin();
  //Continuous Modeに設定する
  Wire.beginTransmission(compassAddress);
  //RAM書き込み用コマンド
  Wire.write('G');
  //書き込み先指定
  Wire.write(0x74);
  //モード設定
  Wire.write(0x72);
  //通信終了
  Wire.endTransmission();
  //処理時間
  delayMicroseconds(70);

  //モータ用PWMの設定
  pinMode(MOTOR_PIN_L, OUTPUT);
  pinMode(MOTOR_PIN_R, OUTPUT);
  pinMode(MOTOR_PIN_V, OUTPUT);

  //モータ正逆ピンの設定
  pinMode(2,  OUTPUT);     //MOTOR_L
  pinMode(6,  OUTPUT);
  pinMode(8,  OUTPUT);     //MOTOR_R
  pinMode(11, OUTPUT);
  pinMode(12, OUTPUT);     //MOTOR_V
  pinMode(13, OUTPUT);
  //モータ正転：2つのピンのどちらかをHIGHにすると正転or逆転，両方HIGHでブレーキ
  //ex1) digitalWrite(2,HIGH);
  //     digitalWrite(6,HIGH);  ブレーキ
  //ex2) digitalWrite(2,HIGH);
  //     digitalWrite(6,LOW);   正転
  digitalWrite(2,  HIGH);  //MOTOR_L
  digitalWrite(6,  LOW);
  digitalWrite(8,  LOW);  //MOTOR_R
  digitalWrite(11, HIGH);
  digitalWrite(12, LOW);  //MOTOR_V
  digitalWrite(13, HIGH);
  digitalWrite(7, HIGH);  // Hでスタンバイ解除

  //表示のためのシリアル通信開始
  Serial.begin(9600);

} 
  
void loop() {
  //デバイスに２バイト分のデータを要求する
  Wire.requestFrom(compassAddress, 2);

  //要求したデータが２バイト分来たら
  if(Wire.available()>1){
    //デジタルコンパスモジュール
    //１バイト分のデータの読み込み 
    compass_Deg = Wire.read();
    //読み込んだデータを８ビット左シフトしておく
    compass_Deg = compass_Deg << 8;
    //次の１バイト分のデータを読み込み
    //一つ目のデータと合成（２バイト）
    compass_Deg += Wire.read();
    //２バイト分のデータを１０で割る
    compass_Deg /= 10;
    Serial.print("Compass [deg] : ");
    Serial.print(compass_Deg);
    Serial.print("\t");
    //指令角度を表示
    Serial.print("CommandAngle [deg] : ");
    Serial.print(commandAngle[stage]);
    Serial.print("\t");

    sensorAngleDiff = compass_Deg - commandAngle[stage];      //コンパスセンサと指令値との差分
    motorDuty_L     = /*sensorAngleDiff *  1.0 +*/ 255;     //角度の差分を用いて比例制御によりモータのデューティ比が決定
    motorDuty_R     = /*sensorAngleDiff *  -1.0 +*/ 255;
    motorDuty_L     = motorDuty(motorDuty_L);           //デューティ比を0-255までに収める
    motorDuty_R     = motorDuty(motorDuty_R);
    Serial.print("MotorDuty_L : ");
    Serial.print(motorDuty_L);
    Serial.print("\t");
    Serial.print("MotorDuty_R : ");
    Serial.print(motorDuty_R);
    Serial.print("\t");

    //赤外線距離センサのデータをA3ピンから読む
    distanceAd_Volt = analogRead(DISTANCE_PIN);
    distanceAd_Volt = distanceAd_Volt/1024.0*5.0;           //電圧[V]に変換
    distance = 1.0/(0.007*distanceAd_Volt-0.0075);          //距離[cm]に変換
    Serial.print("Distance : ");
    Serial.print(distance);
    Serial.print("\t");
    //指令高度を表示
    Serial.print("CommandHeight [cm] : ");
    Serial.print(commandHeight[stage]);
    Serial.print("\n");

    distanceDiff = distance - commandHeight[stage];               //距離センサと指令値との差分
    motorDuty_V  = 255;                      //高度の差分を用いて比例制御によりモータのデューティ比が決定
    motorDuty_V  = motorDuty(motorDuty_V);                  //デューティ比を0-255までに収める
    Serial.print("MotorDuty_V : ");
    Serial.print(motorDuty_V);
    Serial.print("\t");

    currentTime_Sec = millis()/1000;
    //スタートからの時間を表示
    Serial.print("Time[sec] : ");
    Serial.print(currentTime_Sec);
    Serial.print("\n");
  }

  if(currentTime_Sec > commandTime[stage]){
    stage++;
  }
  if(stage >= STAGE_NUM){
    stage = STAGE_NUM-1;
  }

  //モータ用PWM 0-255
  analogWrite(MOTOR_PIN_L, motorDuty_L);
  analogWrite(MOTOR_PIN_R, motorDuty_R);
  analogWrite(MOTOR_PIN_V, motorDuty_V);
  
  //Serial.println("Start");
  //処理のために少し待つ（20Hz)
  delay(500); 
}
