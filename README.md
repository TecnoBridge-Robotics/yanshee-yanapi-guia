# Guia de Referência — YanAPI (Robô Yanshee)

> Guia rápido dos principais comandos da **YanAPI** em Python, usada para
> controlar o robô educacional **Yanshee** via API RESTful local
> (`http://[IP_DO_ROBO]:9090/v1/`).
>
> Este guia cobre os comandos mais usados em sala de aula: inicialização,
> bateria, LEDs, volume, movimentos, música, voz (TTS/ASR), sensores,
> servomotores, visão computacional e controle de marcha (gait).

---

## 📦 Importando e inicializando

```python
import YanAPI

# Sempre inicialize com o IP do robô antes de usar qualquer outro comando
YanAPI.yan_api_init("192.168.1.100")
```

| Função | Parâmetros | Descrição |
|---|---|---|
| `yan_api_init(robot_ip)` | `robot_ip: str` | Inicializa o SDK apontando para o IP do robô. **Deve ser chamado primeiro.** |

---

## 🔋 Bateria

| Função | Retorno | Descrição |
|---|---|---|
| `get_robot_battery_value()` | `int` (0–100) | Retorna o percentual da bateria. Retorna `-1` em caso de erro. |
| `get_robot_battery_info()` | `dict` | Retorna informações completas: `voltage` (mV), `charging` (1=carregando / 0=não) e `percent`. |

```python
percentual = YanAPI.get_robot_battery_value()
print(f"Bateria: {percentual}%")
```

---

## 💡 LEDs (botão do peito / câmera)

| Função | Parâmetros | Descrição |
|---|---|---|
| `set_robot_led(type, color, mode)` | `type`: `button`/`camera`; `color`: `white/red/green/blue/yellow/purple/cyan` (button) ou `red/green/blue` (camera); `mode`: `on/off/blink/breath` (button) ou `on/off/blink` (camera) | Define cor e modo do LED. |
| `get_robot_led()` | — | Retorna o estado atual dos LEDs (tipo, cor, modo). |

```python
# Acende o LED do botão de peito em azul, piscando
YanAPI.set_robot_led(type="button", color="blue", mode="blink")
```

---

## 🔊 Volume

| Função | Parâmetros | Retorno |
|---|---|---|
| `get_robot_volume_value()` | — | `int`: volume atual (0–100), `-1` se falhar |
| `set_robot_volume_value(volume)` | `volume: int (0-100)` | `bool`: sucesso/falha |

```python
YanAPI.set_robot_volume_value(70)
```

---

## 🤖 Movimentos (Motion)

| Função | Parâmetros principais | Descrição |
|---|---|---|
| `get_motion_list_value()` | — | Lista todos os arquivos de movimento disponíveis (padrão + enviados pelo usuário). |
| `start_play_motion(name, direction, speed, repeat, timestamp, version)` | `name`: `reset, raise, crouch, stretch, come on, wave, bend, walk, turn around, head, bow` (ou nome customizado); `direction`: depende do `name` (ex.: `left/right/both`, `forward/backward/left/right`); `speed`: `very slow/slow/normal/fast/very fast`; `repeat`: 1–100 | Inicia a execução de um movimento (assíncrono). |
| `sync_play_motion(...)` | mesmos parâmetros | Igual ao anterior, mas **espera o movimento terminar** antes de retornar. |
| `pause_play_motion(name)` | `name` (vazio = todos) | Pausa a execução do(s) movimento(s). |
| `resume_play_motion(name)` | `name` (vazio = todos) | Retoma movimento pausado. |
| `stop_play_motion(name)` | `name` (vazio = todos) | Para a execução do(s) movimento(s). |
| `upload_motion(...)` | — | Envia um novo arquivo de movimento personalizado para o robô. |

**Tabela de `direction` conforme o `name`:**

| `name` | valores válidos de `direction` |
|---|---|
| `raise`, `stretch`, `come on`, `wave` | `left`, `right`, `both` |
| `bend`, `turn around` | `left`, `right` |
| `walk` | `forward`, `backward`, `left`, `right` |
| `head` | `forward`, `left`, `right` |

