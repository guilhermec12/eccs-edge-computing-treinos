# 03 · ECCS — Edge Computing aplicado aos treinos

Monitoramento das condições ambientais de um local de treino com ESP32, sensor DHT22 e display OLED SSD1306, simulado no Wokwi e integrado ao ThingSpeak.

## Links da entrega

- [Simulação no Wokwi](https://wokwi.com/projects/476175482957977601)
- [Canal público do ThingSpeak e gráficos](https://thingspeak.mathworks.com/channels/3509600)
- [Repositório do projeto](https://github.com/guilhermec12/eccs-edge-computing-treinos)

## Funcionamento

O DHT22 mede temperatura e umidade a cada 2 segundos. O ESP32 mantém uma janela circular com as últimas 5 leituras válidas e calcula as médias localmente. Durante as primeiras leituras, a média usa apenas as amostras já recebidas.

O estado ALERTA ocorre quando a temperatura atual ou média ultrapassa 30 °C, ou quando a umidade atual ou média ultrapassa 70%. Caso contrário, o estado é NORMAL. A verificação do valor atual permite reação imediata; a média mantém a indicação enquanto ainda há medições elevadas na janela.

O OLED exibe valores atuais, médias, quantidade de amostras, estado e conexão Wi-Fi. Leituras inválidas são descartadas e o envio é suspenso até uma nova leitura válida. O ESP32 tenta reconectar o Wi-Fi e continua o processamento local. O envio à nuvem acontece em tarefa separada, a cada 20 segundos, sem bloquear as leituras.

## ESP32 versus nuvem

| No ESP32 (borda) | No ThingSpeak (nuvem) |
|---|---|
| Leitura do DHT22 | Recebimento dos cinco campos |
| Validação das medições | Armazenamento do histórico |
| Média móvel das últimas 5 leituras | Gráficos públicos ao longo do tempo |
| Decisão de alerta | Consulta remota dos resultados |
| Atualização do OLED | Comparação visual entre valores e médias |

O cálculo e a decisão não dependem da internet. A nuvem recebe resultados já processados. Não há armazenamento persistente nem reenvio das amostras perdidas durante uma desconexão.

## Relação com os treinos

O acompanhamento de temperatura e umidade ajuda a observar mudanças no ambiente de treino e discutir ventilação, horários e organização das atividades. Os gráficos permitem comparar períodos; o display fornece informação no próprio local. Os limites de 30 °C e 70% são parâmetros didáticos do projeto, não critérios médicos de segurança ou autorização para treinar. A simulação representa condições ambientais, não medições reais de atletas.

## Componentes e ligações

| Componente | Pino | ESP32 |
|---|---|---|
| DHT22 | VCC / GND | 3V3 / GND |
| DHT22 | SDA (dados) | GPIO 15 |
| OLED SSD1306 | VCC / GND | 3V3 / GND |
| OLED SSD1306 | SDA | GPIO 21 |
| OLED SSD1306 | SCL | GPIO 22 |

OLED: 128 × 64 pixels, endereço I2C 0x3C. Rede virtual: Wokwi-GUEST, sem senha.

## Arquivos

- sketch.ino: programa do ESP32.
- diagram.json: componentes e conexões do Wokwi.
- libraries.txt: bibliotecas usadas pela simulação.
- README.md: funcionamento, execução e testes.

## Como executar

1. Abra o link do Wokwi e clique em ▶.
2. Aguarde as leituras e a conexão Wi-Fi. O OLED funciona mesmo sem configurar o ThingSpeak.
3. No seu canal ThingSpeak, abra API Keys e copie a Write API Key.
4. Cole a chave no campo de entrada do monitor serial do Wokwi e pressione Enter. A mensagem “Chave recebida em RAM” confirma a configuração.
5. Aguarde até 20 segundos e procure “ThingSpeak: registro confirmado”. Abra o canal público para acompanhar os gráficos.
6. Clique no DHT22 para alterar temperatura e umidade.

A chave fica apenas na memória durante a execução e precisa ser informada novamente após reiniciar. Ela não deve ser colocada no código público ou no GitHub. O código usa HTTP no ambiente didático; uma implantação real deve usar HTTPS com validação de certificado. O destino é identificado pela chave de escrita, portanto não é necessário informar o número do canal no programa.

## Campos do ThingSpeak

| Campo | Conteúdo |
|---|---|
| 1 | Temperatura atual (°C) |
| 2 | Umidade atual (%) |
| 3 | Média das últimas 5 temperaturas (°C) |
| 4 | Média das últimas 5 umidades (%) |
| 5 | Estado: 0 = normal; 1 = alerta |

## Testes e evidências

Na validação de 25/09/2026, a simulação compilou e conectou à rede Wokwi-GUEST. Foram observadas leituras de 25 °C e 50%, médias iguais a esses valores e estado NORMAL. Ao elevar a temperatura para 80 °C, o OLED e o serial indicaram ALERTA e a média mudou gradualmente conforme a janela de 5 amostras.

O envio pelo ESP32 foi confirmado no serial (registro 17). A consulta pública do canal confirmou o registro 18, às 23:44:28 UTC, contendo 80 °C, 50%, médias de 80 °C e 50%, e alerta 1. O valor de 80 °C foi usado como teste extremo do simulador e não representa uma condição real de treino. A simulação foi interrompida após a validação; novos dados só chegam quando ela está em execução com a chave configurada.

Roteiro para apresentar: iniciar em 25 °C/50%; elevar a temperatura para 35 °C; observar o alerta e a média; retornar a 25 °C e aguardar a janela se renovar; elevar somente a umidade para 80%; conferir os registros e gráficos. Este roteiro adicional pode ser repetido durante a apresentação.
