-- Serverscript (LocalScript no StarterPlayerScripts)
-- ESP/Wallhack para Roblox

-- Configurações
local ESP_SETTINGS = {
    ENABLED = true, -- Começa ativado
    WALLHACK = true, -- Ver através das paredes
    SHOW_NAMES = true,
    SHOW_DISTANCE = true,
    SHOW_HEALTH = true,
    SHOW_BOX = true,
    SHOW_TRACER = true, -- Linha do pé do jogador até o alvo
    
    MAX_DISTANCE = 1000, -- Distância máxima
    UPDATE_INTERVAL = 0.1, -- Atualização a cada 0.1 segundos
    
    -- Cores
    TEAM_COLOR = true, -- Usar cor do time
    FRIENDLY_COLOR = Color3.fromRGB(0, 255, 0), -- Verde para aliados
    ENEMY_COLOR = Color3.fromRGB(255, 0, 0), -- Vermelho para inimigos
    NEUTRAL_COLOR = Color3.fromRGB(255, 255, 0), -- Amarelo para neutros
    
    -- Tamanhos
    BOX_THICKNESS = 1,
    TEXT_SIZE = 18,
    TRACER_THICKNESS = 1,
    
    -- Posições
    TEXT_OFFSET = Vector2.new(0, -40), -- Offset do texto acima da cabeça
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}
local controlGui = nil

-- Função para atualizar todos os ESPs visuais
local function updateAllESPVisibility()
    for _, espObject in pairs(espCache) do
        -- Atualizar visibilidade de todos os elementos do ESP
        if espObject.Box then 
            espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX 
        end
        if espObject.NameLabel then 
            espObject.NameLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_NAMES 
            espObject.NameLabel.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(100, 100, 100)
        end
        if espObject.DistanceLabel then 
            espObject.DistanceLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_DISTANCE 
        end
        if espObject.HealthLabel then 
            espObject.HealthLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_HEALTH 
        end
        if espObject.Tracer then 
            espObject.Tracer.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_TRACER 
        end
    end
end

