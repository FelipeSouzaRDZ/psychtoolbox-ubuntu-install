# Como instalar o Psychtoolbox no Ubuntu 24.04 com MATLAB R2025a

Este repositório explica como instalar o Psychtoolbox no Ubuntu, lidar com permissões, ativação do MATLAB e resolução de problemas comuns (como erros de display, XServer, etc).

## Pré-requisitos
- Ubuntu 24.04
- MATLAB instalado (versão testada: R2025a)
- Acesso root
---

## ✅ Passo a passo para configurar o Psychtoolbox no Ubuntu 24.04

### 1️⃣ Instalar o Psychtoolbox (via NeuroDebian ou GitHub)

* **Via NeuroDebian** (usamos pacotes da versão Jammy com `trusted=yes`)
  Adicionar o repositório temporariamente:

  ```bash
  echo "deb [trusted=yes] http://neuro.debian.net/debian jammy main contrib non-free" | sudo tee /etc/apt/sources.list.d/neurodebian.list
  sudo apt update
  sudo apt install matlab-psychtoolbox-3
  sudo rm /etc/apt/sources.list.d/neurodebian.list
  sudo apt update
  ```

* **OU Via GitHub** (se preferir código-fonte)

  ```bash
  git clone https://github.com/Psychtoolbox-3/Psychtoolbox-3.git
  ```

---

### 2️⃣ Rodar o script de configuração do sistema

*(Essencial para permissões, acesso à GPU e realtime scheduling)*

Criar o script `PsychLinuxConfiguration.sh` com este conteúdo:

```bash
#!/bin/bash
echo "Adding user '$SUDO_USER' to group 'video'..."
usermod -a -G video "$SUDO_USER"

echo "Adding user '$SUDO_USER' to group 'realtime'..."
groupadd -f realtime
usermod -a -G realtime "$SUDO_USER"

echo "Setting real-time scheduling limits..."
cat <<EOF >/etc/security/limits.d/99-psychtoolbox.conf
@realtime   -  rtprio     99
@realtime   -  memlock    unlimited
EOF

echo "Installing udev rule for GPU access..."
cat <<EOF >/etc/udev/rules.d/60-psychtoolbox.rules
KERNEL=="nvidiactl", GROUP="video"
KERNEL=="nvidia0", GROUP="video"
KERNEL=="nvidia-modeset", GROUP="video"
KERNEL=="nvidia-uvm", GROUP="video"
KERNEL=="nvidia-uvm-tools", GROUP="video"
EOF

echo "Done! Please reboot."
```

Rodar:

```bash
sudo bash PsychLinuxConfiguration.sh
```

⚠️ **Reinicie o computador após rodar!**

---

### 3️⃣ Instalar o pacote GLUT necessário:

```bash
sudo apt install libglut-dev
sudo ln -s /usr/lib/x86_64-linux-gnu/libglut.so /usr/lib/x86_64-linux-gnu/libglut.so.3
```

---

### 4️⃣ Rodar em ambiente X11 (não Wayland)

Verificar:

```bash
echo $XDG_SESSION_TYPE
```

Se for `wayland`, fazer login usando **Ubuntu on Xorg**

---

### 5️⃣ Testar o script MATLAB básico (com debug):

```matlab
Screen('Preference', 'SkipSyncTests', 1);
Screen('Preference', 'ConserveVRAM', 4096);
Screen('Preference', 'VisualDebugLevel', 4);

PsychDefaultSetup(2);

screens = Screen('Screens');
screenNumber = max(screens);
white = WhiteIndex(screenNumber);
grey = white / 2;

smallWindow = [100 100 500 500];
[window, windowRect] = PsychImaging('OpenWindow', screenNumber, grey, smallWindow);

Screen('TextSize', window, 30);
DrawFormattedText(window, 'Teste OK - Pressione qualquer tecla', 'center', 'center', white);
Screen('Flip', window);

KbStrokeWait;
sca;
```

---

## 🛠️ Recursos Úteis

* [Documentação Psychtoolbox](http://psychtoolbox.org/docs/Psychtoolbox)

---

## Colaboradores

Autor: Felipe Rodrigues Souza