# Lab 2 - Preparação do ambiente headless/híbrido na Raspberry Pi 400

## Objetivo

Preparar a Raspberry Pi 400 para um uso híbrido entre **headless (acesso remoto via SSH)** e **console físico utilizando o teclado integrado**.

A ideia é manter a Raspberry Pi sem monitor dedicado durante o desenvolvimento, utilizando SSH pelo computador principal, mas permitindo que o teclado onboard da Pi 400 continue funcional quando necessário.

Para isso serão configurados:

* atalhos de terminal;
* ferramentas auxiliares de administração;
* visualização básica de imagens em console;
* sessão persistente compartilhada utilizando `tmux`;
* autologin no console físico;
* inicialização automática do ambiente de terminal.

## Ferramentas auxiliares

Instalação de ferramentas úteis para exploração e administração do sistema:

```bash
sudo apt update
sudo apt install lynx git tree
```

### lynx

Navegador web executado diretamente no terminal.

Útil para:

* testes de conectividade;
* acesso rápido a páginas sem interface gráfica.

Exemplo:

```bash
lynx https://www.raspberrypi.com
```

### git

Sistema de controle de versão utilizado para desenvolvimento e gerenciamento de projetos.

Verificação:

```bash
git --version
```

### tree

Ferramenta para visualizar estruturas de diretórios.

Exemplo:

```bash
tree
```

## Instalação do tmux

```bash
sudo apt update
sudo apt install tmux
```

## Habilitar autologin no console

O objetivo desta etapa é permitir que a Raspberry Pi entre automaticamente na TTY1 após o boot.

Executar:

```bash
sudo raspi-config
```

Navegar até:

```
1 System Options
    └── S6 Auto Login
```

Selecionar: ```Console Autologin```

Reiniciar:

```bash
sudo reboot
```

Após o boot, a Raspberry Pi entra automaticamente no console sem solicitar usuário e senha.


## Inicialização automática do tmux

Após habilitar o autologin, configurar o Bash para iniciar automaticamente uma sessão tmux.

Editar:

```bash
nano ~/.bash_profile
```

Adicionar:

```bash
# Carrega as configurações do Bash
if [ -f "$HOME/.bashrc" ]; then
    . "$HOME/.bashrc"
fi

# Inicia ou conecta ao tmux
if command -v tmux >/dev/null && [ -z "$TMUX" ]; then
    exec tmux new -A -s lab
fi
```

Durante a inicialização:

```text
Boot
 │
 ▼
Autologin TTY1
 │
 ▼
Bash
 │
 ▼
.bash_profile
 │
 ▼
tmux new -A -s lab
```

A sessão `lab` será criada automaticamente.

Caso ela já exista, o tmux apenas reconecta à sessão existente.

## Verificação da sessão

Após reiniciar:

```bash
tmux ls
```

Exemplo:

```text
lab: 1 windows (created Sun Jul 12 15:42:31 2026)
```

Também é possível verificar se o terminal atual está dentro do tmux:

```bash
echo $TMUX
```

## Acesso via SSH

Ao conectar via SSH, a sessão iniciada automaticamente pode ser acessada:

```bash
tmux attach -t lab
```

Caso o SSH já tenha iniciado dentro do tmux automaticamente, o comando não deve ser executado novamente.

Verificar:

```bash
tmux display-message -p '#S'
```

Retorno esperado:

```text
lab
```
## Ajuste rápido de redimensionamento do tmux

Quando a Raspberry Pi 400 é utilizada pelo console físico, a sessão `tmux` pode iniciar com um tamanho de terminal diferente do esperado. Isso ocorre porque o console local (`tty`) pode informar uma resolução menor que a utilizada pelo terminal SSH.

Para corrigir rapidamente o redimensionamento da janela do `tmux`, foi criado um alias

Adicionar ao final do arquivo:

```bash
nano ~/.bashrc
```

```bash
alias tmux-resize='tmux resize-window -A'
```

Recarregar as configurações:

```bash
source ~/.bashrc
```

Após abrir o terminal local e perceber o tamanho incorreto da janela, executar:

```bash
tmux-resize
```

O comando força o `tmux` a ajustar a janela conforme o tamanho atual do terminal.

##  Comandos básicos do tmux

Listar sessões

```bash
tmux ls
```

Desconectar mantendo a sessão ativa

```text
Ctrl+b
```

Depois: ```d```

## Benefícios do ambiente híbrido

A utilização da Raspberry Pi 400 em modo headless/híbrido permite aproveitar melhor as características únicas deste hardware.

Por possuir um teclado integrado, a Raspberry Pi 400 pode funcionar como um pequeno terminal Linux independente, permitindo intervenções locais, testes e manutenção mesmo sem monitor conectado.

Ao mesmo tempo, utilizando o acesso SSH através do computador principal, é possível manter o conforto de uma estação de trabalho completa, utilizando:

* monitor de maior resolução;
* interface gráfica do computador principal;
* múltiplas janelas de desenvolvimento;
* ferramentas de edição, documentação e pesquisa.

O uso do `tmux` une esses dois ambientes, permitindo que o mesmo terminal seja compartilhado entre o teclado físico da Raspberry Pi 400 e o acesso remoto via SSH.

Dessa forma, a Raspberry Pi mantém a simplicidade e eficiência de um sistema **Raspberry Pi OS Lite**, sem carregar recursos desnecessários de uma interface gráfica local, aproveitando ao máximo CPU, memória e armazenamento disponíveis.

A arquitetura final combina:

```text
                Computador principal
              Interface gráfica completa
                       │
                       │ SSH
                       ▼
                 ┌──────────┐
                 │  tmux    │
                 └──────────┘
                       ▲
                       │
              Teclado integrado
              Raspberry Pi 400
```

###### 12 de julho de 2026