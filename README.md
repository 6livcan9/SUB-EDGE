# SUB-EDGE
READ ME

Otho Oliveira Candido - RM557054

Projeto BlueShield O projeto BlueShield é uma solução desenvolvida para
o desafio \"Blue Future\" da FIAP, com o objetivo de monitorar e
registrar a temperatura da água e a incidência solar, contribuindo para
a proteção dos oceanos. Este projeto foi desenvolvido individualmente
por alunos que não entregaram o projeto anterior dentro do prazo
regulamentar.

Objetivos

Desenvolver um sistema funcional de monitoramento e registro utilizando
sensores NTC e LDR. Implementar um sistema de alerta visual e sonoro
para indicar anomalias na temperatura da água. Configurar a memória
EEPROM para armazenamento eficiente dos dados registrados. Simular e
validar o projeto na plataforma Wokwi.

Especificações do Sistema

Sensores Utilizados Sensor de Temperatura (NTC): Sensor analógico para
medir a temperatura da água. Sensor de Luminosidade (LDR): Sensor para
medir a incidência de luz solar.

Condições de Registro O sistema deve registrar na memória EEPROM o valor
da temperatura média (ºC) e a luminosidade média (em uma escala de 0 a
100%) todas as vezes que a temperatura da água exceder 23ºC. Os valores
médios são o resultado de 30 leituras ao longo de 1 minuto, ou seja, uma
leitura a cada 2 segundos. Utiliza-se média simples para o cálculo.

Indicação de Luminosidade O sistema inclui um gráfico de barras LED que
mostra o nível da luminosidade (0 a 100%).

Detecção de Anomalias Ao detectar valores de temperatura acima de 23ºC,
o sistema ativa um LED vermelho e emite um beep para alertar sobre a
anomalia. A anomalia é registrada na EEPROM imediatamente após a
detecção.

Memória EEPROM Configurada para armazenar até 900 registros.

Display de LCD 16x2 O display apresenta o valor medido de temperatura e
luminosidade, além de apresentar o logo gráfico do BlueShield.

Simulação Utilizar a plataforma Wokwi para simular o funcionamento do
sistema.

Componentes: Arduino UNO Sensor de Temperatura (NTC) Sensor de
Luminosidade (LDR) LED Vermelho Buzzer Display LCD 16x2 com interface
I2C Barra de LEDs (10 LEDs) EEPROM

Código Fonte

#include \<EEPROM.h\> #include \<Wire.h\> #include
\<LiquidCrystal_I2C.h\>

#define TEMP_SENSOR_PIN A0 #define LDR_SENSOR_PIN A1 #define LED_RED_PIN
3 #define BEEP_PIN 2 #define NUM_LEDS 10 #define EEPROM_SIZE 900

LiquidCrystal_I2C lcd(0x27, 16, 2); int ledPins\[NUM_LEDS\] = {13, 12,
11, 10, 9, 8, 7, 6, 5, 4}; float temperatureSum = 0; int lightSum = 0;
int readCount = 0; int eepromIndex = 0;

void setup() { Serial.begin(115200); pinMode(LED_RED_PIN, OUTPUT);
pinMode(BEEP_PIN, OUTPUT);

for (int i = 0; i \< NUM_LEDS; i++) { pinMode(ledPins\[i\], OUTPUT); }

lcd.init(); lcd.backlight(); lcd.setCursor(0, 0);
lcd.print(\"BlueShield\"); lcd.setCursor(0, 1);
lcd.print(\"Inicializando\...\"); delay(2000); lcd.clear(); }

void loop() { float temperature = readTemperature(); int light =
analogRead(LDR_SENSOR_PIN);

Serial.print(\"Temp: \"); Serial.println(temperature);
Serial.print(\"Light: \"); Serial.println(light);

temperatureSum += temperature; lightSum += light; readCount++;

if (readCount == 30) { float avgTemp = temperatureSum / 30.0; float
avgLight = (lightSum / 30.0) / 10.23; // Normaliza a leitura do LDR para
uma escala de 0 a 100%

Serial.print(\"Average Temp: \"); Serial.println(avgTemp);
Serial.print(\"Average Light: \"); Serial.println(avgLight);

if (avgTemp \> 23.0) { Serial.println(\"Anomaly detected: Temperature \>
23C\"); digitalWrite(LED_RED_PIN, HIGH); // Acende o LED vermelho
tone(BEEP_PIN, 1000); // Emite um beep delay(500); noTone(BEEP_PIN);
storeData(avgTemp, avgLight); // Armazena os dados na EEPROM } else {
digitalWrite(LED_RED_PIN, LOW); // Desliga o LED vermelho
noTone(BEEP_PIN); }

updateDisplay(avgTemp, avgLight); // Atualiza o display LCD com os
valores médios updateLedBar(avgLight); // Atualiza a barra de LEDs com o
nível de luminosidade

// Reseta os somatórios e a contagem de leituras temperatureSum = 0;
lightSum = 0; readCount = 0; }

delay(2000); // Aguarda 2 segundos antes da próxima leitura }

float readTemperature() { int analogValue = analogRead(TEMP_SENSOR_PIN);
float voltage = analogValue \* (5.0 / 1023.0); float temperature =
(voltage - 0.5) \* 100.0; return temperature; }

void storeData(float temp, float light) { if (eepromIndex + 4 \>
EEPROM_SIZE) { eepromIndex = 0; }

EEPROM.put(eepromIndex, temp); EEPROM.put(eepromIndex + 2, light);
eepromIndex += 4; }

void updateDisplay(float temp, float light) { lcd.clear();
lcd.setCursor(0, 0); lcd.print(\"Temp: \"); lcd.print(temp);
lcd.print(\" C\"); lcd.setCursor(0, 1); lcd.print(\"Luz: \");
lcd.print(light); lcd.print(\" %\"); }

void updateLedBar(float light) { int numLedsOn = (light / 100.0) \*
NUM_LEDS;

for (int i = 0; i \< NUM_LEDS; i++) { if (i \< numLedsOn) {
digitalWrite(ledPins\[i\], HIGH); } else { digitalWrite(ledPins\[i\],
LOW); } } }
