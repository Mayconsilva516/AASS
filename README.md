local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- === CONFIGURAÇÃO DA UI ===
local UIConfig = {
    Title = "MKZZ HUB",
    Modes = {
        Light = {
            Background = Color3.fromRGB(240, 240, 240),
            Panel = Color3.fromRGB(255, 255, 255),
            Text = Color3.fromRGB(80, 80, 80),
            TextLight = Color3.fromRGB(150, 150, 150),
            Selected = Color3.fromRGB(220, 220, 220)
        },
        Dark = {
            Background = Color3.fromRGB(30, 30, 30),
            Panel = Color3.fromRGB(45, 45, 45),
            Text = Color3.fromRGB(220, 220, 220),
            TextLight = Color3.fromRGB(160, 160, 160),
            Selected = Color3.fromRGB(70, 70, 70)
        }
    },
    CurrentTheme = "Light",
    Keybind = Enum.KeyCode.RightControl
}

-- Atalho para o tema atual
local Theme = UIConfig.Modes.Light

-- === CONFIGURAÇÕES (LÓGICA) ===
local Settings = {
    Player = {
        SpeedEnabled = false,
        Speed = 16,
        JumpEnabled = false,
        Jump = 50,
        InfiniteStamina = false,
        FlyEnabled = false,
        FlySpeed = 50
    },
    Aimbot = {
        Enabled = false,
        WallCheck = true,
        TargetPart = "Head", -- Cabeça
        Smoothness = 0.5,
        FOV = 100,
        ShowFOV = false
    },
    ESP = {
        Enabled = false,
        Name = false,
        Distance = false,
        Weapon = false
    }
}

-- === SISTEMA DE PROTEÇÃO DA GUI ===
local function getSafeParent()
    if gethui then return gethui() end
    if syn and syn.protect_gui then 
        local gui = Instance.new("ScreenGui")
        syn.protect_gui(gui)
        gui.Parent = CoreGui
        return gui.Parent
    elseif CoreGui:FindFirstChild("RobloxGui") then
        return CoreGui
    end
    local player = Players.LocalPlayer
    if player then return player:WaitForChild("PlayerGui") end
    return CoreGui
end

-- === CRIAÇÃO DA INTERFACE ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MKZZ_Interface"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = getSafeParent()

-- Frame Principal (Container)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 600, 0, 350)
MainFrame.Position = UDim2.new(0, 20, 0, 20)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- === SIDEBAR (Esquerda) ===
local Sidebar = Instance.new("Frame")
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 70, 1, -20)
Sidebar.Position = UDim2.new(0, 10, 0, 10)
Sidebar.BackgroundColor3 = Theme.Panel
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0, 10)
SidebarCorner.Parent = Sidebar

local SidebarLayout = Instance.new("UIListLayout")
SidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
SidebarLayout.Padding = UDim.new(0, 10)
SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
SidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Top
SidebarLayout.Parent = Sidebar

local SidebarPadding = Instance.new("UIPadding")
SidebarPadding.PaddingTop = UDim.new(0, 15)
SidebarPadding.Parent = Sidebar

-- === HEADER (Topo Direita) ===
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, -100, 0, 50)
Header.Position = UDim2.new(0, 90, 0, 10)
Header.BackgroundTransparency = 1
Header.Parent = MainFrame

-- Info Box 2 (Usuário)
local UserBox = Instance.new("Frame")
UserBox.Name = "UserBox"
UserBox.Size = UDim2.new(0.75, 0, 1, 0) -- Reduzido para caber botões
UserBox.Position = UDim2.new(0, 0, 0, 0)
UserBox.BackgroundColor3 = Theme.Panel
UserBox.Parent = Header

local UserCorner = Instance.new("UICorner")
UserCorner.CornerRadius = UDim.new(0, 8)
UserCorner.Parent = UserBox

local UserLabel = Instance.new("TextLabel")
UserLabel.Name = "UserLabel"
UserLabel.Size = UDim2.new(1, 0, 1, 0)
UserLabel.BackgroundTransparency = 1
UserLabel.Text = "Usuário: " .. Players.LocalPlayer.Name
UserLabel.TextColor3 = Theme.TextLight
UserLabel.Font = Enum.Font.GothamBold
UserLabel.TextSize = 12
UserLabel.Parent = UserBox

