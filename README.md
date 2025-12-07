local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "❄️ Ice Hub Mobile",
    LoadingTitle = "Carregando Ice Hub...",
    LoadingSubtitle = "Mini City RP - Versão Mobile",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "IceHubMobile_MiniCity",
        FileName = "Config"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    },
    KeySystem = false,
})

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

-- Configurações Mobile
local Settings = {
    Fly = {
        Enabled = false,
        Speed = 50,
        BodyVelocity = nil,
        TouchControls = false,
        ControlButtons = {}
    },
    ESP = {
        Enabled = false,
        Boxes = {},
        Info = {},
        ShowCash = true,
        ShowWeapons = true,
        SimpleMode = true
    },
    Combat = {
        AutoRob = false,
        AutoTp = false,
        AutoFarm = false
    },
    Visual = {
        Speed = false,
        SpeedValue = 16,
        JumpPower = false,
        NoClip = false
    },
    Misc = {
        AntiAfk = false,
        AutoCollect = false,
        TouchUI = false
    }
}

-- Função para obter inventário (mobile otimizada)
local function GetPlayerInventory(player)
    local weapons = {}
    local cash = 0
    
    -- Versão simplificada para mobile
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            if item:IsA("Tool") then
                table.insert(weapons, item.Name)
            end
        end
    end
    
    -- Verificar leaderstats rápido
    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local money = leaderstats:FindFirstChild("Money") or leaderstats:FindFirstChild("Cash")
        if money then
            cash = money.Value
        end
    end
    
    return {
        Weapons = weapons,
        Cash = cash,
        WeaponCount = #weapons
    }
end

-- Criar Tabs otimizadas para mobile
local MainTab = Window:CreateTab("Principal", "rbxassetid://4483345998")
local VisualTab = Window:CreateTab("Visual", "rbxassetid://4483345998")
local CombatTab = Window:CreateTab("Combate", "rbxassetid://4483345998")
local MiscTab = Window:CreateTab("Extra", "rbxassetid://4483345998")

-- Seção FLY MOBILE
local FlySection = MainTab:CreateSection("Fly Mobile")

local FlyToggle = MainTab:CreateToggle({
    Name = "Ativar Fly Mobile",
    CurrentValue = false,
    Flag = "FlyToggle",
    Callback = function(value)
        Settings.Fly.Enabled = value
        if value then
            SetupMobileFly()
        else
            DisableMobileFly()
        end
    end,
})

local FlySpeedSlider = MainTab:CreateSlider({
    Name = "Velocidade do Fly",
    Range = {20, 150},
    Increment = 5,
    Suffix = "speed",
    CurrentValue = 50,
    Flag = "FlySpeed",
    Callback = function(value)
        Settings.Fly.Speed = value
    end,
})

local TouchControlsToggle = MainTab:CreateToggle({
    Name = "Controles Touch",
    CurrentValue = false,
    Flag = "TouchControls",
    Callback = function(value)
        Settings.Fly.TouchControls = value
        if value then
            CreateTouchControls()
        else
            RemoveTouchControls()
        end
    end,
})

-- Função Fly Mobile otimizada
function SetupMobileFly()
    if not LocalPlayer.Character then return end
    
    local humanoidRootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = humanoidRootPart
    
    Settings.Fly.BodyVelocity = bodyVelocity
    
    -- Controles adaptados para mobile
    local flyConnection
    flyConnection = RunService.Heartbeat:Connect(function()
        if not Settings.Fly.Enabled or not Settings.Fly.BodyVelocity or not humanoidRootPart then
            if flyConnection then
                flyConnection:Disconnect()
            end
            return
        end
        
        local direction = Vector3.new(0, 0, 0)
        local speed = Settings.Fly.Speed
        
        -- Usar teclas virtuais ou touch
        if Settings.Fly.TouchControls then
            -- Controles touch serão adicionados separadamente
        else
            -- Controles por teclado (para emuladores)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                direction = direction + (humanoidRootPart.CFrame.LookVector * speed)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                direction = direction - (humanoidRootPart.CFrame.LookVector * speed)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                direction = direction - (humanoidRootPart.CFrame.RightVector * speed)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                direction = direction + (humanoidRootPart.CFrame.RightVector * speed)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                direction = direction + Vector3.new(0, speed, 0)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                direction = direction - Vector3.new(0, speed, 0)
            end
        end
        
        Settings.Fly.BodyVelocity.Velocity = direction
    end)
