# ArduinoCP1


```markdown
# Projeto de Monitoramento de Luminosidade com Alarme Arduino

## 🌟 Visão Geral

Este projeto demonstra um sistema simples e eficaz de monitoramento da intensidade da luz ambiente utilizando uma placa Arduino. O sistema utiliza um sensor de luminosidade para medir a luz e fornece feedback visual através de LEDs e um display LCD, além de um alerta sonoro (buzzer) em condições de luminosidade excessiva. É uma aplicação prática de eletrônica e programação para criar um sistema de alerta automatizado.

## 🛠️ Componentes Utilizados

* **Placa Arduino:** O microcontrolador que processa os dados do sensor e controla os periféricos.
* **Sensor de Luminosidade (LDR ou similar):** Responsável por medir a intensidade da luz ambiente.
* **Display LCD 16x2:** Exibe o status da luminosidade ("Luz: OK", "Luz: Alerta", "ALERTA! Luz Alta", "Desligar a Luz!") e uma mensagem de boas-vindas.
* **LEDs (Verde, Amarelo, Vermelho):** Fornecem feedback visual rápido sobre o nível de luminosidade.
    * **Verde:** Luminosidade dentro da faixa ideal.
    * **Amarelo:** Nível de alerta de luminosidade.
    * **Vermelho:** Luminosidade excessiva, indicando alarme.
* **Buzzer:** Emite um sinal sonoro quando a luminosidade atinge um nível crítico.
* **Resistores:** Utilizados no circuito do sensor de luminosidade e dos LEDs para limitar a corrente.
* **Fios de Conexão (Jumpers):** Para interligar os componentes na placa Arduino.
* **Opcional:** Potenciômetro para ajuste do contraste do LCD.

## ⚙️ Funcionalidades

O sistema implementa as seguintes funcionalidades:

* **Monitoramento Contínuo:** Leitura constante da intensidade da luz ambiente através do sensor.
* **Indicação Visual de Status:**
    * LED verde acende quando a luminosidade está abaixo do limiar de alerta.
    * LED amarelo acende quando a luminosidade está entre o limiar de alerta e o limiar de desligamento.
    * LED vermelho acende e pisca quando a luminosidade ultrapassa o limiar de desligamento.
* **Alertas no Display LCD:** Exibição de mensagens claras sobre o status da luminosidade.
* **Alarme Sonoro:** Emissão de um bipe através do buzzer quando a luminosidade é excessiva, com duração de alguns segundos.
* **Mensagem de Pós-Alarme:** Após o alarme sonoro, o LCD exibe uma mensagem lembrando de desligar a luz.

## 🔌 Diagrama de Conexão (Fritzing ou similar recomendado)

```
![image](https://github.com/user-attachments/assets/e78c3a28-05a9-4400-9974-e8db9c9f559a)


Exemplo de descrição das conexões:

* Sensor de Luminosidade:
    * Um terminal conectado ao pino analógico A0 do Arduino.
    * Outro terminal conectado a um resistor (ex: 10kΩ) que vai para o GND.
    * A junção entre o sensor e o resistor conectada ao +5V do Arduino.
* Display LCD:
    * RS ao pino digital 12 do Arduino.
    * EN ao pino digital 11 do Arduino.
    * D4 ao pino digital 5 do Arduino.
    * D5 ao pino digital 4 do Arduino.
    * D6 ao pino digital 3 do Arduino.
    * D7 ao pino digital 2 do Arduino.
    * Pino do Backlight (se houver) ao pino digital 10 do Arduino (para controle on/off). Conectar ao +5V ou GND diretamente se não for controlar.
    * VSS ao GND.
    * VDD ao +5V.
    * V0 (Contraste) ao pino central de um potenciômetro (opcional), com os outros terminais do potenciômetro conectados ao +5V e GND.
* LEDs:
    * Ânodo do LED Verde ao pino digital 6 do Arduino através de um resistor (ex: 220Ω).
    * Ânodo do LED Amarelo ao pino digital 7 do Arduino através de um resistor (ex: 220Ω).
    * Ânodo do LED Vermelho ao pino digital 8 do Arduino através de um resistor (ex: 220Ω).
    * Cátodo de todos os LEDs ao GND.
* Buzzer:
    * Terminal positivo ao pino digital 9 do Arduino.
    * Terminal negativo ao GND.
```

## 🚀 Como Utilizar

1.  **Clone este repositório (o link do código está neste README).**
2.  **Conecte os componentes** à placa Arduino seguindo o diagrama de conexão.
3.  **Copie o código-fonte (`.ino`)** para o seu ambiente de desenvolvimento Arduino IDE.
4.  **Verifique e carregue o código** para a placa Arduino.
5.  **Ajuste os limiares de luminosidade** (`luminosidadeAlerta` e `luminosidadeDesligar`) no código conforme a sensibilidade desejada.
6.  **Observe o sistema em funcionamento** ao variar a intensidade da luz sobre o sensor.

## ⚙️ Código Fonte

```arduino
#include <LiquidCrystal.h>

// Pinos do LCD (ajuste conforme SUAS conexões)
const int rs = 12;
const int en = 11;
const int backlightPin = 10; // Pino do backlight
const int d4 = 5;
const int d5 = 4;
const int d6 = 3;
const int d7 = 2;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7); // Inicialização sem pino de backlight

