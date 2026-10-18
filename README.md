# Combo Late Game

> Sistema de partidas competitivas para FiveM com votação de modos e mapas, sistema de salas e duelos entre times.

O **Combo Late Game** é um sistema completo de partidas competitivas para servidores FiveM. Os jogadores criam salas, formam times, desafiam outras salas e votam no modo de jogo e mapa antes de iniciar a partida em formato melhor de N rounds com troca de lados opcional, placar ao vivo, kill feed e zona segura que fecha progressivamente.

## 📖 Documentação

- **[INDEX.md](INDEX.md)** - Índice completo da documentação
- **[DOCUMENTACAO.md](DOCUMENTACAO.md)** - Apresentação e visão geral
- **[INSTALACAO.md](INSTALACAO.md)** - Guia de instalação (vRP, QBCore, ESX)
- **[CONFIGURACAO.md](CONFIGURACAO.md)** - Guia de configuração
- **[API.md](API.md)** - Documentação da API

## 🌐 Hospedar a Documentação

### Com GitHub Desktop (Recomendado)

Se você tem GitHub Desktop instalado:

1. **[INICIO_RAPIDO.md](INICIO_RAPIDO.md)** - Guia rápido (5 minutos)
2. **[DEPLOY_GITHUB_DESKTOP.md](DEPLOY_GITHUB_DESKTOP.md)** - Guia completo

### Com Git CLI

Se prefere linha de comando:

1. **[README_GITHUB.md](README_GITHUB.md)** - Guia rápido
2. **[DEPLOY.md](DEPLOY.md)** - Guia completo

## Funcionalidades

### Sistema de Salas
- Criação de salas com nome customizado
- Limite configurável de membros por sala (padrão: 10)
- Sistema de dono (owner) da sala com permissões de kick
- Entrada em salas existentes
- Saída automática ao desconectar

### Sistema de Desafios
- Líderes de sala podem desafiar outras salas
- Sistema de convites com aceitação/rejeição
- Previne desafios para a própria sala
- Bloqueio de solicitações pendentes

### Votação
- Votação em duas fases: Modo de jogo → Mapa
- Chat de votação entre participantes
- Sistema de chat com delay anti-spam
- Tempo configurável por convar para cada fase
- Resolução por maioria ou pluralidade

### Modos de Jogo
- **Pistols**: Pistolas diversas
- **Rifles**: Rifles de assalto
- **Submachine**: Submetralhadoras
- **MixedGuns**: Combinação progressiva de armas
- Configuração de armas por round
- Weapon pool configurável

### Mapas
- Mirage Exclusive
- Dust 1.6
- Dust Go
- ItalyV1
- ClockTower
- Sistema de Safezone que fecha progressivamente
- Configuração de dano por segundo fora da zona
- Spawn points configuráveis por time
- Distância de spawn configurável

### Sistema de Partidas
- Melhor de N rounds (configurável, padrão: 10)
- Troca de lados opcional por round
- Proteção de spawn configurável
- Sistema de death/respawn
- Round delay configurável
- Placar ao vivo (HOME key)
- Kill feed com nomes dos jogadores

### Espectação
- Espectar partidas em andamento
- Sistema de target automático
- Comando para sair do modo espectador
- Acompanhamento de estatísticas em tempo real

### Integração vRP
- Sistema de passport/user_id
- Verificação de permissões
- Sistema de nomes personalizados
- Verificação de nomes reservados
- Webhook de vencedor (configurável)
- Salvamento/restauração de estado do jogador

### Routing Buckets
- Uso de routing buckets isolados por partida
- Verificação de capacidade de entrada
- Salvamento de coordenadas de retorno
- Restauração automática ao sair

## Instalação

### Dependências

O script requer:
- **vrp** ou **vRP** (framework obrigatório)
- **onesync** (habilitado no servidor)
- **server version 2843+**

### Arquivos

Estrutura de arquivos:
```
combo_lategame/
├── fxmanifest.lua
├── _client.lua
├── _server.lua
├── config/
│   ├── config.lua
│   ├── client/
│   │   └── functions.lua
│   └── server/
│       └── functions.lua
├── web/
│   ├── dist/
│   │   ├── index.html
│   │   ├── messages.json
│   │   └── assets/
│   └── images/
└── stream/ (opcional, para mapas)
```

### Passo a Passo

1. **Copie o resource** para a pasta `resources` do seu servidor:
   ```
   resources/[combo_lategame]/combo_lategame
   ```

2. **Configure as integrações** em `config/server/functions.lua`:
   - Verifique as funções de integração com vRP
   - Configure permissões conforme necessário
   - Ajuste o webhook de vencedor se desejar

3. **Configure o script** em `config/config.lua`:
   - Ajuste os mapas e suas configurações
   - Configure os modos de jogo
   - Defina os limites e tempos
   - Personalize as traduções

4. **Adicione ao server.cfg**:
   ```
   ensure combo_lategame
   ```

5. **Configure as convars** (opcional):
   ```cfg
   set combo_lategame_vote_mode_sec 120
   set combo_lategame_vote_map_sec 120
   ```