end

function DisableMobileFly()
    if Settings.Fly.BodyVelocity then
        Settings.Fly.BodyVelocity:Destroy()
        Settings.Fly.BodyVelocity = nil
    end
end

-- Controles Touch para Fly
function CreateTouchControls()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FlyTouchControls"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    if syn and syn.protect_gui then
        syn.protect_gui(screenGui)
    end
    screenGui.Parent = game.CoreGui
    
    -- Joystick para movimento
    local joystickFrame = Instance.new("Frame")
    joystickFrame.Name = "Joystick"
    joystickFrame.Size = UDim2.new(0, 150, 0, 150)
    joystickFrame.Position = UDim2.new(0, 50, 1, -200)
    joystickFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    joystickFrame.BackgroundTransparency = 0.5
    joystickFrame.BorderSizePixel = 0
    joystickFrame.Parent = screenGui
    
    local joystickInner = Instance.new("Frame")
    joystickInner.Name = "JoystickInner"
    joystickInner.Size = UDim2.new(0, 60, 0, 60)
    joystickInner.Position = UDim2.new(0.5, -30, 0.5, -30)
    joystickInner.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    joystickInner.BackgroundTransparency = 0.3
    joystickInner.BorderSizePixel = 0
    joystickInner.Parent = joystickFrame
    
    local UICorner1 = Instance.new("UICorner")
    UICorner1.CornerRadius = UDim.new(1, 0)
    UICorner1.Parent = joystickFrame
    
    local UICorner2 = Instance.new("UICorner")
    UICorner2.CornerRadius = UDim.new(1, 0)
    UICorner2.Parent = joystickInner
    
    -- Botões de subir/descer
    local upButton = Instance.new("TextButton")
    upButton.Name = "UpButton"
    upButton.Size = UDim2.new(0, 80, 0, 80)
    upButton.Position = UDim2.new(1, -300, 1, -200)
    upButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    upButton.BackgroundTransparency = 0.3
    upButton.Text = "↑"
    upButton.TextColor3 = Color3.white
    upButton.Font = Enum.Font.GothamBold
    upButton.TextSize = 24
    upButton.Parent = screenGui
    
    local downButton = Instance.new("TextButton")
    downButton.Name = "DownButton"
    downButton.Size = UDim2.new(0, 80, 0, 80)
    downButton.Position = UDim2.new(1, -300, 1, -100)
    downButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    downButton.BackgroundTransparency = 0.3
    downButton.Text = "↓"
    downButton.TextColor3 = Color3.white
    downButton.Font = Enum.Font.GothamBold
    downButton.TextSize = 24
    downButton.Parent = screenGui
    
    local UICorner3 = Instance.new("UICorner")
    UICorner3.CornerRadius = UDim.new(1, 0)
    UICorner3.Parent = upButton
    
    local UICorner4 = Instance.new("UICorner")
    UICorner4.CornerRadius = UDim.new(1, 0)
    UICorner4.Parent = downButton
    
    Settings.Fly.ControlButtons = {
        Joystick = joystickFrame,
        UpButton = upButton,
        DownButton = downButton
    }
    
    -- Lógica do joystick
    local dragging = false
    local joyStartPos
    
    joystickFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            joyStartPos = input.Position
        end
    end)
    
    joystickFrame.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.Touch then
            local currentPos = input.Position
            local delta = currentPos - joyStartPos
            local maxDistance = 50
            
            -- Calcular posição relativa
            local direction = Vector2.new(
                math.clamp(delta.X / maxDistance, -1, 1),
                math.clamp(delta.Y / maxDistance, -1, 1)
            )
            
            -- Mover joystick visual
            joystickInner.Position = UDim2.new(
                0.5, (direction.X * 30),
                0.5, (direction.Y * 30)
            )
            
            -- Atualizar direção do fly
            if Settings.Fly.BodyVelocity and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local root = LocalPlayer.Character.HumanoidRootPart
                local moveDirection = (root.CFrame.LookVector * direction.Y) + (root.CFrame.RightVector * direction.X)
                Settings.Fly.BodyVelocity.Velocity = moveDirection * Settings.Fly.Speed
            end
        end
    end)
    
    joystickFrame.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            -- Resetar joystick
            TweenService:Create(joystickInner, TweenInfo.new(0.2), {
                Position = UDim2.new(0.5, -30, 0.5, -30)
            }):Play()
            
            -- Parar movimento
            if Settings.Fly.BodyVelocity then
                Settings.Fly.BodyVelocity.Velocity = Vector3.new(0, 0, 0)
            end
        end
    end)
    
    -- Botões de subir/descer
    local function flyUp()
        if Settings.Fly.BodyVelocity then
            Settings.Fly.BodyVelocity.Velocity = Vector3.new(0, Settings.Fly.Speed, 0)
        end
    end
    
    local function flyDown()
        if Settings.Fly.BodyVelocity then
            Settings.Fly.BodyVelocity.Velocity = Vector3.new(0, -Settings.Fly.Speed, 0)
        end
    end
    
    local function stopVertical()
        if Settings.Fly.BodyVelocity then
            local currentVel = Settings.Fly.BodyVelocity.Velocity
            Settings.Fly.BodyVelocity.Velocity = Vector3.new(currentVel.X, 0, currentVel.Z)
        end
    end
    
    upButton.MouseButton1Down:Connect(flyUp)
    upButton.MouseButton1Up:Connect(stopVertical)
    
    downButton.MouseButton1Down:Connect(flyDown)
    downButton.MouseButton1Up:Connect(stopVertical)
