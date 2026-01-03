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
local Workspace = game:GetService("Workspace")

-- Variáveis
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")
local espFolder = Instance.new("Folder")
espFolder.Name = "ESPFolder"
espFolder.Parent = playerGui

local espCache = {}
local connections = {}
local duseteWindow = nil
local isWindowVisible = true

-- Função para criar a janela Dusete Scripts
local function createDuseteWindow()
    -- Criar ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "DuseteScriptsGUI"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 999
    screenGui.Parent = playerGui
    
    -- Frame principal da janela
    local mainWindow = Instance.new("Frame")
    mainWindow.Name = "DuseteWindow"
    mainWindow.Size = UDim2.new(0, 320, 0, 420)
    mainWindow.Position = UDim2.new(0.5, -160, 0.5, -210)
    mainWindow.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
    mainWindow.BackgroundTransparency = 0.1
    mainWindow.BorderSizePixel = 0
    mainWindow.ClipsDescendants = true
    mainWindow.Visible = true
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
    uiCorner.CornerRadius = UDim.new(0, 14)
    uiCorner.Parent = mainWindow
    
    -- Sombra exterior
    local shadow = Instance.new("Frame")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 12, 1, 12)
    shadow.Position = UDim2.new(0, -6, 0, -6)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.85
    shadow.BorderSizePixel = 0
    shadow.ZIndex = -1
    shadow.Parent = mainWindow
    
    local shadowCorner = Instance.new("UICorner")
    shadowCorner.CornerRadius = UDim.new(0, 18)
    shadowCorner.Parent = shadow
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 55)
    titleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    titleBar.BackgroundTransparency = 0.2
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainWindow
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 14)
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
    logo.Size = UDim2.new(0, 45, 0, 45)
    logo.Position = UDim2.new(0, 12, 0.5, -22.5)
    logo.Text = "👁️"
    logo.TextColor3 = Color3.fromRGB(0, 200, 255)
    logo.TextSize = 32
    logo.Font = Enum.Font.GothamBold
    logo.BackgroundTransparency = 1
    logo.Parent = titleBar
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(0.6, 0, 0.5, 0)
    titleLabel.Position = UDim2.new(0, 65, 0, 8)
    titleLabel.Text = "DUSETE SCRIPTS"
    titleLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
    titleLabel.TextSize = 22
    titleLabel.Font = Enum.Font.GothamBlack
    titleLabel.BackgroundTransparency = 1
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Subtítulo
    local subtitle = Instance.new("TextLabel")
    subtitle.Name = "Subtitle"
    subtitle.Size = UDim2.new(0.6, 0, 0, 20)
    subtitle.Position = UDim2.new(0, 65, 0, 35)
    subtitle.Text = "ESP Premium v2.0"
    subtitle.TextColor3 = Color3.fromRGB(150, 200, 255)
    subtitle.TextSize = 13
    subtitle.Font = Enum.Font.GothamMedium
    subtitle.BackgroundTransparency = 1
    subtitle.TextXAlignment = Enum.TextXAlignment.Left
    subtitle.Parent = titleBar
    
    -- Botão de fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0, 35, 0, 35)
    closeButton.Position = UDim2.new(1, -42, 0.5, -17.5)
    closeButton.Text = "×"
    closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    closeButton.TextSize = 32
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
    minimizeButton.Size = UDim2.new(0, 35, 0, 35)
    minimizeButton.Position = UDim2.new(1, -82, 0.5, -17.5)
    minimizeButton.Text = "—"
    minimizeButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    minimizeButton.TextSize = 28
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
    contentFrame.Size = UDim2.new(1, -20, 1, -75)
    contentFrame.Position = UDim2.new(0, 10, 0, 65)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Visible = true
    contentFrame.Parent = mainWindow
    
    -- Cartão do ESP (destaque)
    local espCard = Instance.new("Frame")
    espCard.Name = "ESPCard"
    espCard.Size = UDim2.new(1, 0, 0, 110)
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
    espIcon.Size = UDim2.new(0, 55, 0, 55)
    espIcon.Position = UDim2.new(0, 15, 0.5, -27.5)
    espIcon.Text = "👁️"
    espIcon.TextColor3 = Color3.fromRGB(0, 200, 255)
    espIcon.TextSize = 40
    espIcon.Font = Enum.Font.GothamBold
    espIcon.BackgroundTransparency = 1
    espIcon.Parent = espCard
    
    -- Título do ESP
    local espTitle = Instance.new("TextLabel")
    espTitle.Name = "ESPTitle"
    espTitle.Size = UDim2.new(0.6, 0, 0, 30)
    espTitle.Position = UDim2.new(0, 80, 0, 25)
    espTitle.Text = "WALLHACK ESP"
    espTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    espTitle.TextSize = 20
    espTitle.Font = Enum.Font.GothamBold
    espTitle.BackgroundTransparency = 1
    espTitle.TextXAlignment = Enum.TextXAlignment.Left
    espTitle.Parent = espCard
    
    -- Status do ESP
    local espStatus = Instance.new("TextLabel")
    espStatus.Name = "ESPStatus"
    espStatus.Size = UDim2.new(0.6, 0, 0, 22)
    espStatus.Position = UDim2.new(0, 80, 0, 55)
    espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
    espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
    espStatus.TextSize = 14
    espStatus.Font = Enum.Font.GothamMedium
    espStatus.BackgroundTransparency = 1
    espStatus.TextXAlignment = Enum.TextXAlignment.Left
    espStatus.Parent = espCard
    
    -- Botão principal do ESP
    local espMainButton = Instance.new("TextButton")
    espMainButton.Name = "ESPMainButton"
    espMainButton.Size = UDim2.new(0, 90, 0, 45)
    espMainButton.Position = UDim2.new(1, -100, 0.5, -22.5)
    espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
    espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(0, 255, 150)
    espMainButton.TextSize = 16
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
    
    -- Efeito de brilho no hover
    local espButtonGlow = Instance.new("Frame")
    espButtonGlow.Name = "Glow"
    espButtonGlow.Size = UDim2.new(1, 4, 1, 4)
    espButtonGlow.Position = UDim2.new(0, -2, 0, -2)
    espButtonGlow.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(50, 255, 50)
    espButtonGlow.BackgroundTransparency = 0.9
    espButtonGlow.BorderSizePixel = 0
    espButtonGlow.ZIndex = -1
    espButtonGlow.Parent = espMainButton
    
    local glowCorner = Instance.new("UICorner")
    glowCorner.CornerRadius = UDim.new(0, 10)
    glowCorner.Parent = espButtonGlow
    
    -- Separador
    local separator = Instance.new("Frame")
    separator.Name = "Separator"
    separator.Size = UDim2.new(1, 0, 0, 2)
    separator.Position = UDim2.new(0, 0, 0, 120)
    separator.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    separator.BorderSizePixel = 0
    separator.Parent = contentFrame
    
    -- Título das configurações
    local settingsTitle = Instance.new("TextLabel")
    settingsTitle.Name = "SettingsTitle"
    settingsTitle.Size = UDim2.new(1, 0, 0, 30)
    settingsTitle.Position = UDim2.new(0, 0, 0, 125)
    settingsTitle.Text = "CONFIGURAÇÕES"
    settingsTitle.TextColor3 = Color3.fromRGB(0, 180, 255)
    settingsTitle.TextSize = 16
    settingsTitle.Font = Enum.Font.GothamBold
    settingsTitle.BackgroundTransparency = 1
    settingsTitle.TextXAlignment = Enum.TextXAlignment.Left
    settingsTitle.Parent = contentFrame
    
    -- Container de configurações
    local settingsContainer = Instance.new("ScrollingFrame")
    settingsContainer.Name = "SettingsContainer"
    settingsContainer.Size = UDim2.new(1, 0, 1, -160)
    settingsContainer.Position = UDim2.new(0, 0, 0, 155)
    settingsContainer.BackgroundTransparency = 1
    settingsContainer.BorderSizePixel = 0
    settingsContainer.ScrollBarThickness = 4
    settingsContainer.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
    settingsContainer.CanvasSize = UDim2.new(0, 0, 0, 280)
    settingsContainer.Parent = contentFrame
    
    local settingsLayout = Instance.new("UIListLayout")
    settingsLayout.Padding = UDim.new(0, 12)
    settingsLayout.Parent = settingsContainer
    
    -- Função para criar opção de configuração
    local function createSettingOption(text, settingName, icon)
        local settingFrame = Instance.new("Frame")
        settingFrame.Size = UDim2.new(1, 0, 0, 45)
        settingFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        settingFrame.BackgroundTransparency = 0.4
        settingFrame.Parent = settingsContainer
        
        local settingCorner = Instance.new("UICorner")
        settingCorner.CornerRadius = UDim.new(0, 8)
        settingCorner.Parent = settingFrame
        
        -- Ícone
        local settingIcon = Instance.new("TextLabel")
        settingIcon.Size = UDim2.new(0, 35, 0, 35)
        settingIcon.Position = UDim2.new(0, 10, 0.5, -17.5)
        settingIcon.Text = icon or "⚙"
        settingIcon.TextColor3 = Color3.fromRGB(0, 200, 255)
        settingIcon.TextSize = 22
        settingIcon.Font = Enum.Font.GothamBold
        settingIcon.BackgroundTransparency = 1
        settingIcon.Parent = settingFrame
        
        -- Texto
        local settingLabel = Instance.new("TextLabel")
        settingLabel.Text = text
        settingLabel.Size = UDim2.new(0.6, -45, 1, 0)
        settingLabel.Position = UDim2.new(0, 55, 0, 0)
        settingLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        settingLabel.TextXAlignment = Enum.TextXAlignment.Left
        settingLabel.BackgroundTransparency = 1
        settingLabel.TextSize = 15
        settingLabel.Font = Enum.Font.GothamMedium
        settingLabel.Parent = settingFrame
        
        -- Botão de toggle
        local toggleButton = Instance.new("TextButton")
        toggleButton.Name = settingName
        toggleButton.Size = UDim2.new(0, 75, 0, 32)
        toggleButton.Position = UDim2.new(1, -85, 0.5, -16)
        toggleButton.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
        toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 120, 120)
        toggleButton.TextSize = 14
        toggleButton.Font = Enum.Font.GothamBold
        toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
        toggleButton.BackgroundTransparency = 0.3
        toggleButton.Parent = settingFrame
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(0, 6)
        toggleCorner.Parent = toggleButton
        
        local toggleStroke = Instance.new("UIStroke")
        toggleStroke.Color = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
        toggleStroke.Thickness = 2
        toggleStroke.Parent = toggleButton
        
        -- Efeito de brilho no toggle
        local toggleGlow = Instance.new("Frame")
        toggleGlow.Size = UDim2.new(1, 4, 1, 4)
        toggleGlow.Position = UDim2.new(0, -2, 0, -2)
        toggleGlow.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
        toggleGlow.BackgroundTransparency = 0.9
        toggleGlow.BorderSizePixel = 0
        toggleGlow.ZIndex = -1
        toggleGlow.Parent = toggleButton
        
        local toggleGlowCorner = Instance.new("UICorner")
        toggleGlowCorner.CornerRadius = UDim.new(0, 8)
        toggleGlowCorner.Parent = toggleGlow
        
        toggleButton.MouseButton1Click:Connect(function()
            ESP_SETTINGS[settingName] = not ESP_SETTINGS[settingName]
            toggleButton.Text = ESP_SETTINGS[settingName] and "ON" or "OFF"
            toggleButton.TextColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 120, 120)
            toggleButton.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
            toggleStroke.Color = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
            toggleGlow.BackgroundColor3 = ESP_SETTINGS[settingName] and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
            
            if settingName == "ENABLED" then
                espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
                espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(0, 255, 150)
                espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
                espMainButtonStroke.Color = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
                espButtonGlow.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(50, 255, 50)
                espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
                espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
                
                updateAllESPVisibility()
            end
            
            print("[Dusete Scripts] " .. text .. ": " .. (ESP_SETTINGS[settingName] and "ATIVADO" or "DESATIVADO"))
        end)
        
        -- Efeitos de hover
        toggleButton.MouseEnter:Connect(function()
            local tween = TweenService:Create(toggleGlow, TweenInfo.new(0.2), {
                BackgroundTransparency = 0.7
            })
            tween:Play()
        end)
        
        toggleButton.MouseLeave:Connect(function()
            local tween = TweenService:Create(toggleGlow, TweenInfo.new(0.2), {
                BackgroundTransparency = 0.9
            })
            tween:Play()
        end)
        
        return settingFrame
    end
    
    -- Criar opções de configuração
    createSettingOption("ESP Principal", "ENABLED", "👁️")
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
    footer.Parent = contentFrame
    
    local footerText = Instance.new("TextLabel")
    footerText.Name = "FooterText"
    footerText.Size = UDim2.new(1, 0, 1, 0)
    footerText.Text = "© 2024 Dusete Scripts • v2.0 • Insert: Toggle ESP"
    footerText.TextColor3 = Color3.fromRGB(150, 150, 200)
    footerText.TextSize = 12
    footerText.Font = Enum.Font.Gotham
    footerText.BackgroundTransparency = 1
    footerText.Parent = footer
    
    -- Conectar eventos
    
    -- Efeitos de hover no botão principal
    espMainButton.MouseEnter:Connect(function()
        local tween = TweenService:Create(espButtonGlow, TweenInfo.new(0.2), {
            BackgroundTransparency = 0.7
        })
        tween:Play()
    end)
    
    espMainButton.MouseLeave:Connect(function()
        local tween = TweenService:Create(espButtonGlow, TweenInfo.new(0.2), {
            BackgroundTransparency = 0.9
        })
        tween:Play()
    end)
    
    -- Botão principal do ESP
    espMainButton.MouseButton1Click:Connect(function()
        ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
        
        espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
        espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(0, 255, 150)
        espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
        espMainButtonStroke.Color = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
        espButtonGlow.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(50, 255, 50)
        espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
        espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
        
        -- Atualizar toggle nas configurações
        local toggleBtn = settingsContainer:FindFirstChild("ENABLED")
        if toggleBtn then
            toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
            toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 120, 120)
            toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
        end
        
        updateAllESPVisibility()
        print("[Dusete Scripts] ESP " .. (ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO"))
    end)
    
    -- Botão de fechar
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
        duseteWindow = nil
        isWindowVisible = false
        print("[Dusete Scripts] Janela fechada")
    end)
    
    -- Botão de minimizar
    minimizeButton.MouseButton1Click:Connect(function()
        if isWindowVisible then
            -- Minimizar
            local tween = TweenService:Create(mainWindow, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 320, 0, 55),
                Position = UDim2.new(0.5, -160, 1, -65)
            })
            tween:Play()
            
            contentFrame.Visible = false
            footer.Visible = false
            minimizeButton.Text = "+"
            isWindowVisible = false
        else
            -- Maximizar
            local tween = TweenService:Create(mainWindow, TweenInfo.new(0.3), {
                Size = UDim2.new(0, 320, 0, 420),
                Position = UDim2.new(0.5, -160, 0.5, -210)
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
    
    -- Efeito de entrada suave
    mainWindow.Position = UDim2.new(0.5, -160, 0.5, -250)
    mainWindow.BackgroundTransparency = 1
    
    local entranceTween = TweenService:Create(mainWindow, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, -160, 0.5, -210),
        BackgroundTransparency = 0.1
    })
    
    entranceTween:Play()
    
    -- Aguardar um pouco antes de mostrar conteúdo
    task.wait(0.2)
    contentFrame.Visible = true
    footer.Visible = true
    
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