6. **Inicie o servidor** e verifique se o resource iniciou corretamente.

### Adaptando para Outros Frameworks

Para adaptar o script para outros frameworks (QBCore, ESX, etc.), você precisa modificar os arquivos de configuração:

#### 1. `config/server/functions.lua`

Substitua as funções de integração vRP pelas equivalentes do seu framework:

```lua
-- Exemplo para QBCore
local QBCore = exports['qb-core']:GetCoreObject()

serverConfig = {
    getUserId = function(userSource)
        return QBCore.Functions.GetPlayer(userSource).PlayerData.citizenid
    end,

    getUserSource = function(userId)
        -- Implementar busca de source por citizenid
    end,

    getUserName = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player.PlayerData.charinfo.firstname .. ' ' .. player.PlayerData.charinfo.lastname
    end,

    hasPermission = function(userId, permission)
        -- Implementar verificação de permissões
    end,

    -- ... outras funções
}
```

#### 2. `config/client/functions.lua`

Adapte as funções que dependem do framework:

```lua
-- Exemplo para QBCore
local QBCore = exports['qb-core']:GetCoreObject()

clientConfig = {
    -- Funções que dependem do framework específico
}
```

#### 3. `config/config.lua`

Ajuste a função `notify` para usar o sistema de notificação do seu framework:

```lua
function globalConfig.notify(...)
    local args = {...}

    if IsDuplicityVersion() then
        local source = args[1]
        -- Exemplo QBCore
        TriggerClientEvent('QBCore:Notify', source, args[3], 'primary', args[4])
    else
        -- Exemplo QBCore
        TriggerEvent('QBCore:Notify', args[2], 'primary', args[3])
    end
end
```

#### 4. `fxmanifest.lua`

Remova ou ajuste as dependências do vRP se necessário:

```lua
-- Remova ou comente estas linhas se não usar vRP:
-- server_script '@ElectronAC/src/include/server.lua'
-- client_script '@ElectronAC/src/include/client.lua'
```

## Configuração

### Configuração Principal (`config/config.lua`)

```lua
globalConfig = {
    -- Limite máximo de membros por sala
    MaximumRoomMembers = 10,

    -- Coordenadas dos markers para abrir o painel
    Markers = {
        vector3(193.24,-908.70,28.42),
    },

    -- Desabilitar verificação de dimensions (routing buckets)
    DisableDimensionsVerify = false,

    -- Trocar lados a cada round
    ChangeSideByRound = false,

    -- Vitórias necessárias para encerrar a partida
    MaxRoundWins = 10,

    -- Delay após eliminar time adversário (ms)
    RoundResetDelayMs = 4000,

    -- Proteção de spawn (segundos)
    SpawnTimes = {
        FirstRound = 30,    -- Primeiro round
        OthersRounds = 5,   -- Rounds subsequentes
    },

    -- Configuração de mapas
    Maps = {
        ['NomeDoMapa'] = {
            Name = 'Nome Exibido',
            Banner = '../images/banner.png',
            SpawnDistance = 10,
            SideOne = { 
                [1] = vector3(x, y, z),
            },
            SideTwo = { 
                [1] = vector3(x, y, z),
            },
            Safezone = {
                Center = vector3(x, y, z),
                Variation = 20,
                DamagePerSecond = 25,
                TimeToClose = 150,
                Initial = { Radius = 150 },
                Finale = { Radius = 5 }
            },
        },
    },

    -- Configuração de modos
    Modes = {
        ['Pistols'] = { Name = 'Pistola', Banner = '../images/pistol.png' },
        ['Rifles'] = { Name = 'Rifle', Banner = '../images/rifle.png' },
    },

    -- Configuração de armas por round
    Rounds = {
        Pistols = {
            [1] = { 'WEAPON_PISTOL', 'WEAPON_COMBATPISTOL' },
            [2] = { 'WEAPON_PISTOL_MK2', 'WEAPON_PISTOL' },
            -- ...
        },
    },

    -- Traduções
    Language = {
        WinningTeam = 'Round finalizado! O time ganhador foi: %s',
        RequestSent = 'Solicitação enviada.',
        -- ...
    }
}
```

### Configuração Server (`config/server/functions.lua`)

Funções importantes para adaptar:

- `getUserId(userSource)`: Retorna o ID único do jogador
- `getUserSource(userId)`: Retorna o source do jogador pelo ID
- `getUserName(userSource)`: Retorna o nome do jogador
- `hasPermission(userId, permission)`: Verifica permissões
- `userCanOpenLateGame(userSource)`: Verifica se pode abrir o painel
- `checkQueueName(queueName, queueOwnerSource)`: Valida nomes reservados
- `canUserJoinInGame(userSource)`: Verifica se pode entrar na partida
- `savePlayerInfos(userSource)`: Salva estado do jogador
- `setPlayerInfos(userSource)`: Restaura estado do jogador
- `sendWebhookWinner(...)`: Envia webhook ao finalizar partida

### Configuração Client (`config/client/functions.lua`)

