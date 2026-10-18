# Instalação

Guia completo de instalação do Combo Late Game em diferentes frameworks.

## Requisitos

- **FiveM Server** com OneSync habilitado
- **Server Version** 2843 ou superior
- Framework base (vRP, QBCore, ESX, ou personalizado)

## Instalação Básica

### 1. Download e Extração

Baixe o script e extraia na pasta `resources` do seu servidor:

```
resources/
└── [combo_lategame]/
    └── combo_lategame/
        ├── fxmanifest.lua
        ├── _client.lua
        ├── _server.lua
        ├── config/
        ├── web/
        └── stream/
```

### 2. Adicionar ao server.cfg

Adicione a linha no seu `server.cfg`:

```cfg
ensure combo_lategame
```

**Importante**: Certifique-se de que o resource vRP (ou seu framework) inicia antes do combo_lategame:

```cfg
ensure vrp
ensure combo_lategame
```

### 3. Configurar Convars (Opcional)

Adicione ao seu `server.cfg` para customizar tempos de votação:

```cfg
set combo_lategame_vote_mode_sec 120
set combo_lategame_vote_map_sec 120
```

## Integração por Framework

### vRP / vRP 2.0

O script já vem configurado para vRP. Apenas verifique:

1. **Configuração Server** (`config/server/functions.lua`):

```lua
local vRP = Proxy.getInterface('vRP')
local vRPclient = Tunnel.getInterface('vRP')

serverConfig = {
    getUserId = function(userSource)
        return vRP.Passport(userSource)
    end,

    getUserSource = function(userId)
        return vRP.Source(userId)
    end,

    getUserName = function(userSource)
        local passport = vRP.Passport(userSource)
        local identity = vRP.Identity(passport) or {}
        return (identity.Name or 'Indivíduo') .. ' ' .. (identity.Lastname or 'Indigente')
    end,

    hasPermission = function(userId, permission)
        return vRP.HasPermission(userId, permission)
    end,

    -- ... outras funções
}
```

2. **Configuração Client** (`config/client/functions.lua`):

```lua
local vRP = Proxy.getInterface('vRP')

-- As funções já estão configuradas para vRP
```

3. **Notificações** (`config/config.lua`):

Verifique se o evento `Notify` está configurado corretamente no seu servidor.

### QBCore

Para usar com QBCore, siga estes passos:

#### 1. Modificar `config/server/functions.lua`

```lua
local QBCore = exports['qb-core']:GetCoreObject()

serverConfig = {
    checkQueueName = function(queueName, queueOwnerSource)
        -- Implementar validação de nomes reservados
        return true
    end,

    isUserAdmin = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player and player.PlayerData.permission == 'admin'
    end,

    getUserNameToSpec = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player and player.PlayerData.charinfo.firstname .. ' ' .. player.PlayerData.charinfo.lastname
    end,

    getUserIdentifier = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player and player.PlayerData.citizenid
    end,

    userCanOpenLateGame = function(userSource)
        return true
    end,

    getUserSource = function(userId)
        -- Implementar busca de source por citizenid
        local players = QBCore.Functions.GetPlayers()
        for _, src in ipairs(players) do
            local player = QBCore.Functions.GetPlayer(src)
            if player.PlayerData.citizenid == userId then
                return src
            end
        end
        return nil
    end,

    getUserId = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player and player.PlayerData.citizenid
    end,

    hasPermission = function(userId, permission)
        -- Implementar verificação de permissões
        return true
    end,

    hasUserVip = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        return player and player.PlayerData.metadata['isvip']
    end,

    getUserName = function(userSource)
        local player = QBCore.Functions.GetPlayer(userSource)
        if not player then return 'Unknown' end
        return player.PlayerData.charinfo.firstname .. ' ' .. player.PlayerData.charinfo.lastname
    end,

    request = function(userSource, text)
        -- Implementar sistema de request usando QBCore
        return true
    end,

    setPlayerRoutingBucket = function(userSource, dimension)
        SetPlayerRoutingBucket(userSource, dimension)
    end,

    canUserJoinInGame = function(userSource)
        if GetPlayerRoutingBucket(userSource) > 0 then return false end
        local playerPed = GetPlayerPed(userSource)
        local health = GetEntityHealth(playerPed)
        return health > 101
    end,

    savePlayerInfos = function(userSource)
        Player(userSource).state:set('inLateGame', true, true)
        Wait(50)

        local playerPed = GetPlayerPed(userSource)
        local health = GetEntityHealth(playerPed)

        playerInfos[userSource] = {
            armour = GetPedArmour(playerPed),
            coords = GetEntityCoords(playerPed),
            health = health,
            dimension = GetPlayerRoutingBucket(userSource)
        }

        Player(userSource).state:set('temporaryDataLateGame', playerInfos[userSource], true)
    end,

    setPlayerInfos = function(userSource)
        Player(userSource).state:set('teamLateGame', nil, true)
        Wait(50)

        local userInfos = playerInfos[userSource]

        if userInfos then
            local playerPed = GetPlayerPed(userSource)
            SetEntityCoords(playerPed, userInfos.coords)
            SetPedArmour(playerPed, userInfos.armour)
            SetEntityHealth(playerPed, userInfos.health)
            serverConfig.setPlayerRoutingBucket(userSource, userInfos.dimension)

            playerInfos[userSource] = nil
            Player(userSource).state:set('temporaryDataLateGame', nil, true)
        end

        SetTimeout(2000, function()
            Player(userSource).state:set('inLateGame', false, true)
        end)
    end,

    sendWebhookWinner = function(winningTeamName, looserTeamName, matchMap, matchWeapon)
        -- Implementar webhook se necessário
    end,
}

return serverConfig
```