-- Função para criar um objeto ESP (CORRIGIDA - não desaparece após morte)
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
    
    -- Criar caixa (Outline) - Agora com Parent correto
    if ESP_SETTINGS.SHOW_BOX then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = player.Name .. "_ESPBox"
        box.Adornee = nil
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Size = Vector3.new(4, 6, 2)
        box.Transparency = 0.3
        box.Visible = ESP_SETTINGS.ENABLED
        box.Color3 = Color3.new(1, 1, 1) -- Cor padrão
        box.Parent = Workspace -- IMPORTANTE: Parent no Workspace, não no playerGui
        
        espObject.Box = box
    end
    
    -- Criar labels para informações
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = player.Name .. "_ESPGui"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.Parent = playerGui -- Este permanece no PlayerGui
    
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
    
    -- Função para atualizar quando o personagem muda (CORRIGIDA)
    local function characterAdded(character)
        if not character then return end
        
        -- Esperar um pouco para garantir que as partes existam
        task.wait(0.1)
        
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart", 3)
        local humanoid = character:WaitForChild("Humanoid", 3)
        
        if espObject.Box then
            espObject.Box.Adornee = humanoidRootPart or character:FindFirstChild("Head") or character:FindFirstChild("Torso")
            espObject.Box.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_BOX and (humanoidRootPart ~= nil)
        end
        
        if humanoid and espObject.HealthLabel then
            local function updateHealth()
                if humanoid and humanoid.Parent then
                    local healthPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
                    espObject.HealthLabel.Text = healthPercent .. "%"
                    
                    if healthPercent > 70 then
                        espObject.HealthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                    elseif healthPercent > 30 then
                        espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
                    else
                        espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                    end
                else
                    espObject.HealthLabel.Text = "0%"
                    espObject.HealthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                end
            end
            
            -- Limpar conexões antigas
            for _, conn in ipairs(espObject.Connections) do
                if conn.Connected then
                    conn:Disconnect()
                end
            end
            espObject.Connections = {}
            
            -- Criar nova conexão
            local healthConn = humanoid.HealthChanged:Connect(updateHealth)
            table.insert(espObject.Connections, healthConn)
            
            updateHealth()
        end
    end
    
    -- Função para lidar com quando o personagem é removido
    local function characterRemoving()
        if espObject.Box then
            espObject.Box.Adornee = nil
        end
        
        -- Não destruir os objetos, apenas esconder
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
    end
    
    -- Conectar eventos
    local charAddedConn = player.CharacterAdded:Connect(characterAdded)
    local charRemovingConn = player.CharacterRemoving:Connect(characterRemoving)
    
    table.insert(espObject.Connections, charAddedConn)
    table.insert(espObject.Connections, charRemovingConn)
    
    -- Se já tem personagem, configurar agora
    if player.Character then
        characterAdded(player.Character)
    end
    
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

