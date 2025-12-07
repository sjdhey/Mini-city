-- Ice Menu Style - Mini City RP
-- Menu completo com todas as funcionalidades

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

-- Configurações
local Settings = {
    Fly = {
        Enabled = false,
        Speed = 50,
        BodyVelocity = nil
    },
    ESP = {
        Enabled = false,
        Boxes = {},
        Tracers = {},
        Info = {},
        ShowCash = true,
        ShowWeapons = true,
        ShowWhenDead = true
    },
    Combat = {
        AutoRob = false,
        AutoTp = false,
        Aimbot = false,
        Fov = 100,
        Smoothing = 0.2,
        TeamCheck = true,
        WallCheck = true,
        AimKey = Enum.UserInputType.MouseButton2,
        TargetBone = "Head"
    },
    Visual = {
        NoClip = false,
        Speed = false,
        SpeedValue = 16,
        JumpPower = false,
        JumpValue = 50
    },
    Misc = {
        AntiAfk = false,
        AutoCollect = false,
        InfiniteAmmo = false,
        NoRecoil = false,
        FullBright = false
    },
    AimbotTarget = nil,
    TeleportLocations = {
        ["Banco"] = Vector3.new(100, 5, 100),
        ["DP"] = Vector3.new(-200, 5, 50),
        ["Hospital"] = Vector3.new(150, 5, -150),
        ["Loja de Armas"] = Vector3.new(-100, 5, -100),
        ["Concessionária"] = Vector3.new(50, 5, 200)
    }
}

-- Função para obter inventário do player
local function GetPlayerInventory(player)
    local inventory = {}
    local weapons = {}
    local cash = 0
    local items = {}
    
    -- Verificar Backpack
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            if item:IsA("Tool") then
                table.insert(weapons, item.Name)
            elseif item.Name:find("Cash") or item.Name:find("Money") or item.Name:find("Dinheiro") then
                if item:FindFirstChild("Value") then
                    cash = cash + item.Value
                elseif item:IsA("NumberValue") or item:IsA("IntValue") then
                    cash = cash + item.Value
                end
            end
        end
    end
    
    -- Verificar Character
    local character = player.Character
    if character then
        for _, item in pairs(character:GetChildren()) do
            if item:IsA("Tool") then
                table.insert(weapons, item.Name)
            end
        end
        
        -- Procurar por pasta de itens no Character
        local itemsFolder = character:FindFirstChild("Items") or character:FindFirstChild("Inventory")
        if itemsFolder then
            for _, item in pairs(itemsFolder:GetChildren()) do
                if item:IsA("Tool") then
                    table.insert(weapons, item.Name)
                end
            end
        end
    end
    
    -- Procurar por leaderstats ou Data
    local leaderstats = player:FindFirstChild("leaderstats") or player:FindFirstChild("Leaderstats") or player:FindFirstChild("Data")
    if leaderstats then
        local moneyStat = leaderstats:FindFirstChild("Money") or 
                         leaderstats:FindFirstChild("Cash") or
                         leaderstats:FindFirstChild("$") or
                         leaderstats:FindFirstChild("Dinheiro")
        if moneyStat then
            cash = cash + moneyStat.Value
        end
    end
    
    return {
        Weapons = weapons,
        Cash = cash,
        WeaponCount = #weapons
    }
end

-- Instância da GUI
local IceHub = Instance.new("ScreenGui")
IceHub.Name = "IceHub_MiniCity"
IceHub.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
IceHub.ResetOnSpawn = false

if syn and syn.protect_gui then
    syn.protect_gui(IceHub)
elseif gethui then
    IceHub.Parent = gethui()
else
    IceHub.Parent = game.CoreGui
end

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 500, 0, 350)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -175)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = IceHub

-- Top Bar
local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 30)
TopBar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "❄️ ICE HUB | MINI CITY RP"
Title.TextColor3 = Color3.fromRGB(0, 150, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.BackgroundTransparency = 1
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 16
CloseButton.Parent = TopBar

CloseButton.MouseButton1Click:Connect(function()
    IceHub.Enabled = not IceHub.Enabled
end)

-- Minimizar/Maximizar
local MinMaxButton = Instance.new("TextButton")
MinMaxButton.Name = "MinMaxButton"
MinMaxButton.Size = UDim2.new(0, 30, 0, 30)
MinMaxButton.Position = UDim2.new(1, -60, 0, 0)
MinMaxButton.BackgroundTransparency = 1
MinMaxButton.Text = "_"
MinMaxButton.TextColor3 = Color3.fromRGB(200, 200, 200)
MinMaxButton.Font = Enum.Font.GothamBold
MinMaxButton.TextSize = 16
MinMaxButton.Parent = TopBar

MinMaxButton.MouseButton1Click:Connect(function()
    if MainFrame.Size == UDim2.new(0, 500, 0, 350) then
        MainFrame:TweenSize(UDim2.new(0, 500, 0, 30), "Out", "Quad", 0.3, true)
    else
        MainFrame:TweenSize(UDim2.new(0, 500, 0, 350), "Out", "Quad", 0.3, true)
    end
end)

-- Draggable
local dragging
local dragInput
local dragStart
local startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Sidebar
local SideBar = Instance.new("Frame")
SideBar.Name = "SideBar"
SideBar.Size = UDim2.new(0, 120, 1, -30)
SideBar.Position = UDim2.new(0, 0, 0, 30)
SideBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
SideBar.BorderSizePixel = 0
SideBar.Parent = MainFrame

-- Content Frame
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -120, 1, -30)
ContentFrame.Position = UDim2.new(0, 120, 0, 30)
ContentFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ContentFrame.BorderSizePixel = 0
ContentFrame.Parent = MainFrame