#### 2. Modificar `config/client/functions.lua`

```lua
local QBCore = exports['qb-core']:GetCoreObject()

clientConfig = {
    respawnPed = function(ped, disableRepeat)
        local pedCoords = GetEntityCoords(ped)
        NetworkResurrectLocalPlayer(pedCoords.x, pedCoords.y, pedCoords.z, true, true, false)
        ClearPedBloodDamage(ped)
        ClearPedTasks(ped)
        ClearPedSecondaryTask(ped)
        clientConfig.setPedMaxHealth(ped)

        if disableRepeat then
            return clientConfig.setEntityInvincible(ped, false)
        end

        clientConfig.setEntityInvincible(ped, true)
        SetTimeout(1000, function()
            clientConfig.respawnPed(ped, true)
        end)
    end,

    getPedWeapons = function(ped)
        -- Implementar conforme necessário
        return {}
    end,

    setWeaponToPed = function(ped, weaponsTable, isInfinityAmmo)
        clientConfig.removeAllPedWeapons(ped)
        SetPedConfigFlag(ped, 48, false)
        Wait(2000)

        for _, weaponName in pairs(weaponsTable) do
            GiveWeaponToPed(ped, GetHashKey(weaponName), 250, false, true)
            if isInfinityAmmo then
                SetPedInfiniteAmmo(ped, true, weaponName)
            end
        end
    end,

    setPedWithoutCollision = function(ped)
        for _, players in ipairs(GetActivePlayers()) do
            local otherPed = GetPlayerPed(players)
            if ped ~= otherPed then
                SetEntityNoCollisionEntity(ped, otherPed, true)
            end
        end
    end,

    removeAllPedWeapons = function(ped)
        RemoveAllPedWeapons(ped, true)
    end,

    setEntityCoords = function(...)
        local args = {...}
        local ped = table.remove(args, 1)
        SetEntityCoords(ped, table.unpack(args))
    end,

    setEntityHealth = function(ped, health)
        SetEntityHealth(ped, health)
    end,

    setPedDead = function(ped)
        clientConfig.setEntityHealth(ped, 101)
    end,

    isPedDead = function(ped)
        return GetEntityHealth(ped) <= 101
    end,

    setPedMaxHealth = function(ped)
        local healthToSet = GetPedMaxHealth(ped)
        clientConfig.setEntityHealth(ped, healthToSet)
    end,

    setPlayerInSpectatorMode = function(toggle, playerToSpectate)
        if toggle then
            NetworkSetInSpectatorMode(true, playerToSpectate)
        else
            if NetworkIsInSpectatorMode() then
                NetworkSetInSpectatorMode(false)
            end
        end
    end,

    freezeEntityPosition = function(entity, toggleStatus)
        FreezeEntityPosition(entity, toggleStatus)
    end,

    setEntityInvincible = function(entity, toggleStatus)
        SetEntityInvincible(entity, toggleStatus)
    end,

    isPlayerInvincible = function(playerId)
        return GetPlayerInvincible(playerId)
    end,

    setEntityVisible = function(entity, toggle)
        SetEntityVisible(entity, toggle)
    end,

    isEntityVisible = function(ped)
        return IsEntityVisible(ped)
    end,

    setCurrentPedWeapon = function(ped, weaponHash, forceInHand)
        SetCurrentPedWeapon(ped, GetHashKey(weaponHash), forceInHand)
    end,

    returnPedWeapons = function(ped, weaponsTable)
        clientConfig.removeAllPedWeapons(ped)
        SetPedConfigFlag(ped, 48, true)
    end,

    openHandler = function(handler)
        if #globalConfig.Markers > 0 then
            CreateThread(function()
                while true do
                    local sleepTime = 1000
                    local playerPed = PlayerPedId()
                    local pedCoords = GetEntityCoords(playerPed)

                    for _, markerCoords in ipairs(globalConfig.Markers) do
                        local distance = #(pedCoords - markerCoords)

                        if distance < 7 then
                            sleepTime = 0
                            DrawMarker(21, markerCoords, 0, 0, 0, 0, 180.0, 130.0, 2.0, 2.0, 1.0, 255, 0, 0, 50, 1, 0, 0, 1)

                            if IsControlJustPressed(0, 38) and distance < 2 then
                                handler()
                                Wait(1000)
                            end
                        end
                    end

                    Wait(sleepTime)
                end)
            end)
        else
            RegisterCommand('lategame', handler)
        end
    end,

    openHandlerSpectator = function(handler)
        RegisterCommand('leaveLategameSpectator', handler)
    end,

    drawNotification = function(timer)
        SetNotificationTextEntry('STRING')
        AddTextComponentString('Aperte [HOME] para abrir o placar.')
        DrawNotification(true, false)
    end,

    disableControls = function()
        DisableControlAction(1, 80, true)
    end
}

CreateThread(function()
    RegisterKeyMapping('activeStatistic', 'Abrir Score Board', 'keyboard', 'HOME')
    RegisterKeyMapping('leaveLategameSpectator', 'Sair do espectador', 'keyboard', 'F7')
end)

function apiConfig.setPlayerHealth(health)
    local ped = PlayerPedId()
    clientConfig.setEntityHealth(ped, health)
end

return clientConfig
```

