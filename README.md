-- Serverscript (LocalScript no StarterPlayerScripts)
-- ESP/Wallhack e Aimbot para Roblox

-- Configurações
local ESP_SETTINGS = {
    ENABLED = true,
    WALLHACK = true,
    SHOW_NAMES = true,
    SHOW_DISTANCE = true,
    SHOW_HEALTH = true,
    SHOW_BOX = true,
    SHOW_TRACER = true,
    
    -- Configurações do Aimbot
    AIMBOT_ENABLED = false, -- Aimbot começa desativado
    AIMBOT_KEY = Enum.KeyCode.LeftAlt, -- Tecla para ativar aimbot (segurar)
    AIMBOT_SMOOTHNESS = 5, -- Suavidade do movimento (1 = instantâneo, 10 = muito suave)
    AIMBOT_MAX_DISTANCE = 500, -- Distância máxima para o aimbot
    AIMBOT_FOV = 100, -- Campo de visão do aimbot em graus
    AIMBOT_TARGET_PART = "Head", -- Parte do corpo para mirar
    AIMBOT_PRIORITY = "Closest", -- Closest, LowestHealth, Crosshair
    
    -- Cores
    TEAM_COLOR = true,
    FRIENDLY_COLOR = Color3.fromRGB(0, 255, 0),
    ENEMY_COLOR = Color3.fromRGB(255, 0, 0),
    NEUTRAL_COLOR = Color3.fromRGB(255, 255, 0),
    
    -- Tamanhos
    BOX_THICKNESS = 1,
    TEXT_SIZE = 18,
    TRACER_THICKNESS = 1,
    
    -- Posições
    TEXT_OFFSET = Vector2.new(0, -40),
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ContextActionService = game:GetService("ContextActionService")
local Camera = workspace.CurrentCamera

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}
local controlGui = nil

-- Variáveis do Aimbot
local aimbotTarget = nil
local isAimbotKeyDown = false
local aimbotConnection = nil
local fovCircle = nil

-- Função para verificar se um jogador é válido para o aimbot
local function isValidTarget(player)
    if player == localPlayer then return false end
    if not player.Character then return false end
    
    local character = player.Character
    local humanoid = character:FindFirstChild("Humanoid")
    local targetPart = character:FindFirstChild(ESP_SETTINGS.AIMBOT_TARGET_PART)
    
    if not humanoid or humanoid.Health <= 0 then return false end
    if not targetPart then return false end
    
    -- Verificar time
    if localPlayer.Team and player.Team then
        if localPlayer.Team == player.Team then
            return false -- Não mirar em aliados
        end
    end
    
    -- Verificar distância
    local localCharacter = localPlayer.Character
    local localRootPart = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
    if not localRootPart then return false end
    
    local distance = (targetPart.Position - localRootPart.Position).Magnitude
    if distance > ESP_SETTINGS.AIMBOT_MAX_DISTANCE then return false end
    
    return true, character, targetPart, distance
end

-- Função para obter o melhor alvo
local function getBestTarget()
    local bestTarget = nil
    local bestScore = math.huge
    local bestDistance = math.huge
    local bestHealth = math.huge
    
    local localCharacter = localPlayer.Character
    local localHead = localCharacter and localCharacter:FindFirstChild("Head")
    if not localHead then return nil end
    
    local cameraDirection = Camera.CFrame.LookVector
    local cameraPosition = Camera.CFrame.Position
    
    for _, player in pairs(Players:GetPlayers()) do
        local isValid, character, targetPart, distance = isValidTarget(player)
        
        if isValid then
            -- Calcular direção para o alvo
            local directionToTarget = (targetPart.Position - cameraPosition).Unit
            
            -- Calcular ângulo entre a mira atual e o alvo
            local dotProduct = cameraDirection:Dot(directionToTarget)
            local angle = math.deg(math.acos(math.clamp(dotProduct, -1, 1)))
            
            -- Verificar se está dentro do FOV
            if angle <= (ESP_SETTINGS.AIMBOT_FOV / 2) then
                -- Calcular score baseado na prioridade
                local score = distance
                
                if ESP_SETTINGS.AIMBOT_PRIORITY == "LowestHealth" then
                    local humanoid = character:FindFirstChild("Humanoid")
                    local healthPercent = humanoid.Health / humanoid.MaxHealth
                    score = healthPercent * 1000 + distance * 0.01
                elseif ESP_SETTINGS.AIMBOT_PRIORITY == "Crosshair" then
                    score = angle
                end
                
                if score < bestScore then
                    bestScore = score
                    bestTarget = player
                    bestDistance = distance
                end
            end
        end
    end
    
    return bestTarget, bestDistance
