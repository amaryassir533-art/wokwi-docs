#include <Keypad.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Sensors
#define PIR1 2
#define PIR2 3
#define PIR3 4

#define DOOR1 5
#define DOOR2 6
#define DOOR3 7

// Outputs
#define BUZZER 8

#define GREEN_LED 9
#define YELLOW_LED 10
#define RED_LED 11

// Password
String password = "1234";
String inputPassword = "";

// Keypad
const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] =
{
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {22,23,24,25};
byte colPins[COLS] = {26,27,28,29};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);


// System status
bool armed = false;


void setup()
{
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();

  pinMode(PIR1, INPUT);
  pinMode(PIR2, INPUT);
  pinMode(PIR3, INPUT);

  pinMode(DOOR1, INPUT_PULLUP);
  pinMode(DOOR2, INPUT_PULLUP);
  pinMode(DOOR3, INPUT_PULLUP);

  pinMode(BUZZER, OUTPUT);

  pinMode(GREEN_LED, OUTPUT);
  pinMode(YELLOW_LED, OUTPUT);
  pinMode(RED_LED, OUTPUT);

  digitalWrite(GREEN_LED, HIGH);

  lcd.setCursor(0,0);
  lcd.print("Security System");
  lcd.setCursor(0,1);
  lcd.print("Enter Password");

}


void loop()
{

  char key = keypad.getKey();

  if(key)
  {
    if(key == '#')
    {
      checkPassword();
    }
    else if(key == '*')
    {
      inputPassword="";
      lcd.clear();
      lcd.print("Cleared");
      delay(1000);
    }
    else
    {
      inputPassword += key;
      lcd.setCursor(0,1);
      lcd.print(inputPassword);
    }
  }


  if(armed)
  {
    checkRooms();
  }

}



void checkPassword()
{

  if(inputPassword == password)
  {

    armed = !armed;

    lcd.clear();

    if(armed)
    {
      lcd.print("System Armed");
      digitalWrite(GREEN_LED, LOW);
      digitalWrite(YELLOW_LED, HIGH);
    }

    else
    {
      lcd.print("System OFF");
      digitalWrite(GREEN_LED, HIGH);
      digitalWrite(YELLOW_LED, LOW);
      digitalWrite(RED_LED, LOW);
      noTone(BUZZER);
    }

    delay(1500);
  }

  else
  {
    lcd.clear();
    lcd.print("Wrong Password");
    delay(1500);
  }

  inputPassword="";
}



void checkRooms()
{

  if(digitalRead(PIR1)==HIGH || digitalRead(DOOR1)==LOW)
  {
    alarm(1);
  }


  else if(digitalRead(PIR2)==HIGH || digitalRead(DOOR2)==LOW)
  {
    alarm(2);
  }


  else if(digitalRead(PIR3)==HIGH || digitalRead(DOOR3)==LOW)
  {
    alarm(3);
  }

}



void alarm(int room)
{

  digitalWrite(RED_LED,HIGH);

  tone(BUZZER,1000);

  lcd.clear();
  lcd.print("INTRUDER ROOM ");
  lcd.print(room);

  delay(1000);

}