#### 3. Modificar `config/config.lua`

```lua
function globalConfig.notify(...)
    local args = {...}

    if IsDuplicityVersion() then
        local source = args[1]
        TriggerClientEvent('QBCore:Notify', source, args[3], 'primary', args[4])
    else
        TriggerEvent('QBCore:Notify', args[2], 'primary', args[3])
    end
end
```

#### 4. Modificar `fxmanifest.lua`

Remova as dependências do vRP se não for usar:

```lua
fx_version "bodacious"
game "gta5"

ui_page "web/dist/index.html"
ui_page_preload "yes"

files {
    "config/client/*.lua",
    "config/config.lua",
    "web/dist/*.*",
    "web/dist/**/*.*",
    "web/images/*.*"
}

dependencies {
    "/onesync",
    "/server:2843",
    "qb-core"  -- Adicione seu framework
}

client_script "_client.lua"
server_script "_server.lua"
```

### ESX

Para usar com ESX, siga um processo similar ao QBCore, mas usando as funções do ESX:

#### 1. Modificar `config/server/functions.lua`

```lua
local ESX = exports['es_extended']:getSharedObject()

serverConfig = {
    getUserId = function(userSource)
        local xPlayer = ESX.GetPlayerFromId(userSource)
        return xPlayer and xPlayer.identifier
    end,

    getUserSource = function(userId)
        local players = ESX.GetPlayers()
        for _, src in ipairs(players) do
            local xPlayer = ESX.GetPlayerFromId(src)
            if xPlayer.identifier == userId then
                return src
            end
        end
        return nil
    end,

    getUserName = function(userSource)
        local xPlayer = ESX.GetPlayerFromId(userSource)
        if not xPlayer then return 'Unknown' end
        return xPlayer.getName()
    end,

    hasPermission = function(userId, permission)
        -- Implementar verificação de permissões
        return true
    end,

    -- ... outras funções similares ao QBCore
}
```

#### 2. Modificar `config/config.lua`

```lua
function globalConfig.notify(...)
    local args = {...}

    if IsDuplicityVersion() then
        local source = args[1]
        TriggerClientEvent('esx:showNotification', source, args[3])
    else
        TriggerEvent('esx:showNotification', args[2])
    end
end
```

## Verificação de Instalação

Após instalar, verifique:

1. **Inicie o servidor** e observe o console:
   ```
   [combo_lategame] Script iniciado com sucesso
   ```

2. **Teste o comando**:
   - Entre no servidor
   - Digite `/lategame` ou vá até o marker configurado
   - O painel deve abrir

3. **Crie uma sala**:
   - Clique em "Criar Sala"
   - Digite um nome
   - Verifique se a sala foi criada

4. **Teste um desafio**:
   - Crie duas salas com diferentes jogadores
   - Desafie uma sala da outra
   - Verifique se a votação inicia

## Problemas Comuns

### "vrp/lib/utils.lua nao encontrado"

**Causa**: O resource vRP não está iniciado ou o nome está incorreto.

**Solução**:
1. Verifique se `ensure vrp` está no server.cfg antes de `ensure combo_lategame`
2. Verifique se o nome do resource é `vrp` ou `vRP` (maiúscula)
3. Se não usar vRP, adapte as funções de integração

### Painel não abre

**Causa**: Markers não configurados ou comando não registrado.

**Solução**:
1. Adicione coordenadas em `globalConfig.Markers` em `config/config.lua`
2. Ou use o comando `/lategame`
3. Verifique se a UI está carregando corretamente

### Partidas não iniciam

**Causa**: Mapas ou modos não configurados corretamente.

**Solução**:
1. Verifique se `config.Maps` e `config.Modes` estão configurados
2. Verifique se as coordenadas de spawn estão corretas
3. Verifique se as imagens dos banners existem

### Jogadores não são restaurados

**Causa**: Funções de save/restore não funcionando para o framework.

**Solução**:
1. Verifique `savePlayerInfos` e `setPlayerInfos` em `config/server/functions.lua`
2. Verifique se `DisableDimensionsVerify` está configurado corretamente
3. Teste as funções individualmente

## Próximos Passos

Após a instalação básica:

1. Configure os mapas e modos em `config/config.lua`
2. Ajuste os tempos e limites conforme seu servidor
3. Personalize as traduções e notificações
4. Configure o webhook de vencedor se desejar
5. Teste o sistema completo com jogadores

Para mais detalhes de configuração, veja [README.md](README.md).