-- Função para criar a interface básica
local function createBasicGUI()
    -- Criar ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESPBasicGUI"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Frame principal do botão
    local mainButtonFrame = Instance.new("Frame")
    mainButtonFrame.Name = "MainButtonFrame"
    mainButtonFrame.Size = UDim2.new(0, 100, 0, 50)
    mainButtonFrame.Position = UDim2.new(1, -110, 0, 20)
    mainButtonFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainButtonFrame.BackgroundTransparency = 0.2
    mainButtonFrame.BorderSizePixel = 0
    mainButtonFrame.Parent = screenGui
    
    -- Arredondar cantos
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 10)
    uiCorner.Parent = mainButtonFrame
    
    -- Sombra suave
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(80, 80, 80)
    uiStroke.Thickness = 2
    uiStroke.Parent = mainButtonFrame
    
    -- Botão ESP
    local espButton = Instance.new("TextButton")
    espButton.Name = "ESPButton"
    espButton.Size = UDim2.new(1, -10, 1, -10)
    espButton.Position = UDim2.new(0, 5, 0, 5)
    espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
    espButton.BackgroundTransparency = 0.3
    espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
    espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
    espButton.TextSize = 16
    espButton.Font = Enum.Font.GothamBold
    espButton.Parent = mainButtonFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 8)
    buttonCorner.Parent = espButton
    
    -- Botão de configurações (pequeno)
    local settingsButton = Instance.new("TextButton")
    settingsButton.Name = "SettingsButton"
    settingsButton.Size = UDim2.new(0, 30, 0, 30)
    settingsButton.Position = UDim2.new(0, -35, 0.5, -15)
    settingsButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    settingsButton.BackgroundTransparency = 0.3
    settingsButton.Text = "⚙"
    settingsButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    settingsButton.TextSize = 16
    settingsButton.Font = Enum.Font.GothamBold
    settingsButton.Visible = false
    settingsButton.Parent = mainButtonFrame
    
    local settingsCorner = Instance.new("UICorner")
    settingsCorner.CornerRadius = UDim.new(0, 6)
    settingsCorner.Parent = settingsButton
    
    -- Painel de configurações
    local settingsPanel = Instance.new("Frame")
    settingsPanel.Name = "SettingsPanel"
    settingsPanel.Size = UDim2.new(0, 200, 0, 300)
    settingsPanel.Position = UDim2.new(0, -205, 0, 0)
    settingsPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    settingsPanel.BackgroundTransparency = 0.1
    settingsPanel.BorderSizePixel = 0
    settingsPanel.Visible = false
    settingsPanel.Parent = mainButtonFrame
    
    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 10)
    panelCorner.Parent = settingsPanel
    
    local panelStroke = Instance.new("UIStroke")
    panelStroke.Color = Color3.fromRGB(60, 60, 60)
    panelStroke.Thickness = 2
    panelStroke.Parent = settingsPanel
    
    -- Título do painel
    local panelTitle = Instance.new("TextLabel")
    panelTitle.Name = "Title"
    panelTitle.Size = UDim2.new(1, 0, 0, 40)
    panelTitle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    panelTitle.BackgroundTransparency = 0.5
    panelTitle.Text = "CONFIGURAÇÕES ESP"
    panelTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    panelTitle.TextSize = 16
    panelTitle.Font = Enum.Font.GothamBold
    panelTitle.Parent = settingsPanel
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = panelTitle
    
    -- Container para os toggles
    local toggleContainer = Instance.new("ScrollingFrame")
    toggleContainer.Name = "ToggleContainer"
    toggleContainer.Size = UDim2.new(1, -10, 1, -50)
    toggleContainer.Position = UDim2.new(0, 5, 0, 45)
    toggleContainer.BackgroundTransparency = 1
    toggleContainer.BorderSizePixel = 0
    toggleContainer.ScrollBarThickness = 4
    toggleContainer.CanvasSize = UDim2.new(0, 0, 0, 250)
    toggleContainer.Parent = settingsPanel
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.Parent = toggleContainer
    
    -- Função para criar um toggle
    local function createToggleOption(text, settingName, description)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Size = UDim2.new(1, 0, 0, 35)
        toggleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        toggleFrame.BackgroundTransparency = 0.5
        toggleFrame.Parent = toggleContainer
        
        local frameCorner = Instance.new("UICorner")
        frameCorner.CornerRadius = UDim.new(0, 6)
        frameCorner.Parent = toggleFrame
        
        local label = Instance.new("TextLabel")
        label.Text = text
        label.Size = UDim2.new(0.65, 0, 1, 0)
        label.Position = UDim2.new(0, 10, 0, 0)
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.TextSize = 14
        label.Font = Enum.Font.GothamBold
        label.Parent = toggleFrame
        
        if description then
            label.Text = text .. "\n" .. description
            label.TextYAlignment = Enum.TextYAlignment.Top
            label.TextSize = 12
        end
        
        local toggle = Instance.new("TextButton")
        toggle.Name = settingName
        toggle.Size = UDim2.new(0.3, 0, 0.7, 0)
        toggle.Position = UDim2.new(0.68, 0, 0.15, 0)
        toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
        toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
        toggle.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
        toggle.TextSize = 13
        toggle.Font = Enum.Font.GothamBold
        toggle.Parent = toggleFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggle
        
        toggle.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggle.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
            toggle.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
            toggle.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
            
            -- Se for o toggle principal, atualizar o botão ESP
            if settingName == "ENABLED" then
                espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
                espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
                
                -- Atualizar visibilidade de todos os ESPs
                updateAllESPVisibility()
            end
            
            print(text .. ": " .. (ESP_SETTINGS[settingName] and "ATIVADO" or "DESATIVADO"))
        end)
        
        return toggleFrame
    end
    
    -- Criar toggles
    createToggleOption("ESP", "ENABLED", "Ativar/Desativar sistema")
    createToggleOption("Wallhack", "WALLHACK", "Ver através das paredes")
    createToggleOption("Nomes", "SHOW_NAMES", "Mostrar nomes")
    createToggleOption("Distância", "SHOW_DISTANCE", "Mostrar distância")
    createToggleOption("Vida", "SHOW_HEALTH", "Mostrar saúde")
    createToggleOption("Caixa", "SHOW_BOX", "Caixa ao redor")
    createToggleOption("Linha", "SHOW_TRACER", "Linha guia")
    
    -- Atualizar tamanho do canvas
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        toggleContainer.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
    end)
    
    -- Conectar eventos dos botões
    
    -- Botão ESP principal
    espButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        
        -- Atualizar aparência do botão
        espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
        espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
        
        -- Atualizar toggle no painel
        local toggleBtn = settingsPanel:FindFirstChild("ToggleContainer"):FindFirstChild("ENABLED")
        if toggleBtn then
            toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
            toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
            toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
        end
        
        -- Atualizar visibilidade de todos os ESPs
        updateAllESPVisibility()
        
        print("ESP " .. (ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO"))
    end)
    
    -- Botão de configurações
    settingsButton.MouseButton1Click:Connect(function()
        settingsPanel.Visible = not settingsPanel.Visible
        settingsButton.Text = settingsPanel.Visible and "✕" or "⚙"
        settingsButton.BackgroundColor3 = settingsPanel.Visible and Color3.fromRGB(80, 80, 80) or Color3.fromRGB(50, 50, 50)
    end)
    
    -- Mostrar/ocultar botão de configurações ao passar o mouse
    mainButtonFrame.MouseEnter:Connect(function()
        settingsButton.Visible = true
        local tween = TweenService:Create(settingsButton, TweenInfo.new(0.2), {Position = UDim2.new(0, -35, 0.5, -15)})
        tween:Play()
    end)
    
    mainButtonFrame.MouseLeave:Connect(function()
        if not settingsPanel.Visible then
            settingsButton.Visible = false
        end
    end)
    
    -- Fechar painel ao clicar fora
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton1 and not gameProcessed then
            local mousePos = UserInputService:GetMouseLocation()
            
            if settingsPanel.Visible then
                local panelAbsolutePos = settingsPanel.AbsolutePosition
                local panelAbsoluteSize = settingsPanel.AbsoluteSize
                local buttonAbsolutePos = settingsButton.AbsolutePosition
                local buttonAbsoluteSize = settingsButton.AbsoluteSize
                
                -- Verificar se clique foi fora do painel e fora do botão de configurações
                local clickedOnPanel = (mousePos.X >= panelAbsolutePos.X and mousePos.X <= panelAbsolutePos.X + panelAbsoluteSize.X and
                                       mousePos.Y >= panelAbsolutePos.Y and mousePos.Y <= panelAbsolutePos.Y + panelAbsoluteSize.Y)
                
                local clickedOnButton = (mousePos.X >= buttonAbsolutePos.X and mousePos.X <= buttonAbsolutePos.X + buttonAbsoluteSize.X and
                                        mousePos.Y >= buttonAbsolutePos.Y and mousePos.Y <= buttonAbsolutePos.Y + buttonAbsoluteSize.Y)
                
                if not clickedOnPanel and not clickedOnButton then
                    settingsPanel.Visible = false
                    settingsButton.Text = "⚙"
                    settingsButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                end
            end
        end
    end)
    
    return screenGui
