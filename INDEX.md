# Combo Late Game - Documentação

> Sistema completo de partidas competitivas para FiveM com votação de modos e mapas, sistema de salas e duelos entre times.

## 📚 Índice da Documentação

### 🚀 Começando
- [Apresentação](DOCUMENTACAO.md) - Visão geral do sistema
- [Instalação](INSTALACAO.md) - Guia de instalação para diferentes frameworks
- [Configuração](CONFIGURACAO.md) - Guia completo de configuração

### 🌐 Hospedagem da Documentação
- [Início Rápido](INICIO_RAPIDO.md) - Deploy rápido com GitHub Desktop
- [Deploy com GitHub Desktop](DEPLOY_GITHUB_DESKTOP.md) - Guia completo para GitHub Desktop
- [Deploy (Git CLI)](DEPLOY.md) - Guia para linha de comando

### 🔧 Integração
- [API e Integração](API.md) - Documentação completa da API para desenvolvedores

## 📖 Visão Rápida

### O que é o Combo Late Game?

O **Combo Late Game** é um sistema de partidas competitivas para servidores FiveM que permite:

- ✅ Criação de salas e times
- ✅ Sistema de desafios entre salas
- ✅ Votação de modos de jogo e mapas
- ✅ Partidas em formato melhor de N rounds
- ✅ Sistema de espectação
- ✅ Routing buckets isolados
- ✅ Integração com vRP (adaptável para outros frameworks)

### Funcionalidades

| Funcionalidade | Descrição |
|--------------|-----------|
| **Sistema de Salas** | Crie salas, convide jogadores, gerencie membros |
| **Desafios** | Desafie outras salas para partidas competitivas |
| **Votação** | Vote no modo de jogo e mapa antes da partida |
| **Modos** | Pistols, Rifles, Submachine, MixedGuns (configurável) |
| **Mapas** | Mirage, Dust, Italy, ClockTower (configurável) |
| **Safezone** | Zona que fecha progressivamente durante a partida |
| **Espectação** | Assista partidas em andamento |
| **Routing Buckets** | Partidas isoladas sem interferência externa |

### Frameworks Suportados

- ✅ **vRP** (suporte nativo)
- ✅ **vRP 2.0** (suporte nativo)
- 🔄 **QBCore** (requer adaptação)
- 🔄 **ESX** (requer adaptação)
- 🔄 **Outros** (requer adaptação)

## 🚀 Instalação Rápida

### 1. Download e Extração

```bash
resources/
└── [combo_lategame]/
    └── combo_lategame/
```

### 2. Adicionar ao server.cfg

```cfg
ensure vrp
ensure combo_lategame
```

### 3. Configurar (Opcional)

```cfg
set combo_lategame_vote_mode_sec 120
set combo_lategame_vote_map_sec 120
```

### 4. Adaptar Framework (Se necessário)

Se não usa vRP, adapte as funções em:
- `config/server/functions.lua`
- `config/client/functions.lua`
- `config/config.lua`

Veja [INSTALACAO.md](INSTALACAO.md) para detalhes.

## 📝 Fluxo de Uso

```
┌─────────────┐
│ Criar Sala  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Formar Time │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Desafiar    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Votar Modo  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Votar Mapa  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Jogar!      │
└─────────────┘
```

## 🎮 Comandos

| Comando | Descrição |
|---------|-----------|
| `/lategame` | Abre o painel |
| `/aceitarlg` | Aceita desafio |
| `HOME` | Placar |
| `F7` | Sair do espectador |

## ⚙️ Configuração Principal

```lua
globalConfig = {
    MaximumRoomMembers = 10,
    Markers = { vector3(193.24, -908.70, 28.42) },
    DisableDimensionsVerify = false,
    ChangeSideByRound = false,
    MaxRoundWins = 10,
    RoundResetDelayMs = 4000,
    SpawnTimes = {
        FirstRound = 30,
        OthersRounds = 5,
    },
    -- Maps, Modes, Rounds, Language...
}
```

Veja [CONFIGURACAO.md](CONFIGURACAO.md) para detalhes completos.

## 🔌 API Básica

### Client

```lua
local apiServer = Tunnel.getInterface('combo_lategame')

-- Criar sala
local success, roomId = apiServer.createRoom('Minha Sala')

-- Listar salas
local rooms = apiServer.getRooms()

-- Entrar em sala
local success = apiServer.joinRoom(roomId)
```

### Server

```lua
-- Verificar se está em partida
local inMatch = Player(source).state.combo_lategame_match

-- Obter ID da partida
local matchId = Player(source).state.combo_lategame_mid

-- Obter time
local team = Player(source).state.combo_lategame_team
```

Veja [API.md](API.md) para documentação completa.

## 🆘 Troubleshooting Rápido

### "vrp/lib/utils.lua nao encontrado"
- Verifique se vRP está iniciado antes
- Adapte para seu framework se não usa vRP

### Painel não abre
- Configure markers ou use `/lategame`
- Verifique se a UI está carregando

### Partidas não iniciam
- Verifique configuração de mapas e modos
- Verifique coordenadas de spawn

### Jogadores não restaurados
- Verifique funções de save/restore
- Verifique routing buckets

Veja [DOCUMENTACAO.md](DOCUMENTACAO.md) para mais detalhes.

## 📚 Documentação Completa

### [Apresentação](DOCUMENTACAO.md)
Visão geral do sistema, funcionalidades, compatibilidade e fluxo de uso.

### [Instalação](INSTALACAO.md)
Guia passo a passo de instalação para:
- vRP (suporte nativo)
- QBCore (adaptação)
- ESX (adaptação)
- Outros frameworks

### [Configuração](CONFIGURACAO.md)
Guia completo de configuração:
- Configuração principal
- Configuração de mapas
- Configuração de modos
- Configuração de armas
- Traduções
- Funções de integração
- Convars do servidor

### [API e Integração](API.md)
Documentação completa da API:
- Client events
- Server events
- Server functions
- Player states
- Exemplos de integração
- Boas práticas

## 🛠️ Estrutura de Arquivos

```
combo_lategame/
├── fxmanifest.lua
├── _client.lua
├── _server.lua
├── config/
│   ├── config.lua
│   ├── client/functions.lua
│   └── server/functions.lua
├── web/
│   ├── dist/
│   └── images/
├── README.md
├── INSTALACAO.md
├── CONFIGURACAO.md
├── API.md
├── DOCUMENTACAO.md
└── INDEX.md
```

## 🎯 Roadmap

Funcionalidades planejadas:

- [ ] Sistema de ranking persistente
- [ ] Sistema de apostas
- [ ] Modo Plante/Desarme
- [ ] Sistema de economia
- [ ] Integração nativa QBCore/ESX
- [ ] Sistema de logs
- [ ] Interface de administração

## 📄 Licença

Verifique a licença do script antes de usar em produção.

---

**Combo Late Game** - Sistema de partidas competitivas para FiveM

Para suporte, consulte a documentação específica de cada seção.