```python
# Acena com a mão direita e espera terminar
YanAPI.sync_play_motion(name="wave", direction="right", speed="normal", repeat=1)

# Anda para frente de forma assíncrona
YanAPI.start_play_motion(name="walk", direction="forward", speed="normal", repeat=3)
```

---

## 🚶 Controle de Marcha (Gait)

| Função | Parâmetros | Descrição |
|---|---|---|
| `control_motion_gait(speed_v, speed_h, steps, period, wave)` | `speed_v`: velocidade frente/trás [-5..5]; `speed_h`: velocidade lateral [-5..5]; `steps`: nº de passos (0 = quase infinito); `period`: 0–5 (0 = para); `wave`: bool (balanço dos braços) | Controla a caminhada contínua do robô (modo "gait"), ideal para controle por joystick. |
| `sync_do_motion_gait(...)` | mesmos parâmetros | Versão síncrona: retorna `True`/`False` ao concluir. |
| `get_motion_gait_state()` | — | Consulta o estado atual da marcha. |
| `exit_motion_gait()` | — | Sai do modo de marcha e retorna à posição de pé (stand). |

```python
# Anda para frente, com balanço de braços, por 20 passos
YanAPI.control_motion_gait(speed_v=3, speed_h=0, steps=20, period=2, wave=True)
YanAPI.exit_motion_gait()
```

---

## 🎵 Música

| Função | Parâmetros | Descrição |
|---|---|---|
| `get_media_music_list()` | — | Lista as músicas disponíveis (padrão + enviadas). Formatos suportados: `wav`, `mp3`. |
| `start_play_music(name)` | `name` (padrão: `SorrySorry.mp3`) | Inicia a reprodução (assíncrono). |
| `sync_play_music(name)` | `name` | Reproduz e **espera terminar**. Retorna `bool`. |
| `stop_play_music()` | — | Para a reprodução. |
| `upload_media_music(...)` | — | Envia um novo arquivo de música. |

```python
YanAPI.sync_play_music(name="SorrySorry.mp3")
```

---

## 🗣️ Voz — Síntese de Fala (TTS - Text To Speech)

| Função | Parâmetros | Descrição |
|---|---|---|
| `start_voice_tts(tts, interrupt, timestamp)` | `tts: str` (texto a falar); `interrupt: bool` (pode ser interrompido, padrão `True`) | Inicia a fala (assíncrono). |
| `sync_do_tts(tts, interrupt)` | mesmos | Fala e **espera terminar**. |
| `stop_voice_tts()` | — | Interrompe a fala em andamento. |
| `get_voice_tts_state()` | — | Consulta o estado da síntese de voz. |

```python
YanAPI.sync_do_tts(tts="Olá, eu sou o Yanshee!", interrupt=True)
```

---

## 👂 Voz — Reconhecimento de Fala (ASR)

| Função | Parâmetros | Descrição |
|---|---|---|
| `start_voice_asr(continues, timestamp)` | `continues: bool` (reconhecimento contínuo) | Inicia o reconhecimento de fala (assíncrono). |
| `sync_do_voice_asr()` | — | Executa um reconhecimento e **espera o resultado**. |
| `stop_voice_asr()` | — | Encerra o reconhecimento de fala. |
| `get_voice_asr_state()` | — | Consulta o estado do serviço de ASR. |
| `create_voice_asr_offline_syntax(...)` / `update_voice_asr_offline_syntax(...)` | — | Cria/atualiza gramáticas offline personalizadas para comandos de voz. |

```python
resultado = YanAPI.sync_do_voice_asr()
print(resultado)
```

---

## 📡 Sensores

Funções "de valor direto" (mais simples, retornam o dado já pronto):

| Função | Retorno |
|---|---|
| `get_sensors_environment_value()` | `dict` com `temperature`, `humidity`, `pressure` |
| `get_sensors_infrared_value()` | `int`: distância em mm |
| `get_sensors_ultrasonic_value()` | `int`: leitura do sensor ultrassônico |
| `get_sensors_touch_value()` | `int`: `0` nenhum toque, `1` botão 1, `2` botão 2, `3` ambos |
| `get_sensors_pressure_value()` | `int`: valor do sensor de pressão |
| `get_sensors_gyro()` | `dict`: giroscópio de 9 eixos (gyro-x/y/z por sensor) |
| `get_sensors_list_value()` | `list`: lista de sensores conectados |