Funções importantes para adaptar:

- `respawnPed(ped, disableRepeat)`: Respawn do ped
- `getPedWeapons(ped)`: Obtém armas do ped
- `setWeaponToPed(ped, weaponsTable, isInfinityAmmo)`: Define armas
- `removeAllPedWeapons(ped)`: Remove todas as armas
- `setPlayerInSpectatorMode(toggle, playerToSpectate)`: Modo espectador
- `openHandler(handler)`: Handler para abrir painel
- `openHandlerSpectator(handler)`: Handler para sair do espectador

## API e Exports

### Client Events

- `combo_lategame:enterMatch(matchData)`: Entra na partida
- `combo_lategame:newRound(payload)`: Novo round
- `combo_lategame:matchSeriesEnd(coords, serverRestore)`: Fim da série
- `combo_lategame:challengeInvite(challengeId, message)`: Convite de desafio
- `combo_lategame:votationPhase(phase)`: Fase de votação
- `combo_lategame:votationClosed()`: Votação encerrada
- `combo_lategame:applyWeapons(weapons)`: Aplica armas
- `combo_lategame:receiveWeapon(weaponName, ammo)`: Recebe arma
- `combo_lategame:setScoreboard(show, teams)`: Atualiza scoreboard
- `combo_lategame:killFeed(feed)`: Kill feed
- `combo_lategame:chatMessage(message)`: Mensagem de chat
- `combo_lategame:startSpectateTarget(target, name)`: Inicia espectação
- `combo_lategame:updateStatistic(data)`: Atualiza estatísticas

### Server Events

- `combo_lategame:rejectChallenge(challengeId)`: Rejeita desafio
- `combo_lategame:requestWeapon(weaponName, ammo)`: Solicita arma
- `combo_lategame:clearMatchState()`: Limpa estado da partida
- `combo_lategame:stashReturnCoords(x, y, z, h)`: Salva coords de retorno
- `combo_lategame:roundPlayerDown(killerSrc)`: Jogador morreu

### Server Functions (Tunnel)

- `createRoom(roomName)`: Cria sala
- `joinRoom(roomId)`: Entra em sala
- `leaveRoom(roomId)`: Sai de sala
- `deleteRoom(roomId)`: Deleta sala
- `kickMember(roomId, targetSource)`: Expulsa membro
- `kickFromMyRoom(targetSource)`: Expulsa da própria sala
- `getRooms()`: Lista salas
- `getCurrentRoom()`: Sala atual
- `sendChallenge(targetRoomId)`: Envia desafio
- `acceptChallenge(challengeId)`: Aceita desafio
- `voteGameMode(modeKey)`: Vota modo
- `voteGameMap(mapKey)`: Vota mapa
- `sendChatMessage(content)`: Envia mensagem
- `getActiveMatchCards()`: Partidas ativas
- `spectateRunningMatch(matchId)`: Especta partida
- `monitorRunningMatch(matchId)`: Monitora partida
- `openMatchStatistic()`: Abre estatísticas
- `closeMatchStatistic()`: Fecha estatísticas

### Player States

- `combo_lategame_match`: Boolean, se está em partida
- `combo_lategame_mid`: Number, ID da partida
- `combo_lategame_team`: String, time (A ou B)
- `inPvp`: Boolean, se está em PvP
- `inLateGame`: Boolean, se está no Late Game
- `temporaryDataLateGame`: Table, dados temporários
- `teamLateGame`: Table, time do Late Game

## Comandos

### Client

- `/lategame` ou **Marker**: Abre o painel do Late Game
- `/aceitarlg`: Aceita desafio pendente
- `HOME`: Abre/fecha o scoreboard (configurável)
- `F7`: Sai do modo espectador (configurável)

### Server

- `combo_lategame_reset_voting`: Reseta votações (console only)

## Convars

- `combo_lategame_vote_mode_sec`: Tempo de votação de modo (padrão: 120s)
- `combo_lategame_vote_map_sec`: Tempo de votação de mapa (padrão: 120s)

## Troubleshooting

### Erro: "vrp/lib/utils.lua nao encontrado"
**Solução**: Verifique se o resource vRP está iniciado antes do combo_lategame no server.cfg.

### Jogadores não conseguem entrar na partida
**Solução**: Verifique se `DisableDimensionsVerify` está configurado corretamente e se `canUserJoinInGame` está retornando true.

### Partida não inicia após votação
**Solução**: Verifique se os mapas e modos configurados existem em `config/config.lua` e se as configurações de spawn estão corretas.

### Votação encerra prematuramente
**Solução**: Aumente os tempos das convars `combo_lategame_vote_mode_sec` e `combo_lategame_vote_map_sec`.

### Jogadores não são restaurados ao sair
**Solução**: Verifique se as funções `savePlayerInfos` e `setPlayerInfos` estão funcionando corretamente para o seu framework.

## Suporte

Para suporte adicional, consulte a documentação do seu framework base (vRP, QBCore, ESX, etc.) e adapte as funções de integração conforme necessário.
