# project
Add code to GitHub
#include <Arduino.h>
#include <LiquidCrystal.h>

#define speed_pac 150 //speed of players
#define speed_fant 1000
#define maxx 15 //board size
#define maxy 1
#define btnRigth  0 //configure the board buttons
#define btnUp     1
#define btnDown   2
#define btnLeft   3
#define btnSelect 4
#define btnNone   5
void(* resetFunc) (void) = 0;//declare reset function at address 0

LiquidCrystal lcd (8,9,4,5,6,7);

//draw pacman icon
byte pacman[8]={
  B00000,
  B00000,
  B01110,
  B11011,
  B11100,
  B01110,
  B00000,
  B00000
};
//draw ghost icon
byte ghost[8]={
  B00000,
  B00000,
  B01110,
  B10101,
  B11111,
  B11111,
  B10101,
  B00000
};
//draw points 
byte point[8]={
  B00000,
  B00000,
  B00000,
  B01110,
  B01110,
  B00000,
  B00000,
  B00000
};

// Configure the game board
byte points[maxx+1][maxy+1]; // draw point in all the board
int xpac = 0;//initial pos pacman
int ypac = 1;
int xfant = 15;//initial pos ghost
int yfant = 0;
int score = 0;
int level = 0;
byte alive = false; // when all points are eaten
byte currentGame = true;
long lastpush = 0; // last pulsation of the button
long lastghost = 0; // last movement of the ghost

// initialize the game board
void initialize(){
  lcd.clear();
  // limit the board
  for(int i=0;i<=maxx;i++){
    for(int j=0;j<=maxy;j++){
      points[i][j] = true; // initialize the game
      lcd.setCursor(i,j);
      lcd.write(2); // draw points
    }
  }
  lcd.setCursor(xpac,ypac);// place the pacman
  lcd.write(byte(0));

  lcd.setCursor(xfant,yfant);// place the ghost
  lcd.write(byte(1));
}
//if pacman wins
void winner(){
  level++; // level up
  lcd.setCursor(0,0);
  lcd.print("*You Win!*"); // Winner message
  lcd.setCursor(0,1);
  lcd.print(level,DEC); // Show the level
  delay(2000); // Wait
  initialize(); // Start a new game in the next level
}
// if pacman loose the party
void lose(){
  lcd.setCursor(0,0);
  lcd.print("****");
  lcd.setCursor(4,0);
  lcd.print("GameOver");
  lcd.setCursor(12,0);
  lcd.print("****");
  lcd.setCursor(0,1);
  lcd.print("*******");
  lcd.setCursor(7,1);
  lcd.println(score);
  lcd.setCursor(9,1);
  lcd.print("*******");
  delay(2000);
  resetFunc();
}
// Start mooving
void move(int x, int y){
  int x1 = xpac;
  int y1 = ypac;
  // ensure that players are inside the game board
  if(((xpac+x)>=0)&((xpac+x)<=maxx)){
    xpac = xpac+x;
  }
  if(((ypac+y)>=0)&((ypac+y)<=maxy)){
    ypac = ypac+y;
  }
  lcd.setCursor(xpac,ypac);//place on the new position
  lcd.write(byte(0)); //character 0(pacman)
  lcd.setCursor(x1,y1); //place on the beggining position
  if((xpac!=x1)||(ypac!=y1)){
    lcd.print(" ");// delete pacman
  }
  if(points[xpac][ypac]){
    points[xpac][ypac] = false; //eat a point
    score++;
  }
  alive = true; // check if the board is empty
  for(int i=0;i<=maxx;i++){
    for(int j=0;j<=maxy;j++){
      if(points[i][j]){
        alive = false;
      }      
    }
  }
  if(alive==true){
    winner();
  }
}
//move the ghost
void persecution(){
  int x1 = xfant;
  int y1 = yfant;

  if(yfant<ypac) yfant = yfant+1; // ghost position will change depending on pacman's movements
  else if (yfant>ypac) yfant = yfant-1;
  else if (xfant<xpac) xfant = xfant+1;
  else if (xfant>xpac) xfant = xfant-1;

  lcd.setCursor(xfant,yfant); // new place of ghost
  lcd.write(1); // place the ghost in the position of pacman
  lcd.setCursor(x1, y1); // place in the old pos

  if((x1!=xfant)||(y1!=yfant)){
    if(points[x1][y1]){
      lcd.write(2); // place by a point when is the ghost
    }else{
      lcd.print(" "); // leave it empty when is the pacman
    }
  }
  if((xpac == xfant) && (ypac==yfant)){
    lose();
  }
}
// restart the game
void reset(){
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print(" ");
  initialize();
}
// move ghost
void moveghost(){
  if((millis()-lastghost)>speed_fant/(level+1)+10){
    persecution();
    lastghost = millis();
  }
}
// configure the use of
int pushButton(){
  int a = analogRead(A0);
  if(a>1000) return btnNone;
  else if(a<50) return btnRigth;
  else if (a<180) return btnUp;
  else if (a<330) return btnDown;
  else if (a<520) return btnLeft;
  else if (a<700) return btnSelect;
}
//move pacman (with buttons)
void movepacman(){
  if((millis()-lastpush) > speed_pac){
    int k = pushButton();
    switch (k) {
    case btnNone:
      break;
    case btnLeft: // move characters to the left
      move(-1,0); // move one position 
      lastpush = millis(); //restart the count
      break;
    case btnRigth: // move characters to the right
      move(1,0); // move one position 
      lastpush = millis(); //restart the count
      break;
    case btnUp: // move characters to the right
      move(0,-1); // move one position 
      lastpush = millis(); //restart the count
      break;
    case btnDown: // move characters to the right
      move(0,1); // move one position 
      lastpush = millis(); //restart the count
      break;
    default:
      lastpush = millis();
      break;
    };
  };
}
void setup(){
  lcd.begin(16,2);
  // draw the characters
  lcd.createChar(0, pacman);
  lcd.createChar(1,ghost);
  lcd.createChar(2,point);
  // place on the first row, first column
  lcd.setCursor(0,0);
  lcd.print("Start Pacman!");
  delay(5000);// presentation board
  // start the game
  initialize();
}
void loop() {
  movepacman();
  moveghost();  
}