-- Botão Minimizar (-)
local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Name = "Minimize"
MinimizeButton.Size = UDim2.new(0, 50, 1, 0)
MinimizeButton.Position = UDim2.new(0.78, 0, 0, 0)
MinimizeButton.BackgroundColor3 = Theme.Panel
MinimizeButton.Text = "-"
MinimizeButton.TextColor3 = Theme.Text
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 24
MinimizeButton.Parent = Header

local MinimizeCorner = Instance.new("UICorner")
MinimizeCorner.CornerRadius = UDim.new(0, 8)
MinimizeCorner.Parent = MinimizeButton

local isMinimized = false
MinimizeButton.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        MainFrame:TweenSize(UDim2.new(0, 600, 0, 70), "Out", "Quad", 0.3, true)
        Sidebar.Visible = false
        PagesContainer.Visible = false
        MinimizeButton.Text = "+"
    else
        MainFrame:TweenSize(UDim2.new(0, 600, 0, 350), "Out", "Quad", 0.3, true)
        Sidebar.Visible = true
        PagesContainer.Visible = true
        MinimizeButton.Text = "-"
    end
end)

-- Botão Fechar (X)
local CloseButton = Instance.new("TextButton")
CloseButton.Name = "Close"
CloseButton.Size = UDim2.new(0, 50, 1, 0)
CloseButton.Position = UDim2.new(0.89, 0, 0, 0)
CloseButton.BackgroundColor3 = Theme.Panel
CloseButton.Text = "X"
CloseButton.TextColor3 = Theme.Text
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.Parent = Header

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseButton

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

-- === CONTENT AREA (Conteúdo) ===
local PagesContainer = Instance.new("Frame")
PagesContainer.Name = "PagesContainer"
PagesContainer.Size = UDim2.new(1, -100, 1, -80)
PagesContainer.Position = UDim2.new(0, 90, 0, 70)
PagesContainer.BackgroundTransparency = 1
PagesContainer.Parent = MainFrame

-- === FUNÇÕES DE ABAS ===
local Tabs = {}
local CurrentTab = nil

local function createTab(id, iconText)
    -- Botão da Sidebar
    local TabButton = Instance.new("TextButton")
    TabButton.Name = "Tab_" .. id
    TabButton.Size = UDim2.new(0, 50, 0, 50)
    TabButton.BackgroundColor3 = Theme.Background -- Cinza claro padrão
    TabButton.Text = iconText
    TabButton.TextColor3 = Theme.Text
    TabButton.Font = Enum.Font.GothamBold
    TabButton.TextSize = 24
    TabButton.Parent = Sidebar
    
    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 10)
    TabCorner.Parent = TabButton
    
    -- Página
    local Page = Instance.new("ScrollingFrame")
    Page.Name = "Page_" .. id
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.BorderSizePixel = 0
    Page.ScrollBarThickness = 4
    Page.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Page.Visible = false
    Page.Parent = PagesContainer
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 10)
    UIListLayout.Parent = Page
    
    -- Título da Página (Opções)
    local PageTitle = Instance.new("TextLabel")
    PageTitle.Name = "PageTitle"
    PageTitle.Size = UDim2.new(1, 0, 0, 30)
    PageTitle.BackgroundTransparency = 1
    PageTitle.Text = "Opções"
    PageTitle.TextColor3 = Theme.Text
    PageTitle.Font = Enum.Font.GothamBold
    PageTitle.TextSize = 16
    PageTitle.Parent = Page
    
    -- Lógica de Seleção
    TabButton.MouseButton1Click:Connect(function()
        if CurrentTab then
            CurrentTab.Button.BackgroundColor3 = Theme.Background
            CurrentTab.Page.Visible = false
        end
        
        CurrentTab = {Button = TabButton, Page = Page}
        TabButton.BackgroundColor3 = Theme.Selected
        Page.Visible = true
    end)
    
    return Page, TabButton
end