```python
temp_umid = YanAPI.get_sensors_environment_value()
distancia = YanAPI.get_sensors_infrared_value()
toque = YanAPI.get_sensors_touch_value()
```

> 💡 Também existem versões "completas" (sem `_value`) como
> `get_sensors_environment()`, `get_sensors_touch()` etc., que retornam o
> dicionário bruto da API com `code`/`data`/`msg`.

---

## 🦾 Servomotores

| Função | Parâmetros | Descrição |
|---|---|---|
| `set_servos_angles(angles, runtime)` | `angles: dict {nome_servo: ângulo}`; `runtime: int` (200–4000 ms) | Move um ou mais servos para os ângulos indicados. |
| `sync_set_servo_rotate(angles, runtime)` | mesmos | Igual, mas **espera o movimento terminar**. Retorna `bool`. |
| `get_servos_angles(names)` | `names: list[str]` | Consulta o ângulo atual de um ou mais servos. |
| `get_servo_angle_value(name)` | `name: str` | Consulta o ângulo de um único servo. |
| `set_servos_mode(...)` / `get_servos_mode()` | — | Define/consulta o modo dos servos (ex.: torque ligado/desligado). |

**Nomes de servos comuns:** `RightShoulderRoll`, `RightShoulderFlex`, `RightElbowFlex`, `LeftShoulderRoll`, `LeftShoulderFlex`, `LeftElbowFlex`, `RightHipLR`, `RightHipFB`, `RightKneeFlex`, entre outros (17 servos ao todo, IDs 1 a 17). Cada servo tem uma faixa de ângulo segura (a maioria 0–180°) — consulte a documentação completa antes de mover para evitar danos.

```python
YanAPI.sync_set_servo_rotate(
    angles={"RightShoulderRoll": 90, "LeftShoulderRoll": 90},
    runtime=800
)
angulo = YanAPI.get_servo_angle_value("RightElbowFlex")
```

---

## 👁️ Visão Computacional

### Fotos

| Função | Descrição |
|---|---|
| `take_vision_photo(resolution="640x480")` | Tira uma foto (salva em `/tmp/photo`). Resolução máxima `1920x1080`. |
| `get_vision_photo_list()` | Lista as fotos tiradas. |
| `get_vision_photo(name)` / `delete_vision_photo(name)` | Obtém/apaga uma foto específica. |

### Reconhecimento Facial

| Função | Parâmetros | Descrição |
|---|---|---|
| `start_face_recognition(type)` | `type`: `tracking, recognition, quantity, age_group, gender, age, expression, mask, glass` | Inicia a análise facial do tipo escolhido (assíncrono). |
| `sync_do_face_recognition(type)` | mesmo | Executa e **espera o resultado**. |
| `stop_face_recognition(type)` | mesmo | Encerra a análise. |
| `do_face_entry(...)` | — | Cadastra um rosto para reconhecimento posterior. |

### Reconhecimento de Objetos, Cores e Gestos

| Função | Descrição |
|---|---|
| `start_object_recognition()` / `sync_do_object_recognition()` / `stop_object_recognition()` | Reconhece objetos previamente cadastrados na câmera. |
| `start_color_recognition()` / `sync_do_color_recognition()` / `stop_color_recognition()` | Reconhece cores (`black, gray, white, red, orange, yellow, green, cyan, blue, purple`). |
| `start_gesture_recognition()` / `sync_do_gesture_recognition()` / `stop_gesture_recognition()` | Reconhece gestos com a mão: `ok, good, bad, victory, heart_hand_sign, digital_6_gesture, none`. |

### QR Code

| Função | Parâmetros | Descrição |
|---|---|---|
| `start_QR_code_recognition(enableStream)` | `enableStream: bool` | Inicia a leitura de QR Code (assíncrono). |
| `sync_do_QR_code_recognition(timeOut)` | `timeOut: int` (segundos, 0 = espera indefinida) | Lê o QR Code e **espera o resultado ou timeout**. Retorna `content` com o texto lido. |
| `stop_QR_code_recognition()` | — | Encerra a leitura. |