end

function RemoveTouchControls()
    for _, button in pairs(Settings.Fly.ControlButtons) do
        if button then
            button:Destroy()
        end
    end
    Settings.Fly.ControlButtons = {}
end

-- Seção MOVIMENTO
local MoveSection = MainTab:CreateSection("Movimento")

local SpeedToggle = MainTab:CreateToggle({
    Name = "Speed Hack",
    CurrentValue = false,
    Flag = "SpeedToggle",
    Callback = function(value)
        Settings.Visual.Speed = value
        if value then
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = Settings.Visual.SpeedValue
            end
        else
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = 16
            end
        end
    end,
})

local SpeedSlider = MainTab:CreateSlider({
    Name = "Velocidade",
    Range = {16, 100},
    Increment = 2,
    Suffix = "speed",
    CurrentValue = 16,
    Flag = "SpeedValue",
    Callback = function(value)
        Settings.Visual.SpeedValue = value
        if Settings.Visual.Speed then
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = value
            end
        end
    end,
})

local JumpToggle = MainTab:CreateToggle({
    Name = "Pulo Alto",
    CurrentValue = false,
    Flag = "JumpToggle",
    Callback = function(value)
        Settings.Visual.JumpPower = value
        if value then
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.JumpPower = 100
            end
        else
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid.JumpPower = 50
            end
        end
    end,
})

-- Seção ESP MOBILE
local ESPSection = VisualTab:CreateSection("ESP Mobile")

local EspToggle = VisualTab:CreateToggle({
    Name = "ESP (Inventário)",
    CurrentValue = false,
    Flag = "EspToggle",
    Callback = function(value)
        Settings.ESP.Enabled = value
        if value then
            SetupMobileESP()
        else
            ClearMobileESP()
        end
    end,
})

local SimpleEspToggle = VisualTab:CreateToggle({
    Name = "ESP Simples (otimizado)",
    CurrentValue = true,
    Flag = "SimpleEsp",
    Callback = function(value)
        Settings.ESP.SimpleMode = value
    end,
})

local ShowCashToggle = VisualTab:CreateToggle({
    Name = "Mostrar Dinheiro",
    CurrentValue = true,
    Flag = "ShowCash",
    Callback = function(value)
        Settings.ESP.ShowCash = value
    end,
})

local ShowWeaponsToggle = VisualTab:CreateToggle({
    Name = "Mostrar Armas",
    CurrentValue = true,
    Flag = "ShowWeapons",
    Callback = function(value)
        Settings.ESP.ShowWeapons = value
    end,
})