-- === COMPONENTES DE UI ===
local function createSlider(text, parent, min, max, default, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Name = "Slider_" .. text
    SliderFrame.Size = UDim2.new(1, 0, 0, 50)
    SliderFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderFrame.BorderSizePixel = 0
    SliderFrame.Parent = parent

    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0, 8)
    SliderCorner.Parent = SliderFrame

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.5, -10, 1, 0)
    Label.Position = UDim2.new(0, 15, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Theme.Text
    Label.Font = Enum.Font.GothamBold
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = SliderFrame

    local ValueLabel = Instance.new("TextLabel")
    ValueLabel.Size = UDim2.new(0, 30, 0, 20)
    ValueLabel.Position = UDim2.new(1, -45, 0.5, 5)
    ValueLabel.BackgroundTransparency = 1
    ValueLabel.Text = tostring(default)
    ValueLabel.TextColor3 = Theme.TextLight
    ValueLabel.Font = Enum.Font.Gotham
    ValueLabel.TextSize = 12
    ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
    ValueLabel.Parent = SliderFrame

    local SliderBg = Instance.new("Frame")
    SliderBg.Size = UDim2.new(0.4, 0, 0, 6)
    SliderBg.Position = UDim2.new(0.5, 0, 0.5, 0)
    SliderBg.AnchorPoint = Vector2.new(0, 0.5)
    SliderBg.BackgroundColor3 = Color3.fromRGB(220, 220, 220)
    SliderBg.BorderSizePixel = 0
    SliderBg.Parent = SliderFrame

    local SliderBgCorner = Instance.new("UICorner")
    SliderBgCorner.CornerRadius = UDim.new(1, 0)
    SliderBgCorner.Parent = SliderBg

    local SliderFill = Instance.new("Frame")
    SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    SliderFill.BackgroundColor3 = Theme.Text
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderBg

    local SliderFillCorner = Instance.new("UICorner")
    SliderFillCorner.CornerRadius = UDim.new(1, 0)
    SliderFillCorner.Parent = SliderFill

    local Trigger = Instance.new("TextButton")
    Trigger.Size = UDim2.new(1, 0, 1, 0)
    Trigger.BackgroundTransparency = 1
    Trigger.Text = ""
    Trigger.Parent = SliderBg

    local dragging = false

    local function updateValue(input)
        local pos = UDim2.new(math.clamp((input.Position.X - SliderBg.AbsolutePosition.X) / SliderBg.AbsoluteSize.X, 0, 1), 0, 1, 0)
        SliderFill.Size = pos
        
        local value = math.floor(min + ((max - min) * pos.X.Scale))
        ValueLabel.Text = tostring(value)
        callback(value)
    end

    Trigger.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            updateValue(input)
        end
    end)

    Trigger.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            updateValue(input)
        end
    end)
end

