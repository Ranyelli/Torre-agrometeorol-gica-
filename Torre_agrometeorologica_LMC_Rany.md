// Torre agrometeorológia-nível 1
// Conexão I2C com identificação dos ip's de cada sensor... de acordo com o programa de rastreio de https://github.com/Regisonn/LMC_Sensor_Joice
//Composição: 3 IR MLX90614 (conexão I2C), 1 termohigrometro DHT22, 1 barometro BMP280, 1 clock DS1307, 1 microsd
// Autor: Ranyelli
//Data de criação: 20/08/2021
//Última atualização: 16/09/2021 17:30:00 UTM -4
//Programa foi criado com base nas especicações do fabricante do sensor e baseado no armazenamento de dados de acordo com https://github.com/Regisonn/LMC_Sensor_Joice
//--------------------------------------------

//Bibliotecas

#include <Adafruit_Sensor.h>
#include <i2cmaster.h>
#include <SdFat.h>
#include <Wire.h>
#include <Adafruit_MLX90614.h>
#include <DHT.h>
#include <RTC.h>
#include <SPI.h>
#include <Adafruit_BMP280.h>
//------------------------------------------------

//definindo portas de ligação (SPI) do BMP280
#define BMP_SCK  (13)
#define BMP_MISO (12)
#define BMP_MOSI (11)
#define BMP_CS   (10)
//definido o tipo de termohigrometro
#define DHTTYPE DHT22 //DEFINE O MODELO DO SENSOR (DHT22 / AM2302)

//chamando as bibliotecas

static DS1307 RTC;

Adafruit_BMP280 bmp(BMP_CS, BMP_MOSI, BMP_MISO,  BMP_SCK);

//após passar pelo scanner de I2C seleciona-se os canais de cada IR

Adafruit_MLX90614 mlx1 = Adafruit_MLX90614(0x51); //0x51 is adress device 1
Adafruit_MLX90614 mlx2 = Adafruit_MLX90614(0x52); //0x52 is adress device 2
Adafruit_MLX90614 mlx3 = Adafruit_MLX90614(0x53); //0x53 is adress device 3
//porta digital 28 a escolha fica ao critério do programador
DHT dht(28, DHTTYPE);
//chamando o sd

SdFat sdCard;
SdFile meuArquivo;

//declarando o pino cs do sd (ligação SPI)
//sdk=52 MOSI=51 MISO=50  CS=53 
const int chipSelect = 53;