-- Função para atualizar o ESP de um jogador (CORRIGIDA)
local function updateESP(player, espObject)
    if not ESP_SETTINGS.ENABLED then 
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return 
    end
    
    if not player or not player.Character or not player.Character.Parent then
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return
    end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    local head = character:FindFirstChild("Head")
    local humanoid = character:FindFirstChild("Humanoid")
    
    if not humanoidRootPart or not head or not humanoid then 
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return 
    end
    
    local localCharacter = localPlayer.Character
    local localRootPart = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
    
    if not localRootPart then 
        if espObject.Box then espObject.Box.Visible = false end
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
        return 
    end
    
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
        
        local raycastResult = Workspace:Raycast(
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
        
        -- Garantir que o Adornee ainda está definido
        if not espObject.Box.Adornee then
            espObject.Box.Adornee = humanoidRootPart
        end
    end
    
    -- Atualizar labels na tela
    local screenPosition, onScreen = Workspace.CurrentCamera:WorldToViewportPoint(head.Position + Vector3.new(0, 2, 0))
    
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
            local rootScreenPosition = Workspace.CurrentCamera:WorldToViewportPoint(humanoidRootPart.Position)
            local bottomScreenPosition = Vector2.new(rootScreenPosition.X, rootScreenPosition.Y)
            
            espObject.Tracer.Visible = ESP_SETTINGS.ENABLED and ESP_SETTINGS.SHOW_TRACER and isVisible
            espObject.Tracer.BackgroundColor3 = color
            espObject.Tracer.Position = UDim2.new(0, bottomScreenPosition.X, 0, bottomScreenPosition.Y)
            espObject.Tracer.Size = UDim2.new(0, ESP_SETTINGS.TRACER_THICKNESS, 0, Workspace.CurrentCamera.ViewportSize.Y - bottomScreenPosition.Y)
        end
    else
        if espObject.NameLabel then espObject.NameLabel.Visible = false end
        if espObject.DistanceLabel then espObject.DistanceLabel.Visible = false end
        if espObject.HealthLabel then espObject.HealthLabel.Visible = false end
        if espObject.Tracer then espObject.Tracer.Visible = false end
    end
end

-- Função principal de atualização
local function updateAllESP()
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
        task.wait(1) -- Esperar um pouco para garantir que o jogador está carregado
        createESPObject(player)
    end
end))

