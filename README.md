# ⚡ raios-ffx

**Desvie dos 200 raios das Planícies dos Trovões, em Final Fantasy X, sem precisar apertar nada.**

Um script pequeno para Linux que observa a tela do **Chiaki** (o app de Remote Play de PlayStation) e aperta **X** sozinho no exato momento em que o raio clareia a tela. Você deixa o personagem parado, vai tomar um café, e volta com a **Venus Sigil** (o selo da arma celestial da Lulu) esperando por você.

> Feito e testado com **Final Fantasy X HD (PlayStation 4)** jogado via **Remote Play no chiaki-ng**, em Linux.

---

## 📖 Sumário

- [O desafio](#-o-desafio)
- [Como o script resolve](#-como-o-script-resolve)
- [Onde funciona](#-onde-funciona)
- [O que você precisa](#-o-que-você-precisa)
- [Instalação](#-instalação)
- [Como usar](#-como-usar)
- [Dicas e ajustes](#-dicas-e-ajustes)
- [Problemas comuns](#-problemas-comuns)
- [Perguntas frequentes](#-perguntas-frequentes)
- [Licença](#-licença)

---

## 🎯 O desafio

Nas **Planícies dos Trovões** (*Thunder Plains*), raios caem o tempo todo. Logo antes de cada raio, a tela pisca em branco. Se você apertar **X** nesse instante, o personagem dá um pulinho e desvia.

Para ganhar a **Venus Sigil**, que completa a arma celestial da Lulu (*Onion Knight*), é preciso desviar de **200 raios seguidos**. São uns bons minutos de concentração total, olhando para a tela, sem errar nenhuma vez. Se levar um raio, a contagem volta para zero.

É o tipo de tarefa repetitiva que um computador faz melhor que a gente.

## 🧠 Como o script resolve

A ideia é simples:

```
 ┌──────────────┐     ~60x por segundo      ┌─────────────────────┐
 │ Janela do    │ ────────────────────────▶ │ Mede o brilho médio │
 │ Chiaki       │                           │ do meio da tela     │
 └──────────────┘                           └──────────┬──────────┘
                                                       │
                       brilho deu um salto repentino?  │  (ex.: 60 → 200)
                                                       ▼
 ┌──────────────┐                           ┌─────────────────────┐
 │ PlayStation  │ ◀──── Chiaki envia ────── │ Aperta X num        │
 │ (desvia! ⚡) │                           │ controle virtual    │
 └──────────────┘                           └─────────────────────┘
```

1. **Olha a tela.** O script encontra a janela do Chiaki e, cerca de 60 vezes por segundo, mede o quão clara está a parte central dela.
2. **Aprende o "normal".** As Planícies são escuras. O script acompanha o brilho normal da cena, que muda devagar conforme a câmera se mexe.
3. **Percebe o raio.** Quando o brilho dá um salto repentino e grande, é o clarão do raio.
4. **Aperta X.** O script cria um **controle virtual** (via `uinput`, recurso do próprio Linux) e aperta X nele. O Chiaki junta os botões de todos os controles conectados e manda para o console. Por isso o seu controle de verdade continua funcionando normalmente ao mesmo tempo.
5. **Espera um pouco.** Depois de cada raio, fica pouco mais de 1 segundo sem apertar, para não apertar duas vezes no mesmo clarão.

Nada é instalado no console e nada é modificado no jogo. O script só faz o que você faria: olhar a tela e apertar um botão na hora certa.

## 🕹️ Onde funciona

| Situação | Funciona? |
|---|---|
| **PS4 + Chiaki (chiaki-ng) no Linux** | ✅ Sim, foi onde ele foi feito e testado |
| **PS5 + Chiaki no Linux** | 🟡 Deve funcionar. O Chiaki também conecta ao PS5 do mesmo jeito, mas ainda não foi testado |
| Versão de PC (Steam) | ❓ Não testado. Este projeto, por enquanto, é só para PlayStation via Chiaki |
| Windows ou macOS | ❌ Não. O script usa recursos específicos do Linux (`uinput` e X11) |
| Linux com Wayland | ⚠️ Provavelmente não. A captura de tela usa X11; em Wayland pode vir tudo preto |

## 🧰 O que você precisa

- **Linux** usando **X11** (o padrão em Linux Mint, Ubuntu com sessão "Xorg" etc.)
- **Chiaki** ou **chiaki-ng** já conectado ao seu PlayStation (Flatpak ou nativo, tanto faz)
- **Final Fantasy X** (ou FFX/X-2 HD Remaster) aberto no console
- Python 3 e dois pacotes:
  - `python3-xlib`: para ler a tela
  - `wmctrl`: para achar a janela do Chiaki
- Permissão para criar controles virtuais (`/dev/uinput`). Veja abaixo.

## 📦 Instalação

**1. Instale as dependências** (Debian, Ubuntu, Linux Mint):

```bash
sudo apt install python3-xlib wmctrl
```

No Fedora: `sudo dnf install python3-xlib wmctrl`. No Arch: `sudo pacman -S python-xlib wmctrl`.

**2. Baixe o script e deixe ele executável:**

```bash
mkdir -p ~/.local/bin
curl -o ~/.local/bin/raios-ffx https://raw.githubusercontent.com/KakaXIV/raios-ffx/main/raios-ffx
chmod +x ~/.local/bin/raios-ffx
```

> Ou clique em **Code → Download ZIP** aqui no GitHub, extraia e copie o arquivo `raios-ffx` para `~/.local/bin`.

**3. Confira se você pode criar controles virtuais:**

```bash
test -w /dev/uinput && echo "Tudo certo" || echo "Precisa liberar o uinput"
```

Se aparecer **"Precisa liberar o uinput"**, rode os comandos abaixo e depois **reinicie o computador**:

```bash
echo 'KERNEL=="uinput", SUBSYSTEM=="misc", TAG+="uaccess", OPTIONS+="static_node=uinput"' | sudo tee /etc/udev/rules.d/60-uinput-usuario.rules
sudo udevadm control --reload && sudo udevadm trigger
```

> Se você tem a Steam instalada, provavelmente isso já está liberado, porque ela usa o mesmo recurso.

## ▶️ Como usar

1. Abra o **Chiaki** e conecte ao seu PlayStation. Deixe a janela **visível**: não minimize e não cubra o meio dela com outra janela.
2. No jogo, vá até as **Planícies dos Trovões** e pare o personagem em **campo aberto**, longe das torres de para-raios.
3. Abra um terminal e rode:

   ```bash
   raios-ffx
   ```

4. Pronto. A cada raio, aparece uma linha assim:

   ```
   Rodando! Deixe o personagem parado nas Planícies dos Trovões. Ctrl+C para parar.
   ⚡ raio 1  (brilho 60 -> 203)
   ⚡ raio 2  (brilho 59 -> 203)
   ```

5. Nos **primeiros raios**, olhe a tela e confira se o personagem está **pulando para desviar**.
6. Passou de **200**? Deixe cair mais uns 10 a 15 por segurança, aperte **Ctrl+C** no terminal e vá direto, sem levar raio, até o **baú do lado de fora da Agência de Viagem da Rin** (no meio das Planícies, entre a parte norte e a sul). A Venus Sigil está nele.

> ⚠️ **Desligue o script (Ctrl+C) antes de sair das Planícies.** Qualquer clarão na tela, como o começo de uma batalha, faz ele apertar X.

## 🔧 Dicas e ajustes

**Modo teste.** Mostra o brilho da tela em tempo real sem apertar nada. Bom para conferir se o script está "enxergando" o Chiaki:

```bash
raios-ffx --teste
```

**Atraso.** Se o personagem estiver levando raio porque apertou cedo demais (raro), dá para esperar alguns milissegundos depois do clarão:

```bash
raios-ffx --atraso 50
```

**Sensibilidade.** No começo do arquivo há dois valores que você pode mudar:

| Valor | O que faz | Padrão |
|---|---|---|
| `SALTO` | Quanto o brilho precisa subir de repente para contar como raio. Aumente se ele estiver apertando sem raio; diminua se ele estiver perdendo raios. | `55` |
| `ESPERA` | Segundos sem apertar de novo depois de um raio | `1.2` |

**Conexão.** Quanto mais estável a conexão entre o computador e o console, melhor. Cabo de rede no console ajuda bastante.

## 🩹 Problemas comuns

**"Não achei a janela do Chiaki, esperando..."**
O script procura uma janela com "chiaki" no título. Confira se o Chiaki está aberto e se o `wmctrl -l` mostra a janela dele.

**Os raios caem, mas nada aparece no terminal**
Rode `raios-ffx --teste`. Se o brilho ficar sempre em `0`, sua sessão provavelmente é Wayland. Entre na sessão "Xorg/X11" na tela de login.

**Aparece "⚡ raio", mas o personagem não pula**
- Clique uma vez na janela do Chiaki e confira se o seu controle está funcionando nela.
- Feche e abra o Chiaki **com o script já rodando**, para ele reconhecer o controle virtual.
- Confira a permissão do `/dev/uinput` (passo 3 da instalação).

**`PermissionError` com `/dev/uinput`**
Falta a permissão do passo 3 da instalação.

## ❓ Perguntas frequentes

**Isso é trapaça? Posso ser banido?**
O script não modifica o jogo, o console nem a rede. Ele só olha a tela e aperta um botão, como uma pessoa faria. FFX é um jogo de um jogador só, sem modo online. Mesmo assim, use por sua conta e risco.

**Por que o controle virtual se identifica como um adaptador USB genérico?**
O Chiaki usa a biblioteca SDL para entender os controles. Ela já conhece esse adaptador e sabe qual botão é o X. Assim o script funciona sem nenhuma configuração extra.

**Funciona com outros jogos?**
O princípio (reagir a um clarão na tela apertando um botão) serve para outros minigames de reflexo. O código, porém, foi feito e ajustado para as Planícies dos Trovões.

---

## English summary

**raios-ffx** auto-dodges the lightning bolts in Final Fantasy X's Thunder Plains (needed for Lulu's Venus Sigil) when playing a PS4 through **Chiaki / chiaki-ng Remote Play on Linux (X11)**. It watches the Chiaki window, detects the sudden screen flash, and presses ✕ on a virtual `uinput` gamepad that Chiaki merges with your real controller. Requirements: `python3-xlib`, `wmctrl`, write access to `/dev/uinput`. Run `raios-ffx` (or `raios-ffx --teste` to just print brightness). PS5 should work the same way but is untested.

## 📄 Licença

[MIT](LICENSE): use, modifique e compartilhe à vontade.
