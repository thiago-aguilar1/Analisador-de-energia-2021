# Analisador-de-energia-2021
Analisador de energia 


```c++

#include <Wire.h>
#include "RTClib.h"
#include <LiquidCrystal_I2C.h>
#include <SD.h>
#include <SPI.h>
#include "EmonLib.h" //INCLUSÃO DE BIBLIOTECA
#include <Filters.h>                      //This library does a massive work check it's .cpp file



#define ledRed = 8;
#define ledGreen = 4;
#define botaoUp = 7;
#define botaoDown = 6;
#define botaoMenu = 5;
#define botaoEnter = 3;
#define botaoCancelar = 2;
#define botaoGravar = 1;  //pois tem uma função separada só pra ele.


#define ACS_Pin A2                        //Sensor data pin on A0 analog input
#define ACS_Pin2 A3

#define VOLT_CAL 211.6 //VALOR DE CALIBRAÇÃO

EnergyMonitor emon1; //CRIA UMA INSTÂNCIA

File myFile;
 
int pinoSS = 10; // Pin 53 para Mega / Pin 10 para UNO

LiquidCrystal_I2C lcd(0x27, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE);  //ENDEREÇO DO I2C E DEMAIS INFORMAÇÕES

RTC_DS1307 rtc;







byte seta[8] = { B00000, B00100, B00010, B11111, B00010, B00100, B00000, B00000},
     quadradoOco[8] = {B11111, B10001, B10001, B10001, B10001, B10001, B10001, B11111},
     quadradoCheio[8] = {B11111, B11111, B11111, B11111, B11111, B11111, B11111, B11111},
     dia = 0,
     mes = 0, 
     ano = 0, 
     hora = 0, 
     minuto = 0, 
     segundo = 0,

     minEspera,
     minAgora,
     minAntes;



char diasDaSemana[7][12] = {"Domingo", "Segunda", "Terca", "Quarta", "Quinta", "Sexta", "Sabado"}; //Dias da semana



float supplyVoltage;
float tensao, valorTensao, corrente, valorCorrente, potencia, somaTensao, somaCorrente
      calibracaoTensao = 0.0, 
      calibracaoCorrente = 0.0;


float ACS_Value;                              //Here we keep the raw data valuess
float ACS_Value2;                              //Here we keep the raw data valuess
float testFrequency = 60;                    // test signal frequency (Hz)
float windowLength = 40.0/testFrequency;     // how long to average the signal, for statistist


float intercept = 0; // to be adjusted based on calibration testing
float slope = 0.0752; // to be adjusted based on calibration testing
                      //Please check the ACS712 Tutorial video by SurtrTech to see how to get them because it depends on your sensor, or look below

float Amps_TRMS; // estimated actual current in amps
float AmpsVerdadeiro, pote;


void setup() {
  pinMode(botaoUp, INPUT_PULLUP);
  pinMode(botaoDown, INPUT_PULLUP);
  pinMode(botaoMenu, INPUT_PULLUP);
  pinMode(botaoEnter, INPUT_PULLUP);
  pinMode(botaoCancelar, INPUT_PULLUP);
  pinMode(botaoGravar, INPUT_PULLUP);


  emon1.voltage(1, VOLT_CAL, 1.7); //PASSA PARA A FUNÇÃO OS PARÂMETROS (PINO ANALÓGIO / VALOR DE CALIBRAÇÃO / MUDANÇA DE FASE)

//****************************************************************asfassfsafsafbfag**********************************************fasfsfsafa***************
  SD.begin();  

  

   //Inicializacao do display  
  lcd.begin(16,2);  
  lcd.setBacklight(HIGH);    //LIGA O BACKLIGHT (LUZ DE FUNDO).
  lcd.createChar(1, seta);
  lcd.createChar(2, quadradoOco);
  lcd.createChar(3, quadradoCheio);


  RunningStatistics inputStats;                 // create statistics to look at the raw test signal
  inputStats.setWindowSecs( windowLength );     //Set the window length

  
  /* *****************  Apresentação inicial da equiipe no Display  ********************** */ 
  lcd.clear();
  lcd.setCursor(5, 0);
  lcd.print("AMPERE");

  
  for (int i = 0; i < 16; i++){   /*coloca os quadradinhos ocos */
    lcd.setCursor(i,1);
    lcd.write(2);
  } 


  for (int i = 0, j = 0; i <= 90 && j <= 15 ; i+= 6, j++){     
    meiaSenoide = sin( float(i) * (3.1415/180.0) );
    porcaoTempinho = 3.0 * float(meiaSenoide*255.0/2.0);
    tempinho = 3.0 + porcaoTempinho;
    delay(int(tempinho));
    lcd.setCursor(j,1);
    lcd.write(3);         /* colocando aos poucos os quadradinhos cheios */

  }



}


//****************************************************************************************************************************************************


void loop() {
  
  



  DateTime agora = rtc.now();          // Faz a leitura de dados de data e hora
  lcd.setCursor(0,0);
  lcd.print("Potenc:         ");
  lcd.setCursor(0,1);
  lcd.print("Hora:           ");
  lcd.setCursor(8,1);
  lcd.print(rtc.now().hour());      // Imprime a Hora
  lcd.print(":");                   // Imprime o texto entre aspas
  lcd.print(rtc.now().minute());    // Imprime o Minuto
  

  
  if(!(digitalRead(botaoEnter))){ 
    if(telaMomento == 1){   }
  }
  
  
  

}


//***********************************************************************************************************************************************************










/*

void ***************fafaffasdfghggsf(){
  DateTime agora = rtc.now();          // Faz a leitura de dados de data e hora
  lcd.setCursor(0,0);
  lcd.print("                ");
  lcd.setCursor(5, 0);
  lcd.print(diasDaSemana[agora.dayOfTheWeek()]);
  lcd.setCursor(0,1);
  lcd.print("Hora:           ");
  lcd.setCursor(8,1);
  lcd.print(rtc.now().hour());      // Imprime a Hora
  lcd.print(":");                   // Imprime o texto entre aspas
  lcd.print(rtc.now().minute());    // Imprime o Minuto
}

*/



//**********************************************************************************************************
//----------------------------------------------------------------------------------------------------------

void botaoCalibracao(){

  byte telaMomento = 1;  // pode ser igual a 1, 2 ou 3;
  byte posicaoSeta = 1;    // pode ser igual a 0 ou 1;
  String frasesMenu[4] = {"******MENU******", "     Tensao     ", "    Corrente    ", "   Data e Hora  " };
  
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute();
  
  while(1){

    switch(telaMomento){
      case 1: 
        lcd.setCursor(0,0);
        lcd.print(frasesMenu[0]);
        lcd.setCursor(0,1);
        lcd.print(frasesMenu[1]);
        lcd.setCursor(0,1);
        lcd.write(1);  //imprimindo a seta
        break;
      case 2: 
        lcd.setCursor(0,0);
        lcd.print(frasesMenu[1]);
        lcd.setCursor(0,1);
        lcd.print(frasesMenu[2]);
        lcd.setCursor(0,posicaoSeta);
        lcd.write(1);  //imprimindo a seta
        break;
      case 3: 
        lcd.setCursor(0,0);
        lcd.print(frasesMenu[2]);
        lcd.setCursor(0,1);
        lcd.print(frasesMenu[3]);
        lcd.setCursor(0,posicaoSeta);
        lcd.write(1);  //imprimindo a seta
        break;

    
    }
    if(!(digitalRead(botaoCancelar))){ break;}

    if(!(digitalRead(botaoUp))){
      if((telaMomento == 2)&&(posicaoSeta == 0)){ telaMomento = 1; posicaoSeta = 1;   }
      else if((telaMomento == 2)&&(posicaoSeta == 1)){ posicaoSeta = 0;   }
      else if((telaMomento == 3)&&(posicaoSeta == 0)){ telaMomento = 2; posicaoSeta = 1;   }
      else if((telaMomento == 3)&&(posicaoSeta == 1)){ posicaoSeta = 0;   }

      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();      //renova o tempo de duração
    }
    if(!(digitalRead(botaoDown))){
      if((telaMomento == 1)){ telaMomento = 2; posicaoSeta = 0;   }
      else if((telaMomento == 2)&&(posicaoSeta == 0)){ posicaoSeta = 1;   }
      else if((telaMomento == 2)&&(posicaoSeta == 1)){ telaMomento = 3; posicaoSeta = 0;   }
      else if((telaMomento == 3)&&(posicaoSeta == 0)){ posicaoSeta = 1;   }

      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();         //renova o tempo de duração
    }

    if(!(digitalRead(botaoEnter))){ 
      if(telaMomento == 1){ calibrarTensao(); break;}
      else if((telaMomento == 2)&&(posicaoSeta == 0)){ calibrarTensao(); break; }
      else if((telaMomento == 2)&&(posicaoSeta == 1)){ calibrarCorrente(); break; }
      else if((telaMomento == 3)&&(posicaoSeta == 0)){ calibrarCorrente(); break; }
      else if((telaMomento == 3)&&(posicaoSeta == 1)){ detalhesDatahora(); break; }
    
    
    break;
    }
  
    
    
    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}           //confere se ultrapassou o tempo de duração
    
    
  } 
}












//------------------------------------------------------------------------------------------------

void calibrarTensao{
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute(); 
  
  while(1){
    lcd.setCursor(0,0);
    lcd.print("     Deseja     ");
    lcd.setCursor(0,1);
    lcd.print("calibrar tensao?");

    if(!(digitalRead(botaoCancelar))){ break; }
    if(!(digitalRead(botaoEnter))){ calibrarTensao2(); break; }
    
    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}           //confere se ultrapassou o tempo de duração
  }
  
  
}



void calibrarTensao2{
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute(); 
  
  while(1){
  
  somaTensao = tensao + calibracaoTensao
    
  lcd.setCursor(0,0);
  lcd.print("Calibre a tensao");
  lcd.setCursor(0,1);
  lcd.print("                ");
  lcd.setCursor(4,1);
  lcd.print(somaTensao);
  lcd.setCursor(11,1);
  lcd.print("V");


  if(!(digitalRead(botaoCancelar))){ break; }
  if(!(digitalRead(botaoEnter))){ break; }
    
  if(!(digitalRead(botaoUp))){
    calibracaoTensao++;

    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAntes = agora.minute();        //renova o tempo de duração
  }

    
  if(!(digitalRead(botaoDown))){
    if(calibracaoTensao){
      calibracaoTensao--;  
    }
    else{
      lcd.setCursor(0,0);
      lcd.print("Calibracao tensa");
      lcd.setCursor(0,1);
      lcd.print("   esta nula    ");
      delay(1000);
    }
    
    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAntes = agora.minute();       //renova o tempo de duração
  }

    
    
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAgora = agora.minute(); 
  minEspera = abs(minAgora-minAntes);
  if(minEspera > 1){ break;}           //confere se ultrapassou o tempo de duração
  }
}















//-------------------------------------------------------------------------------------------

void calibrarCorrente{
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute(); 
  
  while(1){
    lcd.setCursor(0,0);
    lcd.print("     Deseja     ");
    lcd.setCursor(0,1);
    lcd.print("calibrar corrent");

    if(!(digitalRead(botaoCancelar))){ break; }
    if(!(digitalRead(botaoEnter))){ calibrarCorrente2(); break; }
    
    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}            //confere se ultrapassou o tempo de duração
  }

  
}



void calibrarCorrente2{
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute(); 
  
  while(1){
  
  somaCorrente = corrente + calibracaoCorrente
    
  lcd.setCursor(0,0);
  lcd.print("Calibre corrente");
  lcd.setCursor(0,1);
  lcd.print("                ");
  lcd.setCursor(4,1);
  lcd.print(somaCorrente);
  lcd.setCursor(11,1);
  lcd.print("A");


  if(!(digitalRead(botaoCancelar))){ break; }
  if(!(digitalRead(botaoEnter))){ break; }
    
  if(!(digitalRead(botaoUp))){
    calibracaoCorrente++;

    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAntes = agora.minute();        //renova o tempo de duração
  }

    
  if(!(digitalRead(botaoDown))){
    if(calibracaoCorrente){
      calibracaoCorrente--;  
    }
    else{
      lcd.setCursor(0,0);
      lcd.print("Calibrac corrent");
      lcd.setCursor(0,1);
      lcd.print("   esta nula    ");
      delay(1000);
    }
    
    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAntes = agora.minute();        //renova o tempo de duração
  }

    
    
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAgora = agora.minute();       
  minEspera = abs(minAgora-minAntes);
  if(minEspera > 1){ break;}          //confere se ultrapassou o tempo de duração
  
}



























//------------------------------------------------------------------------------------------------

void detalhesDatahora{


  byte posicaoSeta = 0;    // pode ser igual a 0 ou 1;
    
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute();
  
  while(1){
    lcd.setCursor(0,0);
    lcd.print(" Ver data e hora");
    lcd.setCursor(0,1);
    lcd.print(" calib data/hora");
    lcd.setCursor(0,posicaoSeta);
    lcd.write(1);
    
    if(!(digitalRead(botaoCancelar))){ break; }
    if(!(digitalRead(botaoEnter))){  
      if(!posicaoSeta){ verDatahora(); break; }
      else{ calibrarDatahora(); break; }
      break; 
    }

    if(!(digitalRead(botaoUp))){ 
      if(posicaoSeta){
        posicaoSeta = 0; 
      }
      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();   // Renova o tempo de duração
    }
    
    if(!(digitalRead(botaoDown))){ 
      if(!posicaoSeta){
        posicaoSeta = 1; 
      }
      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();   // Renova o tempo de duração
    }

    DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}      //confere se ultrapassou o tempo de duração
  }
  
}





//------------------------------------------------------------------

void verDatahora(){
  
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute();
  while(1){
    lcd.setCursor(0,0);
    lcd.print("                ");
    lcd.setCursor(0,1);
    lcd.print("                ");
    lcd.setCursor(5,0);
    lcd.print(diasDaSemana[agora.dayOfTheWeek()]);
    lcd.setCursor(0,1);
    lcd.print(agora.day());
    lcd.print("/");
    lcd.print(agora.month());
    lcd.print("/");
    lcd.print(agora.year());
    lcd.print(" ");
    lcd.print(agora.hour());
    lcd.print(":");
    lcd.print(agora.minute());
    lcd.setCursor(7,1);
    lcd.print(dia);

    if(!(digitalRead(botaoCancelar))){ break;}
    if(!(digitalRead(botaoEnter))){ break;}
  
    DateTime agora = rtc.now();  // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}  //confere se ultrapassou o tempo de duração
  }
}





//-------------------------------------------------------------------

void calibrarDatahora(){

  byte telaMomento = 1;  // pode ser igual a 1, 2, 3, 4, 5 ou 6;
  
  DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
  minAntes = agora.minute();
  
  dia = 01;    
  mes = 01;     
  ano = 2020;   
  hora = 00;    
  minuto = 00;   
  segundo = 00; 
  
  while(1){

    switch(telaMomento){
      case 1: 
        lcd.setCursor(0,0);
        lcd.print("   Qual o dia?  ");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(7,1);
        lcd.print(dia);
        break;
      case 2: 
        lcd.setCursor(0,0);
        lcd.print("   Qual o mes?  ");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(7,1);
        lcd.print(mes);
        break;
      case 3: 
        lcd.setCursor(0,0);
        lcd.print("   Qual o ano?  ");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(6,1);
        lcd.print(ano);
        break;
      case 4: 
        lcd.setCursor(0,0);
        lcd.print("  Qual a hora?  ");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(7,1);
        lcd.print(hora);
        break;
      case 5: 
        lcd.setCursor(0,0);
        lcd.print(" Qual o minuto? ");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(7,1);
        lcd.print(minuto);
        break;
      case 6: 
        lcd.setCursor(0,0);
        lcd.print(" Qual o segundo?");
        lcd.setCursor(0,1);
        lcd.print("                ");
        lcd.setCursor(7,1);
        lcd.print(segundo);
        break;
    }

    
    if(!(digitalRead(botaoCancelar))){ break;}

    if(!(digitalRead(botaoUp))){
      switch(telaMomento){
        case 1: dia++; break;
        case 2: mes++; break;
        case 3: ano++; break;
        case 4: hora++; break;
        case 5: minuto++; break;
        case 6: segundo++; break;
      }
      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();      //renova o tempo de duração
    }
    if(!(digitalRead(botaoDown))){
      switch(telaMomento){
        case 1: 
          if(dia!=0){dia--}; 
          break;
        case 2: 
          if(mes!=0){mes--}; 
          break;
        case 3: 
          if(ano!=0){ano--}; 
          break;
        case 4: 
          if(hora!=0){hora--}; 
          break;
        case 5: 
          if(minuto!=0){minuto--}; 
          break;
        case 6: 
          if(segundo){segundo--}; 
          break;
      }
      DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
      minAntes = agora.minute();         //renova o tempo de duração
    }

    if(!(digitalRead(botaoEnter))){
      if(telaMomento == 6){
        rtc.adjust(DateTime(ano, mes, dia, hora, minuto, segundo));//Ajusta o tempo do RTC para a data e hora definida pelo usuario.
        delay(100);
        break;
      }
      else{ 
        telaMomento++; 
        DateTime agora = rtc.now();    // Faz a leitura de dados de data e hora
        minAntes = agora.minute();         //renova o tempo de duração
      }
    }
    
    DateTime agora = rtc.now();  // Faz a leitura de dados de data e hora
    minAgora = agora.minute(); 
    minEspera = abs(minAgora-minAntes);
    if(minEspera > 1){ break;}  //confere se ultrapassou o tempo de duração 
  } 
}












void botaoGravacao(){
   lcd.setCursor(0,0);
  // lcd.print("                ");
   lcd.print("Deseja comecar  ");
   lcd.setCursor(0,1);
   lcd.print(" uma gravacao?  ");
   while(1){
    if(!(digitalRead(botaoEnter))){
      gravar();
      break; 
    }
    if(!(digitalRead(botaoCancelar))){ break;}
    delay(300);
   }
 
}



void gravar(){
  
  byte
  
  lcd.setCursor(0,0);
//lcd.print("                ");
  lcd.print("  Verificando   ");
  lcd.setCursor(0,1);
  lcd.print(" cartao micro SD");

  delay(700);
  
  
  if (SD.begin()) { // Inicializa o SD Card
    lcd.setCursor(0,0);
    lcd.print(" SD card pronto ");
    lcd.setCursor(0,1);
    lcd.print("    para uso    ");
    delay(700);
  }
  else {
    lcd.setCursor(0,0);
  //lcd.print("                ");
    lcd.print("Falha / ausencia");
    lcd.setCursor(0,1);
    lcd.print("   do SD Card   ");
    delay(700);
  return;
  }

  myFile = SD.open("usina.txt", FILE_WRITE); // Cria / Abre arquivo .txt

  String nomeGravacao;
  byte nomeDia = agora.day(),
       nomeMes = agora.month(),
       nomeAno = agora.year(),
       nomeHora = agora.hour(),
       nomeMinuto = agora.minute();
  
  nomeGravacao = String(nomeDia) + String(nomeMes) + String(nomeAno) + String(nomeHora) + String(nomeMinuto);
  
}











void leituraTensao(){

  emon1.calcVI(17,2000); //FUNÇÃO DE CÁLCULO (17 SEMICICLOS, TEMPO LIMITE PARA FAZER A MEDIÇÃO)      
  supplyVoltage   = emon1.Vrms; //VARIÁVEL RECEBE O VALOR DE TENSÃO RMS OBTIDO
  
  lcd.setCursor(0,0);
  lcd.print(" tensao:        ");
  lcd.setCursor(9,0);
  lcd.print(suplyVoltage);
  lcd.setCursor(14,0);
  lcd.print("V");
  
}



void leituraCorrente(){

    ACS_Value = analogRead(ACS_Pin);  // read the analog in value:
    inputStats.input(ACS_Value);  // log to Stats function
    Amps_TRMS = intercept + slope * inputStats.sigma();

    if (Amps_TRMS > 1.5){
      AmpsVerdadeiro = (Amps_TRMS - 0.16)/2.79;
      Serial.print( " Amps: " ); 
      Serial.println(  AmpsVerdadeiro  );
    }
    else{
      
      ACS_Value = analogRead(ACS_Pin2);  // read the analog in value:
      inputStats.input(ACS_Value2);  // log to Stats function
      Amps_TRMS = intercept + slope * inputStats.sigma();
   
      AmpsVerdadeiro = (Amps_TRMS - 0.22)/31.05;

      if (AmpsVerdadeiro > 0.01){
        Serial.print( " Amps: " ); 
        Serial.println(  AmpsVerdadeiro  );
      }
      else{
        Serial.print( " Amps: " ); 
        Serial.println( "0.0" );
      }
    }

 
    lcd.setCursor(0,1);
    lcd.print("corrente:       ");
    lcd.setCursor(10,1);
    lcd.print( AmpVerdadeiro );
    lcd.setCursor(15,1);
    lcd.print("A");  
    
    
}
  


void potencia(){

  pote = suplyVoltage * AmpVerdadeiro;

  lcd.setCursor(0,0);
  lcd.print("Pot:            ");
  lcd.setCursor(5,0);
  lcd.print(pote);
  lcd.setCursor(12,0);
  lcd.print("W");  
}





void horaMomento(){

  
  DateTime agora = rtc.now();  // Faz a leitura de dados de data e hora
  
  lcd.setCursor(0,1);
  lcd.print(" Hora:          ");
  lcd.setCursor(7,1);
  lcd.print( agora.hour() );
  lcd.print( "/" );
  lcd.print( agora.minute() );
}








```