-- Remover ESP quando jogador sair
table.insert(connections, Players.PlayerRemoving:Connect(function(player)
    local espObject = espCache[player]
    if espObject then
        for _, connection in ipairs(espObject.Connections) do
            if connection.Connected then
                connection:Disconnect()
            end
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

-- Criar a janela Dusete Scripts
task.wait(1) -- Esperar um pouco antes de criar a janela
duseteWindow = createDuseteWindow()

-- Tecla para ativar/desativar ESP (Insert)
table.insert(connections, UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Insert then
            ESP_SETTINGS.ENABLED = not ESP_SETTINGS.ENABLED
            
            -- Atualizar janela se existir
            if duseteWindow and duseteWindow.Parent then
                local espMainButton = duseteWindow:FindFirstChild("DuseteWindow"):FindFirstChild("Content"):FindFirstChild("ESPCard"):FindFirstChild("ESPMainButton")
                local espStatus = duseteWindow:FindFirstChild("DuseteWindow"):FindFirstChild("Content"):FindFirstChild("ESPCard"):FindFirstChild("ESPStatus")
                
                if espMainButton then
                    espMainButton.Text = ESP_SETTINGS.ENABLED and "DESATIVAR" or "ATIVAR"
                    espMainButton.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(0, 255, 150)
                    espMainButton.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(80, 0, 0) or Color3.fromRGB(0, 80, 0)
                end
                
                if espStatus then
                    espStatus.Text = ESP_SETTINGS.ENABLED and "Status: ATIVADO" or "Status: DESATIVADO"
                    espStatus.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 100, 100)
                end
                
                -- Atualizar toggle nas configurações
                local settingsContainer = duseteWindow:FindFirstChild("DuseteWindow"):FindFirstChild("Content"):FindFirstChild("SettingsContainer")
                if settingsContainer then
                    local toggleBtn = settingsContainer:FindFirstChild("ENABLED")
                    if toggleBtn then
                        toggleBtn.Text = ESP_SETTINGS.ENABLED and "ON" or "OFF"
                        toggleBtn.TextColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 120, 120)
                        toggleBtn.BackgroundColor3 = ESP_SETTINGS.ENABLED and Color3.fromRGB(0, 60, 0) or Color3.fromRGB(60, 0, 0)
                    end
                end
            end
            
            -- Atualizar visibilidade de todos os ESPs
            updateAllESPVisibility()
            
            print("[Dusete Scripts] ESP " .. (ESP_SETTINGS.ENABLED and "ATIVADO" or "DESATIVADO") .. " (Insert)")
        elseif input.KeyCode == Enum.KeyCode.F9 then
            -- Tecla para reabrir a janela se fechada
            if not duseteWindow or not duseteWindow.Parent then
                duseteWindow = createDuseteWindow()
                print("[Dusete Scripts] Janela reaberta (F9)")
            end
        end
    end
end))

-- Limpeza quando o script for destruído
local function cleanup()
    for _, connection in ipairs(connections) do
        if connection.Connected then
            connection:Disconnect()
        end
    end
    
    for _, espObject in pairs(espCache) do
        if espObject.Box then espObject.Box:Destroy() end
        if espObject.NameLabel then espObject.NameLabel.Parent:Destroy() end
    end
    
    if duseteWindow then duseteWindow:Destroy() end
    if espFolder then espFolder:Destroy() end
end

-- Conectar evento de saída
localPlayer.AncestryChanged:Connect(function()
    cleanup()
end)

-- Inicialização completa
print("==========================================")
print("DUSETE SCRIPTS - ESP Premium v2.0")
print("==========================================")
print("Janela aberta automaticamente!")
print("Controles:")
print("- Insert: Ativar/Desativar ESP")
print("- F9: Reabrir janela (se fechada)")
print("==========================================")