-- Botões da Sidebar
local Buttons = {
    {"Player", "🚶"},
    {"Visual", "👁️"},
    {"Combat", "⚔️"},
    {"Mundo", "🌍"},
    {"Extra", "⚙️"},
    {"Créditos", "❤️"}
}

local CurrentTab = "Player"
local TabFrames = {}

-- Criar abas
for i, tab in ipairs(Buttons) do
    local TabButton = Instance.new("TextButton")
    TabButton.Name = tab[1] .. "Button"
    TabButton.Size = UDim2.new(1, 0, 0, 40)
    TabButton.Position = UDim2.new(0, 0, 0, (i-1) * 40)
    TabButton.BackgroundColor3 = i == 1 and Color3.fromRGB(30, 30, 30) or Color3.fromRGB(20, 20, 20)
    TabButton.BorderSizePixel = 0
    TabButton.Text = "  " .. tab[2] .. "  " .. tab[1]
    TabButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    TabButton.Font = Enum.Font.Gotham
    TabButton.TextSize = 12
    TabButton.TextXAlignment = Enum.TextXAlignment.Left
    TabButton.Parent = SideBar
    
    local TabFrame = Instance.new("ScrollingFrame")
    TabFrame.Name = tab[1] .. "Frame"
    TabFrame.Size = UDim2.new(1, 0, 1, 0)
    TabFrame.Position = UDim2.new(0, 0, 0, 0)
    TabFrame.BackgroundTransparency = 1
    TabFrame.BorderSizePixel = 0
    TabFrame.ScrollBarThickness = 3
    TabFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
    TabFrame.Visible = i == 1
    TabFrame.Parent = ContentFrame
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Padding = UDim.new(0, 5)
    UIListLayout.Parent = TabFrame
    
    TabFrames[tab[1]] = TabFrame
    
    TabButton.MouseButton1Click:Connect(function()
        CurrentTab = tab[1]
        
        -- Resetar cores dos botões
        for _, btn in pairs(SideBar:GetChildren()) do
            if btn:IsA("TextButton") then
                btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
            end
        end
        
        -- Destacar botão atual
        TabButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        
        -- Mostrar/ocultar abas
        for tabName, frame in pairs(TabFrames) do
            frame.Visible = tabName == tab[1]
        end
    end)
end

-- Função para criar seções
local function CreateSection(parent, name)
    local SectionFrame = Instance.new("Frame")
    SectionFrame.Name = name .. "Section"
    SectionFrame.Size = UDim2.new(1, -20, 0, 40)
    SectionFrame.Position = UDim2.new(0, 10, 0, 0)
    SectionFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    SectionFrame.BorderSizePixel = 0
    SectionFrame.Parent = parent
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Name = "SectionTitle"
    SectionTitle.Size = UDim2.new(1, 0, 0, 40)
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Text = name
    SectionTitle.TextColor3 = Color3.fromRGB(0, 150, 255)
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.TextSize = 14
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    SectionTitle.Parent = SectionFrame
    
    local Content = Instance.new("Frame")
    Content.Name = "Content"
    Content.Size = UDim2.new(1, 0, 0, 0)
    Content.Position = UDim2.new(0, 0, 0, 40)
    Content.BackgroundTransparency = 1
    Content.Parent = SectionFrame
    
    local UIListLayout = Instance.new("UIListLayout")
    UIListLayout.Padding = UDim.new(0, 5)
    UIListLayout.Parent = Content
    
    return Content
end