end

-- Função para criar um objeto ESP
local function createESPObject(player)
    local espObject = {
        Player = player,
        Box = nil,
        NameLabel = nil,
        DistanceLabel = nil,
        HealthLabel = nil,
        Tracer = nil,
        Connections = {}
    }
    
    -- Criar caixa (Outline)
    if ESP_SETTINGS.SHOW_BOX then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_ESPBox"
        box.Adornee = nil
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Size = Vector3.new(4, 6, 2)
        box.Transparency = 0.3
        box.Visible = ESP_SETTINGS.ENABLED
        box.Parent = espFolder
        
        espObject.Box = box
    end
    
    -- Criar labels para informações
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = player.Name .. "_ESPGui"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = espFolder
    
    if ESP_SETTINGS.SHOW_NAMES then
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "Name"
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.new(1, 1, 1)
        nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        nameLabel.TextStrokeTransparency = 0.5
        nameLabel.TextSize = ESP_SETTINGS.TEXT_SIZE
        nameLabel.Visible = ESP_SETTINGS.ENABLED
        nameLabel.Parent = screenGui
        
        espObject.NameLabel = nameLabel
    end
    
    if ESP_SETTINGS.SHOW_DISTANCE then
        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Name = "Distance"
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.Text = "0m"
        distanceLabel.TextColor3 = Color3.new(1, 1, 1)
        distanceLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        distanceLabel.TextStrokeTransparency = 0.5
        distanceLabel.TextSize = ESP_SETTINGS.TEXT_SIZE - 2
        distanceLabel.Visible = ESP_SETTINGS.ENABLED
        distanceLabel.Parent = screenGui
        
        espObject.DistanceLabel = distanceLabel
    end
    
    if ESP_SETTINGS.SHOW_HEALTH then
        local healthLabel = Instance.new("TextLabel")
        healthLabel.Name = "Health"
        healthLabel.BackgroundTransparency = 1
        healthLabel.Text = "100%"
        healthLabel.TextColor3 = Color3.new(0, 1, 0)
        healthLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        healthLabel.TextStrokeTransparency = 0.5
        healthLabel.TextSize = ESP_SETTINGS.TEXT_SIZE - 2
        healthLabel.Visible = ESP_SETTINGS.ENABLED
        healthLabel.Parent = screenGui
        
        espObject.HealthLabel = healthLabel
    end
    
    -- Criar tracer
    if ESP_SETTINGS.SHOW_TRACER then
        local tracer = Instance.new("Frame")
        tracer.Name = "Tracer"
        tracer.BackgroundColor3 = Color3.new(1, 1, 1)
        tracer.BorderSizePixel = 0
        tracer.Size = UDim2.new(0, 2, 0, 100)
        tracer.Rotation = 0
        tracer.Visible = ESP_SETTINGS.ENABLED
        tracer.Parent = screenGui
        
        espObject.Tracer = tracer
    end
    
    -- Conectar eventos de mudança de personagem
    local function characterAdded(character)
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart", 5)
        local humanoid = character:WaitForChild("Humanoid", 5)
        
        if espObject.Box then
            espObject.Box.Adornee = humanoidRootPart
            espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX
        end
        
        if humanoid and espObject.HealthLabel then
            local function updateHealth()
                local healthPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
                espObject.HealthLabel.Text = healthPercent .. "%"
                
                if healthPercent > 70 then
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                elseif healthPercent > 30 then
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
                else
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                end
            end
            
            table.insert(espObject.Connections, humanoid.HealthChanged:Connect(updateHealth))
            updateHealth()
        end
    end
    
    if player.Character then
        characterAdded(player.Character)
    end
    
    table.insert(espObject.Connections, player.CharacterAdded:Connect(characterAdded))
    
    espCache[player] = espObject
    return espObject