local function createToggle(text, parent, default, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Name = "Toggle_" .. text
    ToggleFrame.Size = UDim2.new(1, 0, 0, 50)
    ToggleFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleFrame.BorderSizePixel = 0
    ToggleFrame.Parent = parent

    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0, 8)
    ToggleCorner.Parent = ToggleFrame

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.7, 0, 1, 0)
    Label.Position = UDim2.new(0, 15, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Theme.Text
    Label.Font = Enum.Font.GothamBold
    Label.TextSize = 14
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame

    local ToggleBg = Instance.new("Frame")
    ToggleBg.Size = UDim2.new(0, 40, 0, 20)
    ToggleBg.Position = UDim2.new(1, -55, 0.5, 0)
    ToggleBg.AnchorPoint = Vector2.new(0, 0.5)
    ToggleBg.BackgroundColor3 = default and Theme.Text or Color3.fromRGB(220, 220, 220)
    ToggleBg.BorderSizePixel = 0
    ToggleBg.Parent = ToggleFrame

    local ToggleBgCorner = Instance.new("UICorner")
    ToggleBgCorner.CornerRadius = UDim.new(1, 0)
    ToggleBgCorner.Parent = ToggleBg

    local ToggleCircle = Instance.new("Frame")
    ToggleCircle.Size = UDim2.new(0, 16, 0, 16)
    ToggleCircle.Position = default and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
    ToggleCircle.AnchorPoint = Vector2.new(0, 0.5)
    ToggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleCircle.BorderSizePixel = 0
    ToggleCircle.Parent = ToggleBg

    local ToggleCircleCorner = Instance.new("UICorner")
    ToggleCircleCorner.CornerRadius = UDim.new(1, 0)
    ToggleCircleCorner.Parent = ToggleCircle

    local Trigger = Instance.new("TextButton")
    Trigger.Size = UDim2.new(1, 0, 1, 0)
    Trigger.BackgroundTransparency = 1
    Trigger.Text = ""
    Trigger.Parent = ToggleFrame

    local enabled = default

    Trigger.MouseButton1Click:Connect(function()
        enabled = not enabled
        
        local targetColor = enabled and Theme.Text or Color3.fromRGB(220, 220, 220)
        local targetPos = enabled and UDim2.new(1, -18, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
        
        TweenService:Create(ToggleBg, TweenInfo.new(0.2), {BackgroundColor3 = targetColor}):Play()
        TweenService:Create(ToggleCircle, TweenInfo.new(0.2), {Position = targetPos}):Play()
        
        callback(enabled)
    end)
end

local function createButton(text, parent, callback)
    local ButtonFrame = Instance.new("Frame")
    ButtonFrame.Name = "Button_" .. text
    ButtonFrame.Size = UDim2.new(1, 0, 0, 40)
    ButtonFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ButtonFrame.BorderSizePixel = 0
    ButtonFrame.Parent = parent

    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 8)
    ButtonCorner.Parent = ButtonFrame

    local ButtonBtn = Instance.new("TextButton")
    ButtonBtn.Size = UDim2.new(1, 0, 1, 0)
    ButtonBtn.BackgroundTransparency = 1
    ButtonBtn.Text = text
    ButtonBtn.TextColor3 = Theme.Text
    ButtonBtn.Font = Enum.Font.GothamBold
    ButtonBtn.TextSize = 14
    ButtonBtn.Parent = ButtonFrame

    ButtonBtn.MouseButton1Click:Connect(function()
        -- Efeito de clique simples
        TweenService:Create(ButtonFrame, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(200, 200, 200)}):Play()
        task.wait(0.1)
        TweenService:Create(ButtonFrame, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(255, 255, 255)}):Play()
        callback()
    end)
end

-- === CRIANDO ABAS (Ícones) ===
local PlayerPage, PlayerTab = createTab("Player", "👤") -- Icone de Usuário
local WeaponPage, WeaponTab = createTab("Weapon", "🔫") -- Icone de Arma
local VisualPage, VisualTab = createTab("Visual", "👁️") -- Icone de Olho
local ConfigPage, ConfigTab = createTab("Config", "⚙️")

-- === OPÇÕES DA ABA VISUAL ===
createToggle("Ativar ESP (Box)", VisualPage, false, function(value)
    Settings.ESP.Enabled = value
end)

createToggle("Mostrar Nomes", VisualPage, false, function(value)
    Settings.ESP.Name = value
end)

createToggle("Mostrar Distância", VisualPage, false, function(value)
    Settings.ESP.Distance = value
end)

createToggle("Mostrar Arma", VisualPage, false, function(value)
    Settings.ESP.Weapon = value
end)

-- === OPÇÕES DA ABA CONFIG ===
local HttpService = game:GetService("HttpService")
local ConfigName = "MKZZ_Config.json"

local function SaveConfig()
    if writefile then
        local json = HttpService:JSONEncode(Settings)
        writefile(ConfigName, json)
    end
end

local function ResetConfig()
    if delfile and isfile and isfile(ConfigName) then
        delfile(ConfigName)
    end
    -- Reseta valores visuais (não recarrega o script inteiro para não crashar)
    -- Idealmente, apenas notificaria, mas vamos tentar resetar alguns toggles básicos se possível
    -- Por enquanto, apenas deleta o arquivo.
end