-- Função para criar toggle
local function CreateToggle(parent, name, callback, default)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Name = name .. "Toggle"
    ToggleFrame.Size = UDim2.new(1, 0, 0, 30)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = parent
    
    local ToggleText = Instance.new("TextLabel")
    ToggleText.Name = "ToggleText"
    ToggleText.Size = UDim2.new(0.7, 0, 1, 0)
    ToggleText.BackgroundTransparency = 1
    ToggleText.Text = name
    ToggleText.TextColor3 = Color3.fromRGB(200, 200, 200)
    ToggleText.Font = Enum.Font.Gotham
    ToggleText.TextSize = 12
    ToggleText.TextXAlignment = Enum.TextXAlignment.Left
    ToggleText.Parent = ToggleFrame
    
    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Name = "ToggleButton"
    ToggleButton.Size = UDim2.new(0, 40, 0, 20)
    ToggleButton.Position = UDim2.new(1, -40, 0.5, -10)
    ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    ToggleButton.BorderSizePixel = 0
    ToggleButton.Text = ""
    ToggleButton.Parent = ToggleFrame
    
    local ToggleCircle = Instance.new("Frame")
    ToggleCircle.Name = "ToggleCircle"
    ToggleCircle.Size = UDim2.new(0, 16, 0, 16)
    ToggleCircle.Position = UDim2.new(0, 2, 0.5, -8)
    ToggleCircle.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    ToggleCircle.BorderSizePixel = 0
    ToggleCircle.Parent = ToggleButton
    
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(1, 0)
    Corner.Parent = ToggleCircle
    
    local Toggled = default or false
    
    local function UpdateToggle()
        if Toggled then
            TweenService:Create(ToggleCircle, TweenInfo.new(0.2), {
                Position = UDim2.new(1, -18, 0.5, -8),
                BackgroundColor3 = Color3.fromRGB(50, 255, 50)
            }):Play()
            ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
        else
            TweenService:Create(ToggleCircle, TweenInfo.new(0.2), {
                Position = UDim2.new(0, 2, 0.5, -8),
                BackgroundColor3 = Color3.fromRGB(255, 50, 50)
            }):Play()
            ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        end
    end
    
    ToggleButton.MouseButton1Click:Connect(function()
        Toggled = not Toggled
        UpdateToggle()
        if callback then
            callback(Toggled)
        end
    end)
    
    UpdateToggle()
    return ToggleFrame, function(newValue) 
        Toggled = newValue 
        UpdateToggle()
        if callback then
            callback(newValue)
        end
    end
end

-- Função para criar slider
local function CreateSlider(parent, name, min, max, default, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Name = name .. "Slider"
    SliderFrame.Size = UDim2.new(1, 0, 0, 50)
    SliderFrame.BackgroundTransparency = 1
    SliderFrame.Parent = parent
    
    local SliderText = Instance.new("TextLabel")
    SliderText.Name = "SliderText"
    SliderText.Size = UDim2.new(1, 0, 0, 20)
    SliderText.BackgroundTransparency = 1
    SliderText.Text = name .. ": " .. default
    SliderText.TextColor3 = Color3.fromRGB(200, 200, 200)
    SliderText.Font = Enum.Font.Gotham
    SliderText.TextSize = 12
    SliderText.TextXAlignment = Enum.TextXAlignment.Left
    SliderText.Parent = SliderFrame
    
    local SliderBar = Instance.new("Frame")
    SliderBar.Name = "SliderBar"
    SliderBar.Size = UDim2.new(1, 0, 0, 5)
    SliderBar.Position = UDim2.new(0, 0, 1, -15)
    SliderBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    SliderBar.BorderSizePixel = 0
    SliderBar.Parent = SliderFrame
    
    local SliderFill = Instance.new("Frame")
    SliderFill.Name = "SliderFill"
    SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    SliderFill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderBar
    
    local SliderButton = Instance.new("TextButton")
    SliderButton.Name = "SliderButton"
    SliderButton.Size = UDim2.new(0, 15, 0, 15)
    SliderButton.Position = UDim2.new((default - min) / (max - min), -7.5, 0.5, -7.5)
    SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderButton.Text = ""
    SliderButton.Parent = SliderBar
    
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(1, 0)
    Corner.Parent = SliderButton
    
    local dragging = false
    
    local function updateSlider(input)
        local pos = UDim2.new(
            math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1),
            0,
            0.5,
            -7.5
        )
        SliderButton.Position = pos
        SliderFill.Size = UDim2.new(pos.X.Scale, 0, 1, 0)
        
        local value = math.floor(min + (pos.X.Scale * (max - min)))
        SliderText.Text = name .. ": " .. value
        
        if callback then
            callback(value)
        end
    end
    
    SliderButton.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    
    SliderButton.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            updateSlider(input)
        end
    end)
    
    return SliderFrame
end

-- Função para criar dropdown
local function CreateDropdown(parent, name, options, default, callback)
    local DropdownFrame = Instance.new("Frame")
    DropdownFrame.Name = name .. "Dropdown"
    DropdownFrame.Size = UDim2.new(1, 0,