end

-- Função para suavizar o movimento do mouse
local function smoothAim(targetPosition)
    local mouse = localPlayer:GetMouse()
    local targetScreenPosition, onScreen = Camera:WorldToViewportPoint(targetPosition)
    
    if not onScreen then return end
    
    local currentMousePosition = Vector2.new(mouse.X, mouse.Y)
    local targetPosition2D = Vector2.new(targetScreenPosition.X, targetScreenPosition.Y)
    
    -- Calcular direção
    local direction = targetPosition2D - currentMousePosition
    
    -- Aplicar suavidade
    local smoothDirection = direction / ESP_SETTINGS.AIMBOT_SMOOTHNESS
    
    -- Mover mouse
    mousemoverel(smoothDirection.X, smoothDirection.Y)
end

-- Função principal do aimbot
local function aimbotUpdate()
    if not ESP_SETTINGS.AIMBOT_ENABLED or not isAimbotKeyDown then
        if aimbotTarget then
            aimbotTarget = nil
        end
        return
    end
    
    local targetPlayer, distance = getBestTarget()
    
    if targetPlayer and targetPlayer.Character then
        local targetPart = targetPlayer.Character:FindFirstChild(ESP_SETTINGS.AIMBOT_TARGET_PART)
        if targetPart then
            aimbotTarget = targetPlayer
            smoothAim(targetPart.Position)
        else
            aimbotTarget = nil
        end
    else
        aimbotTarget = nil
    end
end

-- Função para criar o círculo de FOV
local function createFOVCircle()
    if fovCircle then fovCircle:Destroy() end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FOVCircle"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundTransparency = 1
    frame.Parent = screenGui
    
    -- Criar círculo de FOV
    local circle = Instance.new("ImageLabel")
    circle.Name = "Circle"
    circle.Size = UDim2.new(0, ESP_SETTINGS.AIMBOT_FOV * 2, 0, ESP_SETTINGS.AIMBOT_FOV * 2)
    circle.Position = UDim2.new(0.5, -ESP_SETTINGS.AIMBOT_FOV, 0.5, -ESP_SETTINGS.AIMBOT_FOV)
    circle.BackgroundTransparency = 1
    circle.Image = "rbxassetid://3570695787"
    circle.ImageColor3 = Color3.fromRGB(255, 50, 50)
    circle.ImageTransparency = 0.7
    circle.Visible = false
    circle.Parent = frame
    
    fovCircle = screenGui
    return screenGui
end

-- Atualizar todos os ESPs visuais
local function updateAllESPVisibility()
    for _, espObject in pairs(espCache) do
        if espObject.Box then 
            espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX 
        end
        if espObject.NameLabel then 
            espObject.NameLabel.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_NAMES 
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