// Pino do sensor de luminosidade
const int sensorLuzPin = A0;

// Pinos dos LEDs
const int ledVerdePin = 6;
const int ledAmareloPin = 7;
const int ledVermelhoPin = 8;

// Pino do Buzzer
const int buzzerPin = 9;

// Limiares de luminosidade
const int luminosidadeAlerta = 200;
const int luminosidadeDesligar = 400;

// Variável para armazenar o valor da luminosidade
int valorLuminosidade;

// Variável para controlar o estado do alarme
bool alarmeAtivo = false;
unsigned long tempoInicioAlarme;
const long duracaoAlarme = 3000; // 3 segundos

void setup() {
  lcd.begin(16, 2);
  lcd.setCursor(0, 1);
  lcd.print("Vinheria Agnello");
  lcd.setCursor(0, 0); // Prepara a primeira linha para o status da luz

  #ifdef backlightPin
    pinMode(backlightPin, OUTPUT);
    digitalWrite(backlightPin, HIGH); // Liga a luz de fundo
    lcd.backlight(); // Garante que o backlight seja ligado (para algumas bibliotecas)
  #else
    lcd.backlight(); // Tenta ligar o backlight (pode não funcionar sem o pino definido)
  #endif

  // Define os pinos dos LEDs como saída
  pinMode(ledVerdePin, OUTPUT);
  pinMode(ledAmareloPin, OUTPUT);
  pinMode(ledVermelhoPin, OUTPUT);

  // Define o pino do buzzer como saída
  pinMode(buzzerPin, OUTPUT);

  // Inicializa os LEDs e o buzzer apagados/desligados
  digitalWrite(ledVerdePin, LOW);
  digitalWrite(ledAmareloPin, LOW);
  digitalWrite(ledVermelhoPin, LOW);
  digitalWrite(buzzerPin, LOW);

  Serial.begin(9600);
}

void loop() {
  // Lê o valor do sensor de luminosidade
  valorLuminosidade = analogRead(sensorLuzPin);
  Serial.print("Luminosidade: ");
  Serial.println(valorLuminosidade);

  // Controle dos LEDs e exibição do status da luz na primeira linha
  lcd.setCursor(0, 0);
  if (valorLuminosidade < luminosidadeAlerta) {
    digitalWrite(ledVerdePin, HIGH);
    digitalWrite(ledAmareloPin, LOW);
    digitalWrite(ledVermelhoPin, LOW);
    alarmeAtivo = false;
    digitalWrite(buzzerPin, LOW);
    lcd.print("Luz: OK          "); // Limpa a linha e exibe o status
  } else if (valorLuminosidade < luminosidadeDesligar) {
    digitalWrite(ledVerdePin, LOW);
    digitalWrite(ledAmareloPin, HIGH);
    digitalWrite(ledVermelhoPin, LOW);
    alarmeAtivo = false;
    digitalWrite(buzzerPin, LOW);
    lcd.print("Luz: Alerta      "); // Limpa a linha e exibe o status
  } else {
    digitalWrite(ledVerdePin, LOW);
    digitalWrite(ledAmareloPin, LOW);
    digitalWrite(ledVermelhoPin, HIGH);

    // Ativa o alarme se ainda não estiver ativo
    if (!alarmeAtivo) {
      alarmeAtivo = true;
      tempoInicioAlarme = millis(); // Registra o tempo de início do alarme
      lcd.print("ALERTA! Luz Alta");
      digitalWrite(buzzerPin, HIGH); // Liga o buzzer
    }
  }

  // Desativa o alarme após 3 segundos e exibe a mensagem na primeira linha
  if (alarmeAtivo && (millis() - tempoInicioAlarme >= duracaoAlarme)) {
    digitalWrite(buzzerPin, LOW);
    alarmeAtivo = false;
    lcd.print("Desligar a Luz! ");
  }

  delay(100);
}
```

## 📝 Notas e Melhorias Futuras

* **Calibração do Sensor:** Os valores do sensor de luminosidade podem variar. Uma calibração mais precisa poderia ser implementada.
* **Ajuste Dinâmico dos Limiares:** Permitir o ajuste dos limiares de alerta e desligamento através de botões ou uma interface serial.
* **Integração com Outros Sensores:** Adicionar sensores de temperatura ou umidade para um monitoramento ambiental mais completo.
* **Interface Gráfica:** Desenvolver uma interface gráfica (via computador ou aplicativo) para visualizar os dados e configurar o sistema.
* **Persistência de Dados:** Armazenar os dados de luminosidade ao longo do tempo para análise.

## 🧑‍💻 Autor

**Christoffer dos Santos Alves**

## 📄 Link para o projeto e vídeo:

(https://wokwi.com/projects/428079835404257281)

https://youtu.be/2vjYUb9TBU8?si=jzAGpWAQh9th5RaZ

---
![image](https://github.com/user-attachments/assets/66efdd3a-e6b5-4e5b-8a58-ba90faeff775)

```

