# Psychtoolbox 3 — Guia de Instalação (Ubuntu + MATLAB)

Este guia descreve um fluxo **simples, reprodutível e seguro** para instalar a versão mais recente do **Psychtoolbox‑3** no Ubuntu usando **MATLAB** (válido também para Octave com pequenas adaptações). O foco é evitar erros comuns (como `savepath` em diretórios protegidos) e garantir que os **MEX** sejam corretamente compilados.

> **Resumo do fluxo**: Preparar dependências → Obter PTB → Configurar `startup.m` (sem `savepath`) → Rodar `SetupPsychtoolbox` → Verificar.

---

## 1) Pré‑requisitos

### 1.1. Sistema e MATLAB

* Ubuntu 22.04/24.04 (ou compatível).
* MATLAB recente (R2022a+).
* Acesso a um usuário com privilégios de `sudo` para instalar pacotes e ajustar permissões de hardware quando solicitado.

### 1.2. Pacotes do sistema (recomendados)

Instale os pacotes multimídia e de vídeo que o PTB utiliza para I/O, áudio/vídeo e sincronização de tela:

```bash
sudo apt update && sudo apt install -y \
  gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools gstreamer1.0-x \
  libasound2 libasound2-plugins portaudio19-dev \
  libgl1-mesa-glx libgl1-mesa-dri libglu1-mesa \
  libdc1394-25 libusb-1.0-0 \
  libxkbcommon-x11-0 libxcb1 libxrandr2 libxinerama1 libxcursor1 libxi6
```

> Dica: Em sistemas NVIDIA proprietários, mantenha o driver estável e atualizado. Em Wayland, o PTB funciona, mas em alguns setups o **Xorg** ainda oferece timing mais previsível.

---

## 2) Baixar o Psychtoolbox

Você pode baixar o **ZIP** ou **clonar** o repositório. Para um setup simples por usuário, salve em sua pasta `~/Documents/MATLAB/`.

### Opção A — ZIP (recomendado para iniciantes)