-- Criar interface melhorada com botão de Aimbot
local function createEnhancedGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESPEnhancedGUI"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 100, 0, 100)
    mainFrame.Position = UDim2.new(1, -110, 0, 20)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.BackgroundTransparency = 0.2
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 10)
    uiCorner.Parent = mainFrame
    
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(80, 80, 80)
    uiStroke.Thickness = 2
    uiStroke.Parent = mainFrame
    
    -- Container de botões
    local buttonContainer = Instance.new("Frame")
    buttonContainer.Name = "ButtonContainer"
    buttonContainer.Size = UDim2.new(1, -10, 1, -10)
    buttonContainer.Position = UDim2.new(0, 5, 0, 5)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.Parent = mainFrame
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 5)
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.Parent = buttonContainer
    
    -- Botão ESP
    local espButton = Instance.new("TextButton")
    espButton.Name = "ESPButton"
    espButton.Size = UDim2.new(1, 0, 0, 40)
    espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
    espButton.BackgroundTransparency = 0.3
    espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
    espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
    espButton.TextSize = 16
    espButton.Font = Enum.Font.GothamBold
    espButton.Parent = buttonContainer
    
    local espCorner = Instance.new("UICorner")
    espCorner.CornerRadius = UDim.new(0, 8)
    espCorner.Parent = espButton
    
    -- Botão Aimbot
    local aimbotButton = Instance.new("TextButton")
    aimbotButton.Name = "AimbotButton"
    aimbotButton.Size = UDim2.new(1, 0, 0, 40)
    aimbotButton.BackgroundColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
    aimbotButton.BackgroundTransparency = 0.3
    aimbotButton.Text = ESP_SETTINGS.AIMBOT_ENABLED and "AIMBOT: ON" or "AIMBOT: OFF"
    aimbotButton.TextColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
    aimbotButton.TextSize = 16
    aimbotButton.Font = Enum.Font.GothamBold
    aimbotButton.Parent = buttonContainer
    
    local aimbotCorner = Instance.new("UICorner")
    aimbotCorner.CornerRadius = UDim.new(0, 8)
    aimbotCorner.Parent = aimbotButton
    
    -- Botão de configurações
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
    settingsButton.Parent = mainFrame
    
    local settingsCorner = Instance.new("UICorner")
    settingsCorner.CornerRadius = UDim.new(0, 6)
    settingsCorner.Parent = settingsButton
    
    -- Painel de configurações
    local settingsPanel = Instance.new("Frame")
    settingsPanel.Name = "SettingsPanel"
    settingsPanel.Size = UDim2.new(0, 220, 0, 400)
    settingsPanel.Position = UDim2.new(0, -225, 0, 0)
    settingsPanel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    settingsPanel.BackgroundTransparency = 0.1
    settingsPanel.BorderSizePixel = 0
    settingsPanel.Visible = false
    settingsPanel.Parent = mainFrame
    
    local panelCorner = Instance.new("UICorner")
    panelCorner.CornerRadius = UDim.new(0, 10)
    panelCorner.Parent = settingsPanel
    
    local panelStroke = Instance.new("UIStroke")
    panelStroke.Color = Color3.fromRGB(60, 60, 60)
    panelStroke.Thickness = 2
    panelStroke.Parent = settingsPanel
    
    -- Título
    local panelTitle = Instance.new("TextLabel")
    panelTitle.Name = "Title"
    panelTitle.Size = UDim2.new(1, 0, 0, 40)
    panelTitle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    panelTitle.BackgroundTransparency = 0.5
    panelTitle.Text = "CONFIGURAÇÕES"
    panelTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    panelTitle.TextSize = 16
    panelTitle.Font = Enum.Font.GothamBold
    panelTitle.Parent = settingsPanel
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = panelTitle
    
    -- Container com abas
    local tabContainer = Instance.new("Frame")
    tabContainer.Name = "TabContainer"
    tabContainer.Size = UDim2.new(1, 0, 0, 30)
    tabContainer.Position = UDim2.new(0, 0, 0, 40)
    tabContainer.BackgroundTransparency = 1
    tabContainer.Parent = settingsPanel
    
    -- Abas
    local espTab = Instance.new("TextButton")
    espTab.Name = "ESPTab"
    espTab.Size = UDim2.new(0.5, 0, 1, 0)
    espTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    espTab.BackgroundTransparency = 0.5
    espTab.Text = "ESP"
    espTab.TextColor3 = Color3.fromRGB(255, 255, 255)
    espTab.TextSize = 14
    espTab.Font = Enum.Font.GothamBold
    espTab.Parent = tabContainer
    
    local aimbotTab = Instance.new("TextButton")
    aimbotTab.Name = "AimbotTab"
    aimbotTab.Size = UDim2.new(0.5, 0, 1, 0)
    aimbotTab.Position = UDim2.new(0.5, 0, 0, 0)
    aimbotTab.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    aimbotTab.BackgroundTransparency = 0.7
    aimbotTab.Text = "AIMBOT"
    aimbotTab.TextColor3 = Color3.fromRGB(200, 200, 200)
    aimbotTab.TextSize = 14
    aimbotTab.Font = Enum.Font.GothamBold
    aimbotTab.Parent = tabContainer
    
    -- Containers de conteúdo
    local contentContainer = Instance.new("ScrollingFrame")
    contentContainer.Name = "ContentContainer"
    contentContainer.Size = UDim2.new(1, -10, 1, -80)
    contentContainer.Position = UDim2.new(0, 5, 0, 75)
    contentContainer.BackgroundTransparency = 1
    contentContainer.BorderSizePixel = 0
    contentContainer.ScrollBarThickness = 4
    contentContainer.CanvasSize = UDim2.new(0, 0, 0, 500)
    contentContainer.Parent = settingsPanel
    
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Padding = UDim.new(0, 8)
    contentLayout.Parent = contentContainer
    
    -- Função para criar toggles
    local function createToggleOption(text, settingName, description)
        local toggleFrame = Instance.new("Frame")
        toggleFrame.Size = UDim2.new(1, 0, 0, 35)
        toggleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        toggleFrame.BackgroundTransparency = 0.5
        toggleFrame.Parent = contentContainer
        
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
            
            -- Atualizar botões principais
            if settingName == "ENABLED" then
                espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
                espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
                updateAllESPVisibility()
            elseif settingName == "AIMBOT_ENABLED" then
                aimbotButton.Text = ESP_SETTINGS.AIMBOT_ENABLED and "AIMBOT: ON" or "AIMBOT: OFF"
                aimbotButton.TextColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                aimbotButton.BackgroundColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
                
                -- Atualizar visibilidade do círculo de FOV
                if fovCircle then
                    fovCircle:FindFirstChild("Circle").Visible = ESP_SETTINGS.AIMBOT_ENABLED
                end
            end
        end)
        
        return toggleFrame
    end
    
    -- Função para criar slider
    local function createSliderOption(text, settingName, min, max, step)
        local sliderFrame = Instance.new("Frame")
        sliderFrame.Size = UDim2.new(1, 0, 0, 60)
        sliderFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        sliderFrame.BackgroundTransparency = 0.5
        sliderFrame.Parent = contentContainer
        
        local frameCorner = Instance.new("UICorner")
        frameCorner.CornerRadius = UDim.new(0, 6)
        frameCorner.Parent = sliderFrame
        
        local label = Instance.new("TextLabel")
        label.Text = text .. ": " .. ESP_SETTINGS[settingName]
        label.Size = UDim2.new(1, -20, 0, 20)
        label.Position = UDim2.new(0, 10, 0, 5)
        label.TextColor3 = Color3.fromRGB(255, 255, 255)
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.BackgroundTransparency = 1
        label.TextSize = 14
        label.Font = Enum.Font.GothamBold
        label.Name = "Label"
        label.Parent = sliderFrame
        
        local slider = Instance.new("Frame")
        slider.Name = "Slider"
        slider.Size = UDim2.new(1, -20, 0, 20)
        slider.Position = UDim2.new(0, 10, 0, 30)
        slider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        slider.Parent = sliderFrame
        
        local sliderCorner = Instance.new("UICorner")
        sliderCorner.CornerRadius = UDim.new(0, 10)
        sliderCorner.Parent = slider
        
        local fill = Instance.new("Frame")
        fill.Name = "Fill"
        fill.Size = UDim2.new((ESP_SETTINGS[settingName] - min) / (max - min), 0, 1, 0)
        fill.Position = UDim2.new(0, 0, 0, 0)
        fill.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
        fill.Parent = slider
        
        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(0, 10)
        fillCorner.Parent = fill
        
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, 0, 1, 0)
        button.BackgroundTransparency = 1
        button.Text = ""
        button.Parent = slider
        
        local isDragging = false
        
        button.MouseButton1Down:Connect(function()
            isDragging = true
        end)
        
        UserInputService.InputChanged:Connect(function(input)
            if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                local relativeX = (input.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X
                relativeX = math.clamp(relativeX, 0, 1)
                
                local value = min + (max - min) * relativeX
                value = math.floor(value / step) * step
                
                ESP_SETTINGS[settingName] = value
                label.Text = text .. ": " .. value
                fill.Size = UDim2.new((value - min) / (max - min), 0, 1, 0)
                
                -- Se for o FOV, atualizar o círculo
                if settingName == "AIMBOT_FOV" and fovCircle then
                    local circle = fovCircle:FindFirstChild("Circle")
                    if circle then
                        circle.Size = UDim2.new(0, value * 2, 0, value * 2)
                        circle.Position = UDim2.new(0.5, -value, 0.5, -value)
                    end
                end
            end
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                isDragging = false
            end
        end)
        
        return sliderFrame
    end
    
    -- Função para mostrar conteúdo das abas
    local function showESPTab()
        espTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        espTab.BackgroundTransparency = 0.5
        espTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        
        aimbotTab.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        aimbotTab.BackgroundTransparency = 0.7
        aimbotTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        
        -- Limpar conteúdo
        for _, child in ipairs(contentContainer:GetChildren()) do
            if child:IsA("Frame") then
                child:Destroy()
            end
        end
        
        -- Adicionar opções do ESP
        createToggleOption("ESP", "ENABLED", "Ativar/Desativar")
        createToggleOption("Wallhack", "WALLHACK", "Ver através das paredes")
        createToggleOption("Nomes", "SHOW_NAMES")
        createToggleOption("Distância", "SHOW_DISTANCE")
        createToggleOption("Vida", "SHOW_HEALTH")
        createToggleOption("Caixa", "SHOW_BOX")
        createToggleOption("Linha", "SHOW_TRACER")
        createSliderOption("Distância Máx", "MAX_DISTANCE", 100, 5000, 100)
    end
    
    local function showAimbotTab()
        espTab.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        espTab.BackgroundTransparency = 0.7
        espTab.TextColor3 = Color3.fromRGB(200, 200, 200)
        
        aimbotTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        aimbotTab.BackgroundTransparency = 0.5
        aimbotTab.TextColor3 = Color3.fromRGB(255, 255, 255)
        
        -- Limpar conteúdo
        for _, child in ipairs(contentContainer:GetChildren()) do
            if child:IsA("Frame") then
                child:Destroy()
            end
        end
        
        -- Adicionar opções do Aimbot
        createToggleOption("Aimbot", "AIMBOT_ENABLED", "Ativar sistema de mira")
        createToggleOption("FOV Circle", "AIMBOT_FOV_VISIBLE", "Mostrar círculo de FOV")
        createSliderOption("Suavidade", "AIMBOT_SMOOTHNESS", 1, 20, 1)
        createSliderOption("Distância Máx", "AIMBOT_MAX_DISTANCE", 100, 1000, 50)
        createSliderOption("FOV", "AIMBOT_FOV", 10, 360, 10)
    end
    
    -- Inicializar com aba ESP
    showESPTab()
    
    -- Conectar abas
    espTab.MouseButton1Click:Connect(showESPTab)
    aimbotTab.MouseButton1Click:Connect(showAimbotTab)
    
    -- Atualizar tamanho do canvas
    contentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        contentContainer.CanvasSize = UDim2.new(0, 0, 0, contentLayout.AbsoluteContentSize.Y + 10)
    end)
    
    -- Eventos dos botões principais
    espButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        
        espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
        espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
        
        updateAllESPVisibility()
    end)
    
    aimbotButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.AIMBOT_ENABLED = not ESP_SETTINGS.AIMBOT_ENABLED
        
        aimbotButton.Text = ESP_SETTINGS.AIMBOT_ENABLED and "AIMBOT: ON" or "AIMBOT: OFF"
        aimbotButton.TextColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
        aimbotButton.BackgroundColor3 = ESP_SETTINGS.AIMBOT_ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
        
        -- Mostrar/ocultar círculo de FOV
        if fovCircle then
            fovCircle:FindFirstChild("Circle").Visible = ESP_SETTINGS.AIMBOT_ENABLED
        end
        
        print("Aimbot " .. (ESP_SETTINGS.AIMBOT_ENABLED and "ATIVADO" or "DESATIVADO"))
    end)
    
    -- Botão de configurações
    settingsButton.MouseButton1Click:Connect(function()
        settingsPanel.Visible = not settingsPanel.Visible
        settingsButton.Text = settingsPanel.Visible and "✕" or "⚙"
        settingsButton.BackgroundColor3 = settingsPanel.Visible and Color3.fromRGB(80, 80, 80) or Color3.fromRGB(50, 50, 50)
    end)
    
    -- Mostrar/ocultar botão de configurações
    mainFrame.MouseEnter:Connect(function()
        settingsButton.Visible = true
        local tween = TweenService:Create(settingsButton, TweenInfo.new(0.2), {Position = UDim2.new(0, -35, 0.5, -15)})
        tween:Play()
    end)
    
    mainFrame.MouseLeave:Connect(function()
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

-- As funções createESPObject, getPlayerColor, updateESP, e updateAllESP permanecem as mesmas do seu código original
-- [Inserir aqui as funções do seu código original que não foram modificadas]

-- Inicializar sistema
local function initialize()
    -- Criar interface
    controlGui = createEnhancedGUI()
    
    -- Criar círculo de FOV
    createFOVCircle()
    if fovCircle then
        fovCircle:FindFirstChild("Circle").Visible = ESP_SETTINGS.AIMBOT_ENABLED
    end
    
    -- Inicializar ESP para jogadores existentes
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer then
            createESPObject(player)
        end
    end
    
    -- Conectar eventos
    table.insert(connections, Players.PlayerAdded:Connect(function(player)
        if player ~= localPlayer then
            createESPObject(player)
        end
    end))
    
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
    
    -- Loop de atualização do ESP
    table.insert(connections, RunService.RenderStepped:Connect(function()
        updateAllESP()
    end))
    
    -- Eventos do Aimbot
    table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if not gameProcessed then
            -- Tecla Insert para ESP
            if input.KeyCode == Enum.KeyCode.Insert then
                ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
                
                local espButton = controlGui:FindFirstChild("MainFrame"):FindFirstChild("ButtonContainer"):FindFirstChild("ESPButton")
                if espButton then
                    espButton.Text = ESP_SETTINGS.ENABLED and "ESP: ON" or "ESP: OFF"
                    espButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 50, 50)
                    espButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
                end
                
                updateAllESPVisibility()
            end
            
            -- Tecla do aimbot
            if input.KeyCode == ESP_SETTINGS.AIMBOT_KEY then
                isAimbotKeyDown = true
            end
        end
    end))
    
    table.insert(connections, UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.KeyCode == ESP_SETTINGS.AIMBOT_KEY then
            isAimbotKeyDown = false
            aimbotTarget = nil
        end
    end))
    
    -- Loop do Aimbot
    aimbotConnection = RunService.RenderStepped:Connect(aimbotUpdate)
    table.insert(connections, aimbotConnection)
end

-- Iniciar
initialize()

print("ESP + Aimbot carregado!")
print("Teclas:")
print("  Insert - Ativar/Desativar ESP")
print("  " .. ESP_SETTINGS.AIMBOT_KEY.Name .. " - Segurar para usar Aimbot")
print("Interface com botões de controle no canto superior direito")
