# arduino.projects
//Sistema de segurança com teclado matricial, servomotor, led e buzzer ativo
#include <Keypad.h>
#include <Servo.h>
// Configuração do teclado
const byte ROWS = 4;  // Número de linhas
const byte COLS = 4;  // Número de colunas
char keys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};
byte rowPins[ROWS] = {2, 3, 4, 5};  // Pinos das linhas
byte colPins[COLS] = {6, 7, 8, 9};  // Pinos das colunas

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// Configuração dos LEDs e buzzer
const int greenLED = 10;
const int redLED = 11;
const int buzzer = 12;

// Configuração do servomotor
Servo servo;
const int servoPin = 13;  // Pino do sinal do servo
// Configuração da senha
String password = "1234";  // Senha inicial
String input = "";         // Entrada do usuário

// Variáveis para controle de tentativas
int incorrectAttempts = 0;   // Contador de tentativas incorretas
const int maxAttempts = 3;   // Número máximo de tentativas permitidas
const int lockoutTime = 10000;  // Tempo de bloqueio (10 segundos)
unsigned long lockoutStart = 0; // Tempo de início do bloqueio
bool isLocked = false;         // Indica se o sistema está bloqueado

void setup() {
  pinMode(greenLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  pinMode(buzzer, OUTPUT);
  digitalWrite(greenLED, LOW);
  digitalWrite(redLED, LOW);
  digitalWrite(buzzer, LOW);
  Serial.begin(9600);  // Para monitorar a entrada no monitor serial
  
  // Inicializa o servo
  servo.attach(servoPin);
  servo.write(0);  // Define a posição inicial (0 graus)

  Serial.begin(9600);  // Para monitorar a entrada no monitor serial
}

void loop() {
  if (isLocked) {
    handleLockout();  // Gerencia o bloqueio com alerta visual e sonoro
    return;  // Durante o bloqueio, o loop não processa entradas
  }

  char key = keypad.getKey();  // Lê a tecla pressionada

  if (key) {  // Se uma tecla foi pressionada
    Serial.println(key);

    if (key == '#') {  // Confirma a entrada
      checkPassword();
    } else if (key == '*') {  // Limpa a entrada
      input = "";
      Serial.println("Entrada limpa");
    } else {
      input += key;  // Adiciona o caractere pressionado à entrada
    }
  }
}

void checkPassword() {
  if (input == password) {  // Senha correta
    Serial.println("Senha correta!");
    digitalWrite(greenLED, HIGH);  // Acende o LED verde
    digitalWrite(redLED, LOW);
    digitalWrite(buzzer, LOW);
    delay(2000);
    moveServo();  // Chama a função para mover o servo
    digitalWrite(greenLED, LOW);
    incorrectAttempts = 0;  // Reseta o contador de tentativas incorretas
 
  } else {  // Senha incorreta
    Serial.println("Senha incorreta!");
    digitalWrite(redLED, HIGH);   // Acende o LED vermelho
    digitalWrite(buzzer, HIGH);  // Liga o alarme
    delay(2000);
    digitalWrite(redLED, LOW);
    digitalWrite(buzzer, LOW);

    incorrectAttempts++;  // Incrementa o contador de tentativas incorretas
    Serial.print("Tentativas incorretas: ");
    Serial.println(incorrectAttempts);

    if (incorrectAttempts >= maxAttempts) {  // Verifica se atingiu o limite
      Serial.println("Sistema bloqueado por 10 segundos devido a multiplas tentativas incorretas.");
      lockoutStart = millis();  // Marca o início do bloqueio
      isLocked = true;  // Ativa o bloqueio
    }
  }
  input = "";  // Reseta a entrada
}

void handleLockout() {
  unsigned long currentMillis = millis();

  // Durante o bloqueio, faz o LED vermelho piscar
  if ((currentMillis / 500) % 2 == 0) {
    digitalWrite(redLED, HIGH);
    tone(buzzer, 1000);  // Emite som no buzzer
  } else {
    digitalWrite(redLED, LOW);
    noTone(buzzer);  // Silencia o buzzer
  }

  // Verifica se o tempo de bloqueio terminou
  if (currentMillis - lockoutStart >= lockoutTime) {
    isLocked = false;  // Desbloqueia o sistema
    incorrectAttempts = 0;  // Reseta o contador de tentativas
    digitalWrite(redLED, LOW);  // Apaga o LED vermelho
    noTone(buzzer);  // Certifica-se de que o buzzer está desligado
    Serial.println("Sistema desbloqueado. Voce pode tentar novamente.");
  }
}
void moveServo() {
  Serial.println("Movendo o servo...");
  servo.write(90);  // Gira o servo para 90 graus
  delay(3000);      // Mantém o servo nessa posição por 3 segundos
  servo.write(0);   // Retorna o servo à posição inicial
  Serial.println("Servo retornou à posição inicial.");
}