1. Acesse: [http://psychtoolbox.org/](http://psychtoolbox.org/) → **Download** (GitHub) → **Download ZIP**.
2. Extraia o conteúdo para: `~/Documents/MATLAB/Psychtoolbox`

### Opção B — Git (útil para atualizações)

```bash
mkdir -p ~/Documents/MATLAB
cd ~/Documents/MATLAB
git clone https://github.com/Psychtoolbox-3/Psychtoolbox-3.git Psychtoolbox
```

> **Não** copie arquivos para pastas do sistema (ex.: `/usr/local/MATLAB/...`). Mantenha tudo no seu `$HOME` para evitar problemas de permissão.

---

## 3) Configure o `startup.m` (sem `savepath`)

Crie (ou edite) o arquivo `startup.m` no diretório de inicialização do MATLAB **do seu usuário**.

```bash
cd ~/Documents/MATLAB
nano startup.m
```

Conteúdo recomendado:

```matlab
% Adiciona o Psychtoolbox e subpastas ao path a cada inicialização
addpath(genpath(fullfile(getenv('HOME'), 'Documents', 'MATLAB', 'Psychtoolbox')));

disp('Psychtoolbox path carregado via startup.m');
```

> **Por que não usar `savepath`?**
> Em instalações padrão, `savepath` tenta gravar em `pathdef.m` dentro do diretório do MATLAB (protegido). Isso gera avisos/erros e **não é necessário**. O `startup.m` já garante o path em cada sessão.

---

## 4) Instalação/compilação com `SetupPsychtoolbox`

Abra o **MATLAB**, e rode:

```matlab
cd(fullfile(getenv('HOME'),'Documents','MATLAB','Psychtoolbox'))
SetupPsychtoolbox
```

O script irá:

* Verificar dependências do sistema.
* **Compilar MEX** necessários (ponte MATLAB ↔ bibliotecas nativas).
* Solicitar sua senha (`sudo`) se precisar ajustar permissões de acesso a hardware (p.ex., DRM/placa de vídeo), importantes para **timing**.

> Se aparecer diálogos de licença, aceite pelo MATLAB ou rode: `PsychLicense('AcceptLicense')`.

---

## 5) Verificação pós‑instalação

No MATLAB, execute:

```matlab
% Verifica versão detectada
PsychtoolboxVersion

% Rotina de pós-instalação (opcional)
PsychtoolboxPostInstallRoutine

% Teste de sincronização (ideal sem janelas/monitores extras)
PerceptualVBLSyncTest
```

* O teste de sincronização deve rodar **sem** avisos críticos.
* Se você **precisar** pular os SyncTests temporariamente (ex.: em VMs), use **apenas para diagnóstico**:

  ```matlab
  Screen('Preference','SkipSyncTests', 1);
  ```

  e retorne a `0` assim que possível.

---

## 6) Atualizar ou desinstalar

### Atualizar (ZIP → substituição)

1. Baixe o ZIP mais recente.
2. **Feche o MATLAB**.
3. Substitua o diretório `~/Documents/MATLAB/Psychtoolbox` pelo novo conteúdo.
4. Reabra o MATLAB e rode `SetupPsychtoolbox` novamente.

### Atualizar (Git)

```bash
cd ~/Documents/MATLAB/Psychtoolbox
git pull
```

Depois, no MATLAB: `SetupPsychtoolbox`.

### Desinstalar

* Remova a linha do `addpath` do seu `startup.m`.
* (Opcional) Exclua a pasta `~/Documents/MATLAB/Psychtoolbox`.

---

## 7) Solução de problemas (FAQ)

**Q1. Vejo aviso: “Unable to save path to file '.../pathdef.m'”.**
*A:* É normal se você tentou `savepath`. Use **apenas** o `startup.m` no seu `$HOME`. Isso evita a escrita em diretórios protegidos.

**Q2. MATLAB não abre após editar variáveis de Java (ex.: `MATLAB_JAVA`)**
*A:* **Não** defina `MATLAB_JAVA` manualmente para escalonamento/HiDPI. Prefira:

* Iniciar o MATLAB com `-scaling` (versões novas) **ou** ajustar escalas via sistema/DE.
* Se já quebrou a inicialização, remova as variáveis personalizadas do shell (ex.: `~/.bashrc`, `~/.profile`).

**Q3. Wayland vs Xorg: Sync falhando/instável**
*A:* Em alguns drivers/setups, o **Xorg** oferece melhor previsibilidade de **VBLSync**. Faça login em sessão Xorg e re‑teste.

**Q4. HDMI vs DP, 120/144/180 Hz**
*A:* Use cabos/portas que suportem sua taxa de atualização. O DP geralmente oferece maior banda e estabilidade em monitores de alta taxa de atualização.

**Q5. Como confirmar que os MEX foram compilados?**
*A:* Rode `PsychtoolboxVersion` e observe mensagens do `SetupPsychtoolbox`. Verifique se funções como `Screen` executam sem erros.

**Q6. Mensagem de licença**
*A:* No MATLAB:

```matlab
PsychLicense('AcceptLicense')
```

**Q7. Octave**
*A:* O fluxo é similar. Garanta que as dependências do sistema estejam presentes, e rode os scripts equivalentes no Octave.

---

## 8) Boas práticas e dicas

* **Um PTB por usuário**: Evite instalações globais; minimize conflitos de permissão.
* **Ambiente limpo**: Feche apps/monitores extras ao rodar testes de sincronização.
* **Drivers de GPU**: Mantenha o driver estável (NVIDIA/Intel/AMD) e coerente com seu servidor gráfico.
* **Monitores**: Prefira conexão que entregue RGB 4:4:4 **Full** no desktop; evite modos YCbCr limitados que podem afetar legibilidade.

---

## 9) Exemplo de `startup.m` portátil

Se pretende compartilhar entre máquinas/usuários, use caminhos relativos ao `$HOME`:

```matlab
function startup
  ptbdir = fullfile(getenv('HOME'),'Documents','MATLAB','Psychtoolbox');
  if exist(ptbdir,'dir')
    addpath(genpath(ptbdir));
    disp(['PTB em: ', ptbdir]);
  else
    warning('Psychtoolbox não encontrado em %s', ptbdir);
  end
end
```

---

## 10) Verificação rápida (checklist)

* [ ] Dependências instaladas via `apt`.
* [ ] Pasta `~/Documents/MATLAB/Psychtoolbox` presente.
* [ ] `startup.m` com `addpath(genpath(...))` (sem `savepath`).
* [ ] `SetupPsychtoolbox` executado sem erros.
* [ ] `PerceptualVBLSyncTest` roda sem avisos críticos.
* [ ] (Opcional) `PsychtoolboxPostInstallRoutine` concluído.

---

### Licença

Psychtoolbox é software de código aberto. Verifique e aceite a licença via:

```matlab
PsychLicense('AcceptLicense')
```

---

**Pronto!** Com isso, seu PTB deve estar instalado, no path a cada sessão, e com MEX funcionando. Se precisar, abra issues no seu repositório apontando logs do `SetupPsychtoolbox` e do `PerceptualVBLSyncTest` para diagnóstico rápido.