-- Função ESP otimizada para mobile
function SetupMobileESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            CreateMobileESP(player)
        end
    end
    
    Players.PlayerAdded:Connect(function(player)
        if Settings.ESP.Enabled then
            CreateMobileESP(player)
        end
    end)
    
    Players.PlayerRemoving:Connect(function(player)
        if Settings.ESP.Boxes[player] then
            if Settings.ESP.Boxes[player] then
                Settings.ESP.Boxes[player]:Destroy()
            end
            Settings.ESP.Boxes[player] = nil
        end
        if Settings.ESP.Info[player] then
            if Settings.ESP.Info[player] then
                Settings.ESP.Info[player]:Destroy()
            end
            Settings.ESP.Info[player] = nil
        end
    end)
end

function CreateMobileESP(player)
    if Settings.ESP.Boxes[player] then
        if Settings.ESP.Boxes[player] then
            Settings.ESP.Boxes[player]:Destroy()
        end
    end
    
    if Settings.ESP.Info[player] then
        if Settings.ESP.Info[player] then
            Settings.ESP.Info[player]:Destroy()
        end
    end
    
    if not player.Character then return end
    
    local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- ESP Box simplificado para mobile
    local espBox = Instance.new("BoxHandleAdornment")
    espBox.Name = player.Name .. "_ESPMobile"
    espBox.Adornee = rootPart
    espBox.Size = Vector3.new(2, 4, 1)
    espBox.Color3 = player.Team and player.TeamColor.Color or Color3.fromRGB(0, 255, 0)
    espBox.Transparency = 0.4
    espBox.ZIndex = 1
    espBox.AlwaysOnTop = true
    espBox.Parent = game.CoreGui
    
    Settings.ESP.Boxes[player] = espBox
    
    -- Informações simplificadas
    local billboard = Instance.new("BillboardGui")
    billboard.Name = player.Name .. "_ESPMobileInfo"
    billboard.Adornee = rootPart
    billboard.Size = UDim2.new(0, 150, 0, 80) -- Menor para mobile
    billboard.StudsOffset = Vector3.new(0, 3.5, 0)
    billboard.AlwaysOnTop = true
    billboard.MaxDistance = 300 -- Menor distância para otimizar
    billboard.Parent = game.CoreGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BackgroundTransparency = 0.7
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Parent = billboard
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0, 20)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 12
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.Parent = frame
    
    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(1, 0, 0, 40)
    infoLabel.Position = UDim2.new(0, 0, 0, 20)
    infoLabel.BackgroundTransparency = 1
    infoLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.TextSize = 10
    infoLabel.TextWrapped = true
    infoLabel.TextXAlignment = Enum.TextXAlignment.Left
    infoLabel.Parent = frame
    
    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Size = UDim2.new(1, 0, 0, 20)
    distanceLabel.Position = UDim2.new(0, 0, 0, 60)
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    distanceLabel.Font = Enum.Font.Gotham
    distanceLabel.TextSize = 10
    distanceLabel.Parent = frame
    
    Settings.ESP.Info[player] = billboard
    
    -- Atualizar em tempo real (otimizado)
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if not Settings.ESP.Enabled or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
            if connection then
                connection:Disconnect()
            end
            return
        end
        
        local localRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local targetRoot = player.Character.HumanoidRootPart
        
        if localRoot and targetRoot then
            -- Distância
            local distance = (localRoot.Position - targetRoot.Position).Magnitude
            distanceLabel.Text = "Dist: " .. math.floor(distance) .. "m"
            
            -- Inventário (atualizado menos frequentemente)
            if tick() % 1 < 0.1 then -- Atualizar a cada ~1 segundo
                local inventory = GetPlayerInventory(player)
                local infoText = ""
                
                if Settings.ESP.ShowCash and inventory.Cash > 0 then
                    infoText = infoText .. "💰$" .. inventory.Cash .. "\n"
                end
                
                if Settings.ESP.ShowWeapons and inventory.WeaponCount > 0 then
                    infoText = infoText .. "🔫" .. inventory.WeaponCount .. " armas"
                end
                
                infoLabel.Text = in