end

-- Função para obter cor baseada no time
local function getPlayerColor(player)
    if ESP_SETTINGS.TEAM_COLOR and player.Team then
        return player.Team.TeamColor.Color
    end
    
    if localPlayer.Team and player.Team then
        if localPlayer.Team == player.Team then
            return ESP_SETTINGS.FRIENDLY_COLOR
        else
            return ESP_SETTINGS.ENEMY_COLOR
        end
    end
    
    return ESP_SETTINGS.NEUTRAL_COLOR
end

-- Função para atualizar o ESP de um jogador
local function updateESP(player, espObject)
    if not ESP_SETTINGS.ENABLED then return end
    
    if not player or not player.Character then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    
    if not humanoidRootPart or not humanoid then return end
    
    local localCharacter = localPlayer.Character
    local localRootPart = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
    
    if not localRootPart then return end
    
    -- Calcular distância
    local distance = (humanoidRootPart.Position - localRootPart.Position).Magnitude
    
    if distance > ESP_SETTINGS.MAX_DISTANCE then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    -- Verificar se está visível (para wallhack)
    local isVisible = true
    if not ESP_SETTINGS.WALLHACK then
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = {localCharacter, character}
        raycastParams.IgnoreWater = true
        
        local raycastResult = workspace:Raycast(
            localRootPart.Position,
            (humanoidRootPart.Position - localRootPart.Position).Unit * distance,
            raycastParams
        )
        
        isVisible = not raycastResult
    end
    
    -- Obter cor
    local color = getPlayerColor(player)
    
    -- Atualizar caixa
    if espObject.Box then
        espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX and isVisible
        espObject.Box.Color3 = color
        espObject.Box.Size = Vector3.new(4, humanoid.HipHeight * 2, 2)
    end
    
    -- Atualizar labels na tela
    local head = character:FindFirstChild("Head")
    if head then
        local screenPosition, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position + Vector3.new(0, 2, 0))
        
        if onScreen then
            -- Nome
            if espObject.NameLabel then
                espObject.NameLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_NAMES and isVisible
                espObject.NameLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                         0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y)
                espObject.NameLabel.TextColor3 = color
            end
            
            -- Distância
            if espObject.DistanceLabel then
                espObject.DistanceLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_DISTANCE and isVisible
                espObject.DistanceLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                             0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 20)
                espObject.DistanceLabel.Text = math.floor(distance) .. "m"
                espObject.DistanceLabel.TextColor3 = color
            end
            
            -- Saúde
            if espObject.HealthLabel then
                espObject.HealthLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_HEALTH and isVisible
                espObject.HealthLabel.Position = UDim2.new(0, screenPosition.X + ESP_SETTINGS.TEXT_OFFSET.X, 
                                                           0, screenPosition.Y + ESP_SETTINGS.TEXT_OFFSET.Y + 40)
            end
            
            -- Tracer
            if espObject.Tracer then
                local rootScreenPosition = workspace.CurrentCamera:WorldToViewportPoint(humanoidRootPart.Position)
                local bottomScreenPosition = Vector2.new(rootScreenPosition.X, rootScreenPosition.Y)
                
                espObject.Tracer.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_TRACER and isVisible
                espObject.Tracer.BackgroundColor3 = color
                espObject.Tracer.Position = UDim2.new(0, bottomScreenPosition.X, 0, bottomScreenPosition.Y)
                espObject.Tracer.Size = UDim2.new(0, ESP_SETTINGS.TRACER_THICKNESS, 0, workspace.CurrentCamera.ViewportSize.Y - bottomScreenPosition.Y)
            end
        else
            if espObject.NameLabel then espObject.NameLabel.Visible = false end
            if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
            if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
            if espObject.Tracer then espObject.Tracer.Visible = false end
        end
    end