local function ApplyTheme(mode)
    local colors = UIConfig.Modes[mode]
    Theme = colors -- Atualiza a referência global
    
    -- Main Layout
    MainFrame.BackgroundColor3 = colors.Background
    Sidebar.BackgroundColor3 = colors.Panel
    UserBox.BackgroundColor3 = colors.Panel
    MinimizeButton.BackgroundColor3 = colors.Panel
    MinimizeButton.TextColor3 = colors.Text
    CloseButton.BackgroundColor3 = colors.Panel
    CloseButton.TextColor3 = colors.Text
    
    -- Iterate all descendants for generic updates
    for _, obj in pairs(MainFrame:GetDescendants()) do
        -- Tabs
        if obj.Name:find("Tab_") and obj:IsA("TextButton") then
            -- Check if selected
            if CurrentTab and CurrentTab.Button == obj then
                obj.BackgroundColor3 = colors.Selected
            else
                obj.BackgroundColor3 = colors.Background
            end
            obj.TextColor3 = colors.Text
        end
        
        -- Sliders/Toggles/Buttons Containers
        if (obj.Name:find("Slider_") or obj.Name:find("Toggle_") or obj.Name:find("Button_")) and obj:IsA("Frame") then
            obj.BackgroundColor3 = colors.Panel
        end
        
        -- Labels
        if obj:IsA("TextLabel") then
            if obj.Name == "UserLabel" then
                 obj.TextColor3 = colors.TextLight
            elseif obj.Name == "PageTitle" or obj.Name == "Label" then
                 obj.TextColor3 = colors.Text
            elseif obj.Name == "ValueLabel" then
                 obj.TextColor3 = colors.TextLight
            end
        end
        
        -- Toggle Switch
        if obj.Name == "ToggleBg" then
             -- Precisamos saber se está ativo ou não para definir a cor
             -- Mas o loop de render/evento cuidará da cor do toggle ativo (Theme.Text)
             -- Aqui definimos apenas o fundo inativo se necessário, mas o Tween cuida disso.
             -- Vamos deixar o ToggleBg ser atualizado apenas pelo clique por enquanto, 
             -- ou forçar update visual se necessário.
             -- Simplificação: Atualiza apenas cores estáticas.
        end
        
        -- TextButton dentro de Botões
        if obj.Parent.Name:find("Button_") and obj:IsA("TextButton") then
            obj.TextColor3 = colors.Text
        end
    end
end

createToggle("Tema Escuro", ConfigPage, false, function(value)
    if value then
        ApplyTheme("Dark")
    else
        ApplyTheme("Light")
    end
end)

createButton("Salvar Configuração", ConfigPage, function()
    SaveConfig()
end)

createButton("Resetar Configuração", ConfigPage, function()
    ResetConfig()
end)

createToggle("Stream Proof (Hide UI)", ConfigPage, true, function(value)
    if value then
        ScreenGui.Parent = getSafeParent()
    else
        ScreenGui.Parent = Players.LocalPlayer:WaitForChild("PlayerGui")
    end
end)

-- === OPÇÕES DA ABA WEAPON ===
createToggle("Aimbot (Cabeça)", WeaponPage, false, function(value)
    Settings.Aimbot.Enabled = value
end)

createToggle("Wall Check", WeaponPage, true, function(value)
    Settings.Aimbot.WallCheck = value
end)

createSlider("Smoothness", WeaponPage, 1, 10, 5, function(value)
    Settings.Aimbot.Smoothness = value / 10
end)

createToggle("Show FOV", WeaponPage, false, function(value)
    Settings.Aimbot.ShowFOV = value
end)

createSlider("FOV Radius", WeaponPage, 50, 500, 100, function(value)
    Settings.Aimbot.FOV = value
end)

-- === LÓGICA DO AIMBOT ===
local Camera = workspace.CurrentCamera
local Mouse = Players.LocalPlayer:GetMouse()

-- Círculo FOV (Fallback para GUI se Drawing não existir)
local FOVCircle
local FOVGui
local FOVFrame

if Drawing then
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Color = Color3.fromRGB(255, 255, 255)
    FOVCircle.Thickness = 1
    FOVCircle.NumSides = 60
    FOVCircle.Radius = Settings.Aimbot.FOV
    FOVCircle.Visible = false
    FOVCircle.Filled = false
    FOVCircle.Transparency = 1
else
    -- Fallback: Cria um GUI Circle se o executor não suportar Drawing API
    FOVGui = Instance.new("ScreenGui")
    FOVGui.Name = "FOVGui"
    FOVGui.Parent = getSafeParent()
    FOVGui.ResetOnSpawn = false
    
    FOVFrame = Instance.new("ImageLabel")
    FOVFrame.Name = "FOVCircle"
    FOVFrame.Image = "rbxassetid://3570695787" -- Textura de círculo vazado
    FOVFrame.BackgroundTransparency = 1
    FOVFrame.ImageColor3 = Color3.fromRGB(255, 255, 255)
    FOVFrame.ImageTransparency = 0.5
    FOVFrame.Parent = FOVGui
    FOVFrame.Visible = false
