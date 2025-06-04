### GS_PhysicalComputing
---

##  `README.md`



**Sistema de Alerta por Gesto de Emergência com MediaPipe Pose**

---

##  Problema

Durante eventos climáticos extremos, como chuvas intensas ou vendavais, falhas de energia elétrica podem ocorrer, afetando seriamente a visibilidade e a comunicação, principalmente em ambientes como **hospitais, abrigos ou residências de pessoas com mobilidade reduzida**.

Nesses cenários, muitas pessoas não conseguem se comunicar verbalmente ou acessar sistemas eletrônicos convencionais para pedir ajuda.

---

##  Solução

Este projeto propõe um **sistema de reconhecimento de gestos** com **visão computacional**, baseado na biblioteca **MediaPipe Pose**, que detecta um padrão específico de movimento de socorro:

* O usuário **levanta uma das mãos acima da linha dos ombros**.
* Move a mão horizontalmente **de um lado para o outro (vai-volta)** por **3 vezes completas** (total de 6 oscilações).
* Ao detectar o padrão corretamente, um **alerta visual e sonoro** é disparado.

Esse sistema pode funcionar mesmo **sem conexão com a internet** e pode ser alimentado por **energia auxiliar (geradores ou no-breaks)** em casos de apagão.

---

##  Objetivo

* Criar uma ferramenta acessível e automática para pedir ajuda em **apagões ou situações emergenciais**.
* Priorizar pessoas com limitações físicas ou visuais, oferecendo um gesto **natural e fácil de executar**.
* Garantir que o sistema funcione com **alto grau de confiabilidade**, mesmo em baixa luz ou sem áudio.

---

##  Tecnologias Utilizadas

* **Python 3.8+**
* **OpenCV**
* **MediaPipe Pose**
* **Numpy**
* **time**

---

##  Como Executar

1. Instale as dependências:

```bash
pip install opencv-python mediapipe numpy
```

2. Execute o programa:

```bash
python alerta_gesto_pose.py
```

3. Pressione **`q`** para sair do sistema.

---

## Comportamento do Sistema

* Desenha uma **linha azul** horizontal indicando a altura dos ombros.
* Rastrea a mão visível (direita ou esquerda) e detecta movimentos alternados **acima da linha azul**.
* Se o padrão de vai-volta for executado corretamente **3 vezes completas**, o **alerta é ativado** por 10 segundos.
* Um efeito visual (borda vermelha piscando) e uma mensagem são exibidos.
* O contador mostra o progresso dos movimentos registrados.
---

##  Observações Importantes

* A detecção exige **iluminação suficiente** e que a **câmera esteja posicionada corretamente**.
* Para uso em ambientes críticos, o sistema deve estar conectado a uma **fonte de energia reserva**.
* Pode ser expandido com integração a alarmes sonoros, envio de SMS, integração com painéis de controle hospitalares, etc.

---

##  Código (`gs_iot.py`)

```python
import cv2
import mediapipe as mp
import time
from collections import deque
import os

# Configurações
FRAME_WIDTH = 320
FRAME_HEIGHT = 240
PROCESS_EVERY = 2
GESTURE_WINDOW = 20  # Número de posições para verificar padrão de gesto

# Inicialização do MediaPipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(
    max_num_hands=1,
    min_detection_confidence=0.6,
    min_tracking_confidence=0.6
)

# Armazena posições da mão no eixo X (em pixels)
x_positions = deque(maxlen=GESTURE_WINDOW)

def detect_x_motion(x_positions):
    """
    Detecta se houve um gesto em X com base nas mudanças bruscas de direção no eixo X.
    Retorna True se detectado padrão de vai-volta-vai ou volta-vai-volta.
    """
    if len(x_positions) < GESTURE_WINDOW:
        return False

    direction_changes = 0
    for i in range(2, len(x_positions)):
        prev_diff = x_positions[i-1] - x_positions[i-2]
        curr_diff = x_positions[i] - x_positions[i-1]
        if prev_diff * curr_diff < 0:  # mudança de direção (troca de sinal)
            direction_changes += 1

    return direction_changes >= 3  # mais robusto

def log_alert():
    with open("alert_logs.txt", "a") as f:
        f.write(f"Alerta detectado em {time.strftime('%Y-%m-%d %H:%M:%S')}\n")

def main():
    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        print("Erro: Não foi possível acessar a câmera.")
        return

    cap.set(cv2.CAP_PROP_FRAME_WIDTH, FRAME_WIDTH)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, FRAME_HEIGHT)

    frame_count = 0
    gesture_detected = False
    last_alert_time = 0

    try:
        while True:
            frame_count += 1
            if frame_count % PROCESS_EVERY != 0:
                cap.grab()
                continue

            ret, frame = cap.read()
            if not ret:
                continue

            frame = cv2.flip(frame, 1)
            rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            results = hands.process(rgb)

            if results.multi_hand_landmarks:
                hand_landmarks = results.multi_hand_landmarks[0]
                frame_width = frame.shape[1]
                x_pixel = int(hand_landmarks.landmark[9].x * frame_width)
                x_positions.append(x_pixel)

                if detect_x_motion(x_positions):
                    current_time = time.time()
                    if current_time - last_alert_time > 3:
                        gesture_detected = True
                        last_alert_time = current_time
                        log_alert()
                else:
                    gesture_detected = False

            # Exibe mensagem na tela
            if gesture_detected:
                cv2.putText(frame, "GESTO DE EMERGENCIA DETECTADO!", (10, 40),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)
            else:
                cv2.putText(frame, "Aguardando gesto...", (10, 40),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (200, 200, 200), 1)

            cv2.imshow("Gesto de Emergencia", frame)
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break
    finally:
        cap.release()
        cv2.destroyAllWindows()

if __name__ == "__main__":
    main()
```
---
Video Pitch:
https://youtu.be/_1F7okvuNBo
---
##Nome e rm dos integrantes
---
Adriano Lopes rm98574
---
Henrique de Brito rm
---
Rodrigo Aparecido rm
