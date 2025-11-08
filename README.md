const int buttonPin = 2;
const int RledPin = 9;
const int GledPin = 10;
const int BledPin = 11;

int mood = 10; // 初始中性心情
const int maxMood = 20;
const int minMood = 0;

int buttonState = 0;
bool ButtonPressed = false;

unsigned long touchedTimer = 0;
unsigned long reducedTimer = 0;
const long unTouchInterval = 5000; // 5秒無操作後開始下降
const long reducedInterval = 1500; // 每秒下降1.5分

// 閃爍變數
unsigned long blinkTimer = 0;
bool blinkState = false;

void setup(){
  pinMode(buttonPin, INPUT);
  pinMode(RledPin, OUTPUT);
  pinMode(GledPin, OUTPUT);
  pinMode(BledPin, OUTPUT);
}

void loop(){
  buttonState = digitalRead(buttonPin);

  // 按一下心情+1
  if(buttonState == HIGH && !ButtonPressed){
    mood++;
    if(mood > maxMood) mood = maxMood;
    touchedTimer = millis();
    ButtonPressed = true;
  }

  if(buttonState == LOW && ButtonPressed){
    ButtonPressed = false;
  }

  // 心情自動下降
  unsigned long currentMillis = millis();
  if(currentMillis - touchedTimer > unTouchInterval){
    if(currentMillis - reducedTimer > reducedInterval){
      mood--;
      if(mood < minMood) mood = minMood;
      reducedTimer = currentMillis;
    }
  }

  showLEDState(mood);
}

void showLEDState(int mood){
  int r = 0, g = 0, b = 0;

  // 剩 5 分以下 → 閃爍紅燈
  if(mood <= 5){
    unsigned long currentMillis = millis();
    if(currentMillis - blinkTimer > 500){
      blinkState = !blinkState;
      blinkTimer = currentMillis;
    }
    if(blinkState){
      r = 255; g = 0; b = 255;
    } else {
      r = 0; g = 0; b = 0;
    }
  }
  else if(mood <= 10){
    // 5~10：紅 -> 綠 漸變
    float ratio = (mood - 5) / 5.0;
    r = 255 * (1 - ratio);
    g = 255 * ratio;
    b = 0;
  }
  else if(mood <= 20){
    // 10~20：綠 -> 藍 漸變
    float ratio = (mood - 10) / 10.0;
    r = 0;
    g = 255 * (1 - ratio);
    b = 255 * ratio;
  }

  analogWrite(RledPin, r);
  analogWrite(GledPin, g);
  analogWrite(BledPin, b);
}