end

-- Função principal de atualização
local function updateAllESP()
    if not ESP_SETTINGS.ENABLED then return end
    
    for player, espObject in pairs(espCache) do
        if player ~= localPlayer then
            updateESP(player, espObject)
        end
    end
end

-- Inicializar ESP para jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer then
        createESPObject(player)
    end
end

-- Conectar para novos jogadores
table.insert(connections, Players.PlayerAdded:Connect(function(player)
    if player ~= localPlayer then
        createESPObject(player)
    end
end))

-- Remover ESP quando jogador sair
table.insert(connections, Players.PlayerRemoving:Connect(function(player)
    local espObject = espCache[player]
    if espObject then
        for _, connection in ipairs(espObject.Connections) do
            connection:Disconnect()
        end
        
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
        
        espCache[player] = nil
    end
end))

-- Loop de atualização
table.insert(connections, RunService.RenderStepped:Connect(function()
    updateAllESP()
end))

-- Criar interface
controlGui = createBasicGUI()

-- Tecla para ativar/desativar (Insert)
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Insert then
            ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
            
            -- Atualizar botão na interface
            local espButton = controlGui:FindFirstChild("MainButtonFrame"):FindFirstChild("ESPButton")
            if espButton then
                espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
                espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
            end
            
            -- Atualizar toggle no painel
            local settingsPanel = controlGui:FindFirstChild("MainButtonFrame"):FindFirstChild("SettingsPanel")
            if settingsPanel and settingsPanel.Visible then
                local toggleBtn = settingsPanel:FindFirstChild("ToggleContainer"):FindFirstChild("ENABLED")
                if toggleBtn then
                    toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
                    toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 120, 0) or Color3.fromRGB(120, 0, 0)
                    toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
                end
            end
            
            -- Atualizar visibilidade de todos os ESPs
            updateAllESPVisibility()
            
            print("ESP " .. (ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO"))
        end
    end
end))

-- Limpeza quando o script for destruído
local function cleanup()
    for _, connection in ipairs(connections) do
        connection:Disconnect()
    end
    
    for _, espObject in pairs(espCache) do
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
    end
    
    if controlGui then controlGui:Destroy() end
    espFolder:Destroy()
end

-- Conectar evento de saída
game:GetService("Players").LocalPlayer.AncestryChanged:Connect(function()
    cleanup()
end)

print("ESP/Wallhack script carregado!")
print("Interface criada - Botão ESP no canto superior direito")
print("Tecla Insert - Ativar/Desativar ESP")
print("Passe o mouse sobre o botão para ver opções de configuração")