//amostragem e armazenamento dass variáveis
void setup()
{
  Serial.begin(9600);
  mlx1.begin();
  mlx2.begin();
  mlx3.begin();
  bmp.begin();
  
  //o pino 7 para alimentação do termohigrometro
  pinMode(7,OUTPUT); //pino 
  digitalWrite(7,HIGH);
  dht.begin();

  
 \\o led azul mostra se os sensores estão funcionando perfeitamente
  \\ a cada medida o led vermelho pisca
  pinMode(4,OUTPUT); //define o pino de saída para ascender o VERMELHO 
  pinMode(5,OUTPUT);
  
  pinMode(9,OUTPUT); //define o pine de saída para ascender o LED 
  pinMode(8,OUTPUT);

  digitalWrite(4,LOW);
  digitalWrite(9,LOW);

    
  bmp.setSampling(Adafruit_BMP280::MODE_NORMAL,     /* Operating Mode. */
                    Adafruit_BMP280::SAMPLING_X2,     /* Temp. oversampling */
                    Adafruit_BMP280::SAMPLING_X16,    /* Pressure oversampling */
                    Adafruit_BMP280::FILTER_X16,      /* Filtering. */
                    Adafruit_BMP280::STANDBY_MS_500);   /* Pressure oversampling */
  
  
  
     // Inicializa o sd
  if(!sdCard.begin(chipSelect,SPI_HALF_SPEED))sdCard.initErrorHalt();
  // Abre o arquivo LER.TXT TESTE_Rany_M
  if (!meuArquivo.open("TESTE_Rany_N1.dat", O_RDWR | O_CREAT | O_AT_END))
 //nome da planilha de acordo com o local de instalção
  {
    sdCard.errorHalt("Erro na abertura do arquivo LER_temp.TXT!");
    
    //digitalWrite(8,LOW); //Ascende o led para avisar erro de leitura
  }
RTC.begin();

  //Serial.print("Is Clock Running: ");
  //if (RTC.isRunning())
  //1{
    //Serial.println("Yes");
 
    /*if (RTC.getHourMode() == CLOCK_H12)
    {
      switch (RTC.getMeridiem()) {
      case HOUR_AM:
        Serial.print(" AM");
        break;
      case HOUR_PM:
        Serial.print(" PM");
        break;
      }
    }
    Serial.println("");
    delay(1000);
  }
  else
  {
    delay(1500);

    Serial.println("No");
    Serial.println("Setting Time");
    //RTC.setHourMode(CLOCK_H12);
    RTC.setHourMode(CLOCK_H24);
    RTC.setDateTime(__DATE__, __TIME__);
    Serial.println("New Time Set");
    Serial.print(__DATE__);
    Serial.print(" ");
    Serial.println(__TIME__);
    RTC.startClock();*/
  //}
  {
  //cabeçalho da tabela  
  meuArquivo.println("Ranyelli Experiment - Torre Agrometeorologica nivel 1 ");
  meuArquivo.println("Station: Teste  "); //Nome do local de instalação
  //meuArquivo.println("time,T_ar1,T_obj1,T_ar2,T_obj2,T_ar3,T_obj3,Temp,RH");
  meuArquivo.println("Date,time,T_ar1,T_obj1,T_ar2,T_obj2,T_ar3,T_obj3,Temp,RH,T_bar, Press, Alt");
 }
}
//******Criação do loop*******
void loop()
{
  // ________________Leitura do Sensor___________
  //t = rtc.getTime().;

  //Usado para piscar o LED toda vez que for efetuada uma leitura. #caso o LED fique acesso, ocorreu algum problema
  digitalWrite(5,HIGH);
  digitalWrite(8,HIGH);
  delay(60);
  digitalWrite(8,LOW);
  
  //****Exibição do painel serial***   (usado para visualizar os dados pelo computador )
  Serial.print("Amb1 = ");
  Serial.print(mlx1.readAmbientTempC()); 
  Serial.print("ºC\tObj1 = "); 
  Serial.print(mlx1.readObjectTempC()); 

  Serial.print("   Amb2 = ");
  Serial.print(mlx2.readAmbientTempC()); 
  Serial.print("ºC\tObj2 = "); 
  Serial.print(mlx2.readObjectTempC()); 

  Serial.print("    Amb3 = ");
  Serial.print(mlx3.readAmbientTempC()); 
  Serial.print("ºC\tObj3 = "); 
  Serial.print(mlx3.readObjectTempC()); 
  
  //Serial.print("    Amb4 = ");
  //Serial.print(mlx3.readAmbientTempC()); 
  //Serial.print("ºC\tObj4 = "); 
  //Serial.print(mlx3.readObjectTempC()); 

  Serial.print("    t_ar = ");
  Serial.print(dht.readTemperature()); 
  Serial.print("ºC H% = "); 
  Serial.print(dht.readHumidity()); 
  

  Serial.print(" t_bar = ");
  Serial.print(bmp.readTemperature());
  Serial.print(" press = ");
  Serial.print(bmp.readPressure());
  Serial.print(" alt = ");
  Serial.print(bmp.readAltitude(1013.25));
  Serial.print("---- "); 

  Serial.print(RTC.getDay());
  Serial.print("-");
  Serial.print(RTC.getMonth());
  Serial.print("-");
  Serial.print(RTC.getYear());
  Serial.print(" --");
  Serial.print(RTC.getHours());
  Serial.print(":");
  Serial.print(RTC.getMinutes());
  Serial.print(":");
  Serial.print(RTC.getSeconds());
  Serial.print("--");

  
  //declaração de variaveis
  float TA1=(mlx1.readAmbientTempC());  
  float TO1=(mlx1.readObjectTempC());

  float TA2=(mlx2.readAmbientTempC());
  float TO2=(mlx2.readObjectTempC());
  
  float TA3=(mlx3.readAmbientTempC());
  float TO3=(mlx3.readObjectTempC());

  

  //Exemplo
  //float TA4=(mlx4.readAmbientTempC(),4); // numero 4 indica a quantidade de casas decimais
  //float TO4=(mlx4.readObjectTempC(),4);
  
  //float t = (hdc1080.readTemperature());
  //float h = (hdc1080.readHumidity());
  float h = (dht.readHumidity());
  float t = (dht.readTemperature());
  
  //float H=(distanceSensor.measureDistanceCm());
  // Grava dados do sensor em LER_Temp_test.TXT
 
  float tt = (bmp.readTemperature());
  float p = (bmp.readPressure());
  float al =(bmp.readAltitude());
  
  //Escrever na tabela (data,T_ar, T_obj)             //(data, dia, mes, ano, hora, T_ar, T_obj)

  meuArquivo.print(RTC.getDay(), DEC);
  meuArquivo.print('/');
  meuArquivo.print(RTC.getMonth(), DEC);
  meuArquivo.print('/');
  meuArquivo.print(RTC.getYear(), DEC);
  meuArquivo.print(", ");
  meuArquivo.print(RTC.getHours(), DEC);
  meuArquivo.print(':');
  meuArquivo.print(RTC.getMinutes(), DEC);
  meuArquivo.print(':');
  meuArquivo.print(RTC.getSeconds(), DEC);
  meuArquivo.print(",");
   
 
  

  //dados sensor de temperatura (T_ar  T_obj)
  //meuArquivo.print(",");
  meuArquivo.print(TA1);
  meuArquivo.print(",");
  meuArquivo.print(TO1);

  meuArquivo.print(",");
  meuArquivo.print(TA2);
  meuArquivo.print(",");
  meuArquivo.print(TO2);

  meuArquivo.print(",");
  meuArquivo.print(TA3);
  meuArquivo.print(",");
  meuArquivo.print(TO3);
  
  
  //exemplo
  //meuArquivo.print(",");
  //meuArquivo.print(TA4,4);
  //meuArquivo.print(",");
  //meuArquivo.print(TO4,4);

  meuArquivo.print(",");
  meuArquivo.print(t);
  meuArquivo.print(",");
  meuArquivo.print(h);

  meuArquivo.print(",");
  meuArquivo.print(tt);
  meuArquivo.print(",");
  meuArquivo.print(p);
  meuArquivo.print(",");
  meuArquivo.print(al);
  
  meuArquivo.println(); //quebra de linha 
  meuArquivo.close();
  if(!sdCard.begin(chipSelect,SPI_HALF_SPEED))sdCard.initErrorHalt();
  
  // Abre o arquivo LER_Temp.TXT
  if (!meuArquivo.open("TESTE_Rany_N1.dat", O_RDWR | O_CREAT | O_AT_END));  //abri o arquivo novamente para escrever a leitura do proximo loop
   

  delay(5000); //define o intervalo de leitura 1000 = 1 segundo 
}