```python
# Exemplo: reconhecimento síncrono de gesto
gesto = YanAPI.sync_do_gesture_recognition()
print(gesto["data"]["gesture"])

# Exemplo: leitura de QR Code com timeout de 10s
qr = YanAPI.sync_do_QR_code_recognition(timeOut=10)
print(qr["content"])
```

---

## 🕹️ Reconhecimento AprilTag e Rastreamento de Objetos

| Função | Descrição |
|---|---|
| `start_aprilTag_recognition()` / `stop_aprilTag_recognition()` / `get_aprilTag_recognition_status()` | Controla o reconhecimento de marcadores AprilTag. |
| `start_object_tracking()` / `stop_object_tracking()` / `config_object_tracking(...)` | Controla o rastreamento (tracking) de um objeto pela câmera. |

---

## 🎮 Joystick / Gamepad

| Função | Descrição |
|---|---|
| `get_joystick_buttons_list()` / `get_joystick_buttons_list_value()` | Lista os botões do joystick e seus estados. |
| `get_gamepad_keymap()` / `set_gamepad_keymap(...)` / `set_gamepad_keymaps(...)` | Consulta/define o mapeamento de teclas do gamepad. |
| `reset_gamepad_keymap(...)` / `reset_gamepad_keymaps(...)` | Restaura o mapeamento padrão. |

---

## 🌐 Outras funções úteis

| Função | Descrição |
|---|---|
| `get_robot_version_info()` / `get_robot_version_info_value()` | Retorna a versão de hardware/software do robô. |
| `get_robot_mode()` | Retorna o modo de operação atual do robô. |
| `get_robot_language()` / `set_robot_language(language)` | Consulta/define o idioma (`'zh'` chinês, `'en'` inglês). |
| `get_robot_fall_management_state()` / `set_robot_fall_management_state(enable)` | Consulta/ativa a proteção contra quedas. |
| `sensor_calibration()` | Executa a calibração dos sensores. |

---

## 📋 Boas práticas para os alunos

1. **Sempre inicialize** com `YanAPI.yan_api_init("IP_DO_ROBO")` antes de qualquer outro comando.
2. Prefira as funções **`sync_...`** (ex.: `sync_play_motion`, `sync_do_tts`, `sync_do_voice_asr`) quando quiser que o script **espere** a ação terminar antes de continuar — ótimo para scripts sequenciais e didáticos.
3. Use as funções **`start_...`/`stop_...`** quando precisar de comportamento assíncrono (ex.: tocar música enquanto o robô caminha).
4. Sempre verifique o campo `"code"` no retorno: **`0` significa sucesso**; qualquer outro valor indica erro (consulte `"msg"` para detalhes).
5. Ao mover servos manualmente, **respeite os limites de ângulo de cada ID** para evitar danos físicos ao robô.
6. Use `get_..._list()` (motion, música, fotos) antes de tentar tocar/usar um recurso, para confirmar que ele existe no robô.

---

## 🧪 Exemplo completo: rotina simples

```python
import YanAPI

# 1. Conectar ao robô
YanAPI.yan_api_init("192.168.1.100")

# 2. Verificar bateria
print("Bateria:", YanAPI.get_robot_battery_value(), "%")

# 3. Cumprimentar com voz e movimento
YanAPI.sync_do_tts(tts="Olá! Eu sou o Yanshee, muito prazer!")
YanAPI.sync_play_motion(name="wave", direction="right")

# 4. Tirar uma foto
foto = YanAPI.take_vision_photo()
print("Foto salva:", foto["data"]["name"])

# 5. Reconhecer um gesto do usuário
resultado = YanAPI.sync_do_gesture_recognition()
print("Gesto detectado:", resultado["data"]["gesture"])

# 6. Encerrar com uma dancinha
YanAPI.sync_play_motion(name="bow", direction="left")
```

---

*Guia gerado a partir do arquivo `YanAPI.py` (SDK Python do robô Yanshee). Para
detalhes completos de todos os parâmetros e formatos de retorno, consulte os
docstrings de cada função diretamente no código-fonte.*