end

RunService.RenderStepped:Connect(function()
    if Drawing and FOVCircle then
        FOVCircle.Visible = Settings.Aimbot.ShowFOV
        FOVCircle.Radius = Settings.Aimbot.FOV
        FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
    elseif FOVFrame then
        FOVFrame.Visible = Settings.Aimbot.ShowFOV
        local diameter = Settings.Aimbot.FOV * 2
        FOVFrame.Size = UDim2.new(0, diameter, 0, diameter)
        FOVFrame.Position = UDim2.new(0, Mouse.X - Settings.Aimbot.FOV, 0, Mouse.Y - Settings.Aimbot.FOV + 36)
    end
end)

local function isVisible(targetPart)
    if not Settings.Aimbot.WallCheck then return true end
    
    local origin = Camera.CFrame.Position
    local direction = targetPart.Position - origin
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {Players.LocalPlayer.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.IgnoreWater = true
    
    local result = workspace:Raycast(origin, direction, raycastParams)
    
    -- Se bater em algo e esse algo for descendente do personagem do alvo, está visível
    if result and result.Instance:IsDescendantOf(targetPart.Parent) then
        return true
    end
    -- Se não bater em nada (alcance infinito) ou bater em algo que não é o alvo mas está muito perto (bug fix), retorna false
    -- Mas Raycast retorna nil se não bater, então precisamos verificar se a distância é livre
    -- Correção: Se bater em algo que NÃO é o personagem inimigo, então tem parede.
    if result then
        return false
    end
    
    return true -- Caminho livre (ou alvo muito longe para o raycast padrão, mas assumimos visível se não bateu em parede próxima)
end

local function getClosestPlayer()
    local closest = nil
    local shortestDistance = Settings.Aimbot.FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local part = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("HumanoidRootPart")
            
            -- Verifica WallCheck
            if isVisible(part) then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                
                if onScreen then
                    local magnitude = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                    
                    if magnitude < shortestDistance then
                        closest = player
                        shortestDistance = magnitude
                    end
                end
            end
        end
    end
    
    return closest
end

-- Loop principal do Aimbot
RunService.RenderStepped:Connect(function()
    if Settings.Aimbot.Enabled then
        local target = getClosestPlayer()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("HumanoidRootPart")
            
            if targetPart then
                 -- Aimbot (Camera Lock)
                local currentCFrame = Camera.CFrame
                local targetCFrame = CFrame.new(currentCFrame.Position, targetPart.Position)
                
                -- Interpolação suave (Smoothness corrigido: 1 = lento, 0.1 = muito lento. Vamos inverter lógica)
                -- 1 no slider = 0.1 alpha (lento)
                -- 10 no slider = 1 alpha (instantâneo)
                Camera.CFrame = currentCFrame:Lerp(targetCFrame, Settings.Aimbot.Smoothness)
            end
        end
    end
end)

-- === LÓGICA DO ESP ===
local ESP_Cache = {}

local function UpdateESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer then
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                
                if not ESP_Cache[player] then
                    ESP_Cache[player] = {}
                end

                -- 1. HIGHLIGHT (Box/Chams)
                if Settings.ESP.Enabled then
                    if not ESP_Cache[player].Highlight then
                        local hl = Instance.new("Highlight")
                        hl.Name = "ESPHighlight"
                        hl.FillColor = Color3.fromRGB(255, 0, 0)
                        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                        hl.FillTransparency = 0.5
                        hl.OutlineTransparency = 0
                        hl.Adornee = player.Character
                        hl.Parent = getSafeParent()
                        ESP_Cache[player].Highlight = hl
                    else
                        ESP_Cache[player].Highlight.Adornee = player.Character
                        ESP_Cache[player].Highlight.Enabled = true
                    end
                else
                    if ESP_Cache[player].Highlight then
                        ESP_Cache[player].Highlight.Enabled = false
                    end
                end

                -- 2. TEXT INFO (Name, Dist, Weapon)
                if Settings.ESP.Name or Settings.ESP.Distance or Settings.ESP.Weapon then
                    if not ESP_Cache[player].Billboard then
                        local bb = Instance.new("BillboardGui")
                        bb.Name = "ESPInfo"
                        bb.Size = UDim2.new(0, 200, 0, 50)
                        bb.StudsOffset = Vector3.new(0, 4, 0) -- Acima da cabeça
                        bb.AlwaysOnTop = true
                        bb.Adornee = player.Character:FindFirstChild("Head") or player.Character.HumanoidRootPart
                        bb.Parent = getSafeParent()
                        
                        local txt = Instance.new("TextLabel")
                        txt.Size = UDim2.new(1, 0, 1, 0)
                        txt.BackgroundTransparency = 1
                        txt.TextColor3 = Color3.fromRGB(255, 255, 255)
                        txt.TextStrokeTransparency = 0
                        txt.TextSize = 13
                        txt.Font = Enum.Font.GothamBold
                        txt.Parent = bb
                        
                        ESP_Cache[player].Billboard = bb
                        ESP_Cache[player].TextLabel = txt
                    else
                        ESP_Cache[player].Billboard.Adornee = player.Character:FindFirstChild("Head") or player.Character.HumanoidRootPart
                        ESP_Cache[player].Billboard.Enabled = true
                    end

                    -- Construct Text
                    local content = ""
                    if Settings.ESP.Name then
                        content = content .. player.Name .. "\n"
                    end
                    if Settings.ESP.Distance and Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                        local dist = (Players.LocalPlayer.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                        content = content .. string.format("[%d m]\n", math.floor(dist))
                    end
                    if Settings.ESP.Weapon then
                        local weapon = player.Character:FindFirstChildOfClass("Tool")
                        if weapon then
                            content = content .. "Arma: " .. weapon.Name
                        else
                            content = content .. "Arma: Nenhuma"
                        end
                    end
                    
                    ESP_Cache[player].TextLabel.Text = content
                else
                     if ESP_Cache[player].Billboard then
                        ESP_Cache[player].Billboard.Enabled = false
                    end
                end

            else
                -- Player dead or invalid, hide ESP
                if ESP_Cache[player] then
                    if ESP_Cache[player].Highlight then ESP_Cache[player].Highlight.Enabled = false end
                    if ESP_Cache[player].Billboard then ESP_Cache[player].Billboard.Enabled = false end
                end
            end
        end
    end
end

RunService.RenderStepped:Connect(UpdateESP)

-- Limpeza de Cache ao sair
Players.PlayerRemoving:Connect(function(player)
    if ESP_Cache[player] then
        if ESP_Cache[player].Highlight then ESP_Cache[player].Highlight:Destroy() end
        if ESP_Cache[player].Billboard then ESP_Cache[player].Billboard:Destroy() end
        ESP_Cache[player] = nil
    end
end)

-- === OPÇÕES DA ABA PLAYER ===
createToggle("Jump Power", PlayerPage, false, function(value)
    Settings.Player.JumpEnabled = value
    if not value and Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
        Players.LocalPlayer.Character.Humanoid.JumpPower = 50
    end
end)

createSlider("Força do Pulo", PlayerPage, 50, 300, 50, function(value)
    Settings.Player.Jump = value
end)

createToggle("Fly (Voar)", PlayerPage, false, function(value)
    Settings.Player.FlyEnabled = value
end)

createSlider("Velocidade do Fly", PlayerPage, 10, 200, 50, function(value)
    Settings.Player.FlySpeed = value
end)

-- === LÓGICA DO FLY ===
local FlyKeys = {W = false, A = false, S = false, D = false, Space = false, Control = false}

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.W then FlyKeys.W = true
    elseif input.KeyCode == Enum.KeyCode.A then FlyKeys.A = true
    elseif input.KeyCode == Enum.KeyCode.S then FlyKeys.S = true
    elseif input.KeyCode == Enum.KeyCode.D then FlyKeys.D = true
    elseif input.KeyCode == Enum.KeyCode.Space then FlyKeys.Space = true
    elseif input.KeyCode == Enum.KeyCode.LeftControl then FlyKeys.Control = true end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then FlyKeys.W = false
    elseif input.KeyCode == Enum.KeyCode.A then FlyKeys.A = false
    elseif input.KeyCode == Enum.KeyCode.S then FlyKeys.S = false
    elseif input.KeyCode == Enum.KeyCode.D then FlyKeys.D = false
    elseif input.KeyCode == Enum.KeyCode.Space then FlyKeys.Space = false
    elseif input.KeyCode == Enum.KeyCode.LeftControl then FlyKeys.Control = false end
end)

RunService.RenderStepped:Connect(function()
    local player = Players.LocalPlayer
    if not player or not player.Character then return end
    
    local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
    local humanoid = player.Character:FindFirstChild("Humanoid")
    
    if rootPart and humanoid and humanoid.Health > 0 then
        if Settings.Player.FlyEnabled then
            local camera = workspace.CurrentCamera
            local speed = Settings.Player.FlySpeed
            
            -- BodyVelocity
            local bv = rootPart:FindFirstChild("FlyVelocity") or Instance.new("BodyVelocity")
            bv.Name = "FlyVelocity"
            bv.Parent = rootPart
            bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            
            -- BodyGyro
            local bg = rootPart:FindFirstChild("FlyGyro") or Instance.new("BodyGyro")
            bg.Name = "FlyGyro"
            bg.Parent = rootPart
            bg.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
            bg.P = 10000
            bg.D = 100
            
            -- Movimento
            local newVelocity = Vector3.zero
            if FlyKeys.W then newVelocity = newVelocity + camera.CFrame.LookVector end
            if FlyKeys.S then newVelocity = newVelocity - camera.CFrame.LookVector end
            if FlyKeys.A then newVelocity = newVelocity - camera.CFrame.RightVector end
            if FlyKeys.D then newVelocity = newVelocity + camera.CFrame.RightVector end
            if FlyKeys.Space then newVelocity = newVelocity + Vector3.new(0, 1, 0) end
            if FlyKeys.Control then newVelocity = newVelocity - Vector3.new(0, 1, 0) end
            
            bv.Velocity = newVelocity * speed
            bg.CFrame = camera.CFrame
            
            -- Desativa estado de queda/animação de andar para parecer voo
            humanoid.PlatformStand = true
        else
            -- Remove física do Fly
            if rootPart:FindFirstChild("FlyVelocity") then rootPart.FlyVelocity:Destroy() end
            if rootPart:FindFirstChild("FlyGyro") then rootPart.FlyGyro:Destroy() end
            humanoid.PlatformStand = false
        end
    end
end)

-- Loop para forçar status
RunService.RenderStepped:Connect(function()
    pcall(function()
        local player = Players.LocalPlayer
        if not player then return end
        
        local character = player.Character
        if not character then return end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            if Settings.Player.JumpEnabled then
                humanoid.UseJumpPower = true
                humanoid.JumpPower = Settings.Player.Jump
            end
        end
    end)
end)


-- Seleciona primeira aba
PlayerTab.BackgroundColor3 = UIConfig.Theme.Selected
PlayerPage.Visible = true
CurrentTab = {Button = PlayerTab, Page = PlayerPage}

-- === FUNCIONALIDADE DE ARRASTAR ===
local DraggingState = {
    IsDragging = false,
    DragStart = nil,
    StartPos = nil,
    InputObj = nil
}

local function RegisterDraggable(frame)
    frame.Active = true
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            DraggingState.IsDragging = true
            DraggingState.DragStart = input.Position
            DraggingState.StartPos = MainFrame.Position
            DraggingState.InputObj = input
        end
    end)
end

-- Conecta o movimento global
UserInputService.InputChanged:Connect(function(input)
    if DraggingState.IsDragging then
        if input == DraggingState.InputObj or input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - DraggingState.DragStart
            MainFrame.Position = UDim2.new(
                DraggingState.StartPos.X.Scale, 
                DraggingState.StartPos.X.Offset + delta.X, 
                DraggingState.StartPos.Y.Scale, 
                DraggingState.StartPos.Y.Offset + delta.Y
            )
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == DraggingState.InputObj or input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        DraggingState.IsDragging = false
        DraggingState.InputObj = nil
    end
end)

-- Aplica a todos os frames que podem ser usados para arrastar
RegisterDraggable(MainFrame)
RegisterDraggable(Sidebar)
RegisterDraggable(Header)
RegisterDraggable(UserBox)
RegisterDraggable(PagesContainer)

-- Toggle Keybind
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == UIConfig.Keybind then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- Notificação
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "MKZZ HUB",
    Text = "Interface carregada! Use Right Control para abrir/fechar.",
    Duration = 5
})
