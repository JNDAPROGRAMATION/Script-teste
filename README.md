-- Serverscript (LocalScript no StarterPlayerScripts)
-- ESP/Wallhack para Roblox com Janela Dusete Scripts

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
local ContentProvider = game:GetService("ContentProvider")

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}
local mainWindow = nil
local isWindowVisible = true

-- Função para carregar assets
local function preloadAssets()
    -- Tentar carregar algumas imagens (opcional)
    local success, err = pcall(function()
        ContentProvider:PreloadAsync({
            "rbxassetid://7072717364", -- Ícone de olho
            "rbxassetid://7072720867", -- Ícone de engrenagem
            "rbxassetid://7072724779"  -- Ícone de power
        })
    end)
end

-- Função para criar a janela Dusete Scripts
local function createDuseteWindow()
    -- Criar ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DuseteScriptsGUI"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Frame principal da janela
    local mainWindow = Instance.new("Frame")
    mainWindow.Name = "DuseteWindow"
    mainWindow.Size = UDim2.new(0, 300, 0, 400)
    mainWindow.Position = UDim2.new(0.5, -150, 0.5, -200)
    mainWindow.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    mainWindow.BackgroundTransparency = 0.1
    mainWindow.BorderSizePixel = 0
    mainWindow.ClipsDescendants = true
    mainWindow.Parent = screenGui
    
    -- Gradiente de fundo
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 35)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 20))
    })
    gradient.Rotation = 90
    gradient.Parent = mainWindow
    
    -- Arredondar cantos
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 12)
    uiCorner.Parent = mainWindow
    
    -- Sombra exterior
    local shadow = Instance.new("Frame")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 10, 1, 10)
    shadow.Position = UDim2.new(0, -5, 0, -5)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.8
    shadow.BorderSizePixel = 0
    shadow.ZIndex = -1
    shadow.Parent = mainWindow
    
    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, 16)
    shadowCorner.Parent = shadow
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 50)
    titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    titleBar.BackgroundTransparency = 0.2
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainWindow
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar
    
    -- Gradiente na barra de título
    local titleGradient = Instance.new("UIGradient")
    titleGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 40, 60)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 50))
    })
    titleGradient.Rotation = 90
    titleGradient.Parent = titleBar
    
    -- Logo/Ícone
    local logo = Instance.new("TextLabel")
    logo.Name = "Logo"
    logo.Size = UDim2.new(0, 40, 0, 40)
    logo.Position = UDim2.new(0, 10, 0.5, -20)
    logo.Text = "👁️"
    logo.TextColor3 = Color3.fromRGB(0, 200, 255)
    logo.TextSize = 30
    logo.Font = Enum.Font.GothamBold
    logo.BackgroundTransparency = 1
    logo.Parent = titleBar
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(0.6, 0, 1, 0)
    titleLabel.Position = UDim2.new(0, 60, 0, 0)
    titleLabel.Text = "DUSETE SCRIPTS"
    titleLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
    titleLabel.TextSize = 20
    titleLabel.Font = Enum.Font.GothamBlack
    titleLabel.BackgroundTransparency = 1
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Subtítulo
    local subtitle = Instance.new("TextLabel")
    subtitle.Name = "Subtitle"
    subtitle.Size = UDim2.new(0.6, 0, 0, 20)
    subtitle.Position = UDim2.new(0, 60, 0, 30)
    subtitle.Text = "ESP Premium v2.0"
    subtitle.TextColor3 = Color3.fromRGB(150, 200, 255)
    subtitle.TextSize = 12
    subtitle.Font = Enum.Font.GothamMedium
    subtitle.BackgroundTransparency = 1
    subtitle.TextXAlignment = Enum.TextXAlignment.Left
    subtitle.Parent = titleBar
    
    -- Botão de fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -40, 0.5, -15)
    closeButton.Text = "×"
    closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    closeButton.TextSize = 28
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BackgroundColor3 = Color3.fromRGB(40, 20, 20)
    closeButton.BackgroundTransparency = 0.7
    closeButton.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeButton
    
    -- Botão de minimizar
    local minimizeButton = Instance.new("TextButton")
    minimizeButton.Name = "MinimizeButton"
    minimizeButton.Size = UDim2.new(0, 30, 0, 30)
    minimizeButton.Position = UDim2.new(1, -75, 0.5, -15)
    minimizeButton.Text = "—"
    minimizeButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    minimizeButton.TextSize = 24
    minimizeButton.Font = Enum.Font.GothamBold
    minimizeButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    minimizeButton.BackgroundTransparency = 0.7
    minimizeButton.Parent = titleBar
    
    local minimizeCorner = Instance.new("UICorner")
    minimizeCorner.CornerRadius = UDim.new(0, 8)
    minimizeCorner.Parent = minimizeButton
    
    -- Área de conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, -20, 1, -70)
    contentFrame.Position = UDim2.new(0, 10, 0, 60)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainWindow
    
    -- Cartão do ESP (destaque)
    local espCard = Instance.new("Frame")
    espCard.Name = "ESPCard"
    espCard.Size = UDim2.new(1, 0, 0, 100)
    espCard.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    espCard.BackgroundTransparency = 0.3
    espCard.Parent = contentFrame
    
    local cardCorner = Instance.new("UICorner")
    cardCorner.CornerRadius = UDim.new(0, 10)
    cardCorner.Parent = espCard
    
    local cardStroke = Instance.new("UIStroke")
    cardStroke.Color = Color3.fromRGB(0, 150, 255)
    cardStroke.Thickness = 2
    cardStroke.Transparency = 0.5
    cardStroke.Parent = espCard
    
    -- Ícone do ESP
    local espIcon = Instance.new("TextLabel")
    espIcon.Name = "ESPIcon"
    espIcon.Size = UDim2.new(0, 50, 0, 50)
    espIcon.Position = UDim2.new(0, 15, 0.5, -25)
    espIcon.Text = "👁️"
    espIcon.TextColor3 = Color3.fromRGB(0, 200, 255)
    espIcon.TextSize = 36
    espIcon.Font = Enum.Font.GothamBold
    espIcon.BackgroundTransparency = 1
    espIcon.Parent = espCard
    
    -- Título do ESP
    local espTitle = Instance.new("TextLabel")
    espTitle.Name = "ESPTitle"
    espTitle.Size = UDim2.new(0.6, 0, 0, 30)
    espTitle.Position = UDim2.new(0, 75, 0, 20)
    espTitle.Text = "WALLHACK ESP"
    espTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    espTitle.TextSize = 18
    espTitle.Font = Enum.Font.GothamBold
    espTitle.BackgroundTransparency = 1
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.Parent = espCard
    
    -- Status do ESP
    local espStatus = Instance.new("TextLabel")
    espStatus.Name = "ESPStatus"
    espStatus.Size = UDim2.new(0.6, 0, 0, 20)
    espStatus.Position = UDim2.new(0, 75, 0, 50)
    espStatus.Text = "Status: PRONTO"
    espStatus.TextColor3 = Color3.fromRGB(0, 255, 0)
    espStatus.TextSize = 14
    espStatus.Font = Enum.Font.GothamMedium
    espStatus.BackgroundTransparency = 1
    espStatus.TextXAlignment = Enum.TextXAlignment.Left
    espStatus.Parent = espCard
    
    -- Botão principal do ESP
    local espMainButton = Instance.new("TextButton")
    espMainButton.Name = "ESPMainButton"
    espMainButton.Size = UDim2.new(0, 80, 0, 40)
    espMainButton.Position = UDim2.new(1, -90, 0.5, -20)
    espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
    espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(0, 255, 100)
    espMainButton.TextSize = 14
    espMainButton.Font = Enum.Font.GothamBold
    espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
    espMainButton.BackgroundTransparency = 0.3
    espMainButton.Parent = espCard
    
    local espButtonCorner = Instance.new("UICorner")
    espButtonCorner.CornerRadius = UDim.new(0, 8)
    espButtonCorner.Parent = espMainButton
    
    local espButtonStroke = Instance.new("UIStroke")
    espButtonStroke.Color = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
    espButtonStroke.Thickness = 2
    espButtonStroke.Parent = espMainButton
    
    -- Separador
    local separator = Instance.new("Frame")
    separator.Name = "Separator"
    separator.Size = UDim2.new(1, 0, 0, 2)
    separator.Position = UDim2.new(0, 0, 0, 110)
    separator.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    separator.BorderSizePixel = 0
    separator.Parent = contentFrame
    
    -- Container de configurações
    local settingsContainer = Instance.new("ScrollingFrame")
    settingsContainer.Name = "SettingsContainer"
    settingsContainer.Size = UDim2.new(1, 0, 1, -120)
    settingsContainer.Position = UDim2.new(0, 0, 0, 120)
    settingsContainer.BackgroundTransparency = 1
    settingsContainer.BorderSizePixel = 0
    settingsContainer.ScrollBarThickness = 4
    settingsContainer.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
    settingsContainer.CanvasSize = UDim2.new(0, 0, 0, 250)
    settingsContainer.Parent = contentFrame
    
    local settingsLayout = Instance.new("UIListLayout")
    settingsLayout.Padding = UDim.new(0, 10)
    settingsLayout.Parent = settingsContainer
    
    -- Função para criar opção de configuração
    local function createSettingOption(text, settingName, icon)
        local settingFrame = Instance.new("Frame")
        settingFrame.Size = UDim2.new(1, 0, 0, 40)
        settingFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        settingFrame.BackgroundTransparency = 0.4
        settingFrame.Parent = settingsContainer
        
        local settingCorner = Instance.new("UICorner")
        settingCorner.CornerRadius = UDim.new(0, 8)
        settingCorner.Parent = settingFrame
        
        -- Ícone
        if icon then
            local settingIcon = Instance.new("TextLabel")
            settingIcon.Size = UDim2.new(0, 30, 0, 30)
            settingIcon.Position = UDim2.new(0, 10, 0.5, -15)
            settingIcon.Text = icon
            settingIcon.TextColor3 = Color3.fromRGB(0, 200, 255)
            settingIcon.TextSize = 20
            settingIcon.Font = Enum.Font.GothamBold
            settingIcon.BackgroundTransparency = 1
            settingIcon.Parent = settingFrame
        end
        
        -- Texto
        local settingLabel = Instance.new("TextLabel")
        settingLabel.Text = text
        settingLabel.Size = UDim2.new(0.6, -40, 1, 0)
        settingLabel.Position = icon and UDim2.new(0, 50, 0, 0) or UDim2.new(0, 15, 0, 0)
        settingLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        settingLabel.TextXAlignment = Enum.TextXAlignment.Left
        settingLabel.BackgroundTransparency = 1
        settingLabel.TextSize = 14
        settingLabel.Font = Enum.Font.GothamMedium
        settingLabel.Parent = settingFrame
        
        -- Botão de toggle
        local toggleButton = Instance.new("TextButton")
        toggleButton.Name = settingName
        toggleButton.Size = UDim2.new(0, 70, 0, 30)
        toggleButton.Position = UDim2.new(1, -80, 0.5, -15)
        toggleButton.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
        toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
        toggleButton.TextSize = 13
        toggleButton.Font = Enum.Font.GothamBold
        toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
        toggleButton.BackgroundTransparency = 0.3
        toggleButton.Parent = settingFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggleButton
        
        local toggleStroke = Instance.new("UIStroke")
        toggleStroke.Color = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
        toggleStroke.Thickness = 1
        toggleStroke.Parent = toggleButton
        
        toggleButton.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggleButton.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
            toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
            toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
            toggleStroke.Color = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
            
            if settingName == "ENABLED" then
                espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
                espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(0, 255, 100)
                espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
                espMainButtonStroke.Color = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
                espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
                espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
                
                updateAllESPVisibility()
            end
            
            print("[Dusete Scripts] " .. text .. ": " .. (ESP_SETTINGS[settingName] and "ATIVADO" or "DESATIVADO"))
        end)
        
        return settingFrame
    end
    
    -- Criar opções de configuração
    createSettingOption("Wallhack", "WALLHACK", "🧱")
    createSettingOption("Mostrar Nomes", "SHOW_NAMES", "📝")
    createSettingOption("Mostrar Distância", "SHOW_DISTANCE", "📏")
    createSettingOption("Mostrar Vida", "SHOW_HEALTH", "❤️")
    createSettingOption("Caixa de Destaque", "SHOW_BOX", "🔲")
    createSettingOption("Linha Guia", "SHOW_TRACER", "➖")
    
    -- Atualizar tamanho do canvas
    settingsLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        settingsContainer.CanvasSize = UDim2.new(0, 0, 0, settingsLayout.AbsoluteContentSize.Y + 10)
    end)
    
    -- Rodapé
    local footer = Instance.new("Frame")
    footer.Name = "Footer"
    footer.Size = UDim2.new(1, -20, 0, 30)
    footer.Position = UDim2.new(0, 10, 1, -40)
    footer.BackgroundTransparency = 1
    footer.Parent = mainWindow
    
    local footerText = Instance.new("TextLabel")
    footerText.Name = "FooterText"
    footerText.Size = UDim2.new(1, 0, 1, 0)
    footerText.Text = "© 2024 Dusete Scripts • v2.0 • Insert: Toggle ESP"
    footerText.TextColor3 = Color3.fromRGB(150, 150, 200)
    footerText.TextSize = 11
    footerText.Font = Enum.Font.Gotham
    footerText.BackgroundTransparency = 1
    footerText.Parent = footer
    
    -- Conectar eventos
    
    -- Botão principal do ESP
    espMainButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        
        espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
        espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(0, 255, 100)
        espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
        espMainButtonStroke.Color = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
        espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
        espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
        
        -- Atualizar toggle nas configurações
        local toggleBtn = settingsContainer:FindFirstChild("ENABLED")
        if toggleBtn then
            toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
            toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
            toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
        end
        
        updateAllESPVisibility()
        print("[Dusete Scripts] ESP " .. (ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO"))
    end)
    
    -- Botão de fechar
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        isWindowVisible = false
        print("[Dusete Scripts] Janela fechada")
    end)
    
    -- Botão de minimizar
    minimizeButton.MouseButton1Click:Connect(function()
        if isWindowVisible then
            -- Minimizar
            local tween = TweenService:Create(mainWindow, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 300, 0, 50),
                Position = UDim2.new(0.5, -150, 1, -60)
            })
            tween:Play()
            
            contentFrame.Visible = false
            footer.Visible = false
            minimizeButton.Text = "+"
            isWindowVisible = false
        else
            -- Maximizar
            local tween = TweenService:Create(mainWindow, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 300, 0, 400),
                Position = UDim2.new(0.5, -150, 0.5, -200)
            })
            tween:Play()
            
            contentFrame.Visible = true
            footer.Visible = true
            minimizeButton.Text = "—"
            isWindowVisible = true
        end
    end)
    
    -- Tornar a janela arrastável
    local dragging = false
    local dragStart
    local startPos
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = mainWindow.Position
        end
    end)
    
    titleBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainWindow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, 
                                           startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    
    -- Efeito de entrada
    mainWindow.Position = UDim2.new(0.5, -150, 0.5, -250)
    mainWindow.BackgroundTransparency = 1
    titleBar.BackgroundTransparency = 1
    contentFrame.Visible = false
    
    local entranceTween = TweenService:Create(mainWindow, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -150, 0.5, -200),
        BackgroundTransparency = 0.1
    })
    
    local titleTween = TweenService:Create(titleBar, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundTransparency = 0.2
    })
    
    entranceTween:Play()
    titleTween:Play()
    
    entranceTween.Completed:Connect(function()
        contentFrame.Visible = true
        local fadeIn = TweenService:Create(contentFrame, TweenInfo.new(0.3), {
            BackgroundTransparency = 0
        })
        fadeIn:Play()
    end)
    
    return screenGui
end

-- Função para atualizar todos os ESPs visuais
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
    
    if not player or not player
