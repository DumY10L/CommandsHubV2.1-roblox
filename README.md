-- COMMANDS HUB V2.1

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

for _, gui in ipairs({"CommandsHubGUI", "CommandsSystem", "ErrorNotification"}) do
    if playerGui:FindFirstChild(gui) then
        playerGui:FindFirstChild(gui):Destroy()
    end
end

-- Variables  
local espEnabled = false
local speedEnabled = false
local antiLagEnabled = false
local hitboxEnabled = false
local currentSpeed = 40
local normalSpeed = 16
local hitboxSize = 5
local espConnections = {}
local highlights = {}
local hitboxParts = {}
local isMinimized = false
local espUpdateConnection = nil
local originalSettings = {}
local disabledObjects = {}
local originalRenderSettings = {}
local originalTextures = {}
local originalMaterials = {}

-- Função para mostrar notificação de erro
local function showErrorNotification(title, message)
    local errorGui = Instance.new("ScreenGui")
    errorGui.Name = "ErrorNotification"
    errorGui.Parent = playerGui
    errorGui.ResetOnSpawn = false
    local errorFrame = Instance.new("Frame")
    errorFrame.Name = "ErrorFrame"
    errorFrame.Size = UDim2.new(0, 250, 0, 60)
    errorFrame.Position = UDim2.new(1, -270, 1, -150)
    errorFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    errorFrame.BorderSizePixel = 0
    errorFrame.Parent = errorGui
    local errorCorner = Instance.new("UICorner")
    errorCorner.CornerRadius = UDim.new(0, 10)
    errorCorner.Parent = errorFrame 
    local errorStroke = Instance.new("UIStroke")
    errorStroke.Color = Color3.fromRGB(255, 255, 255)
    errorStroke.Thickness = 1
    errorStroke.Parent = errorFrame 
    local errorIcon = Instance.new("TextLabel")
    errorIcon.Name = "ErrorIcon"
    errorIcon.Size = UDim2.new(0, 30, 1, 0)
    errorIcon.Position = UDim2.new(0, 10, 0, 0)
    errorIcon.BackgroundTransparency = 1
    errorIcon.Text = "⚠️"
    errorIcon.TextColor3 = Color3.fromRGB(255, 255, 255)
    errorIcon.TextSize = 20
    errorIcon.Font = Enum.Font.GothamBold
    errorIcon.TextXAlignment = Enum.TextXAlignment.Center
    errorIcon.Parent = errorFrame
    local errorTitle = Instance.new("TextLabel")
    errorTitle.Name = "ErrorTitle"
    errorTitle.Size = UDim2.new(1, -45, 0, 25)
    errorTitle.Position = UDim2.new(0, 40, 0, 5)
    errorTitle.BackgroundTransparency = 1
    errorTitle.Text = title
    errorTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    errorTitle.TextSize = 15
    errorTitle.Font = Enum.Font.GothamBold
    errorTitle.TextXAlignment = Enum.TextXAlignment.Left
    errorTitle.Parent = errorFrame  
    local errorMessage = Instance.new("TextLabel")
    errorMessage.Name = "ErrorMessage"
    errorMessage.Size = UDim2.new(1, -45, 0, 20)
    errorMessage.Position = UDim2.new(0, 40, 0, 25)
    errorMessage.BackgroundTransparency = 1
    errorMessage.Text = message
    errorMessage.TextColor3 = Color3.fromRGB(255, 255, 255)
    errorMessage.TextSize = 15
    errorMessage.Font = Enum.Font.Gotham
    errorMessage.TextXAlignment = Enum.TextXAlignment.Left
    errorMessage.Parent = errorFrame 
    errorFrame.Position = UDim2.new(1, 0, 1, -150)
    local enterTween = TweenService:Create(errorFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(1, -270, 1, -150)
    })
    enterTween:Play() 
    spawn(function()
        wait(3)
        local exitTween = TweenService:Create(errorFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
            Position = UDim2.new(1, 0, 1, -150)
        })
        exitTween:Play()
        exitTween.Completed:Connect(function()
            errorGui:Destroy()
        end)
    end)
end

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CommandsHubGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 350, 0, 280)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -140)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(30, 30, 30)
stroke.Thickness = 2
stroke.Parent = mainFrame

-- Header
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 40)
header.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
header.BorderSizePixel = 0
header.Parent = mainFrame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 12)
headerCorner.Parent = header

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -100, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "⚙ Commands Hub v2.1"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

-- Minimize button 
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Name = "MinimizeButton"
minimizeBtn.Size = UDim2.new(0, 25, 0, 25)
minimizeBtn.Position = UDim2.new(1, -58, 0, 7.5)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
minimizeBtn.Text = "_"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.TextSize = 14
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Parent = header

local minimizeBtnCorner = Instance.new("UICorner")
minimizeBtnCorner.CornerRadius = UDim.new(0, 5)
minimizeBtnCorner.Parent = minimizeBtn

-- Close or end button 
local closeBtn = Instance.new("TextButton")
closeBtn.Name = "CloseButton"
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -28, 0, 7.5)
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 12
closeBtn.Font = Enum.Font.GothamBold
closeBtn.BorderSizePixel = 0
closeBtn.Parent = header

local closeBtnCorner = Instance.new("UICorner")
closeBtnCorner.CornerRadius = UDim.new(0, 5)
closeBtnCorner.Parent = closeBtn

-- Scroll Frame
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, -20, 1, -50)
scrollFrame.Position = UDim2.new(0, 10, 0, 45)
scrollFrame.BackgroundTransparency = 1
scrollFrame.BorderSizePixel = 0
scrollFrame.ScrollBarThickness = 12
scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 500) -- Aumentado para acomodar todas as ferramentas
scrollFrame.ScrollingDirection = Enum.ScrollingDirection.Y
scrollFrame.VerticalScrollBarInset = Enum.ScrollBarInset.None
scrollFrame.VerticalScrollBarPosition = Enum.VerticalScrollBarPosition.Right
scrollFrame.Parent = mainFrame

-- ESP
local espFrame = Instance.new("Frame")
espFrame.Size = UDim2.new(1, -10, 0, 80)
espFrame.Position = UDim2.new(0, 5, 0, 10)
espFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
espFrame.BorderSizePixel = 0
espFrame.Parent = scrollFrame

local espCorner = Instance.new("UICorner")
espCorner.CornerRadius = UDim.new(0, 8)
espCorner.Parent = espFrame

local espTitle = Instance.new("TextLabel")
espTitle.Size = UDim2.new(1, 0, 0, 25)
espTitle.Position = UDim2.new(0, 10, 0, 5)
espTitle.BackgroundTransparency = 1
espTitle.Text = "👁 Highlight-Player"
espTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
espTitle.TextSize = 14
espTitle.Font = Enum.Font.GothamBold
espTitle.TextXAlignment = Enum.TextXAlignment.Left
espTitle.Parent = espFrame

local espStatus = Instance.new("TextLabel")
espStatus.Size = UDim2.new(0.6, 0, 0, 20)
espStatus.Position = UDim2.new(0, 10, 0, 30)
espStatus.BackgroundTransparency = 1
espStatus.Text = "Status: Inactive"
espStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
espStatus.TextSize = 11
espStatus.Font = Enum.Font.Gotham
espStatus.TextXAlignment = Enum.TextXAlignment.Left
espStatus.Parent = espFrame

local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0, 120, 0, 35)
espBtn.Position = UDim2.new(1, -130, 0, 25)
espBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
espBtn.Text = "ENABLE"
espBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
espBtn.TextSize = 12
espBtn.Font = Enum.Font.GothamBold
espBtn.BorderSizePixel = 0
espBtn.Parent = espFrame

local espBtnCorner = Instance.new("UICorner")
espBtnCorner.CornerRadius = UDim.new(0, 8)
espBtnCorner.Parent = espBtn

local speedFrame = Instance.new("Frame")
speedFrame.Size = UDim2.new(1, -10, 0, 120) 
speedFrame.Position = UDim2.new(0, 5, 0, 100)
speedFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
speedFrame.BorderSizePixel = 0
speedFrame.Parent = scrollFrame

local speedCorner = Instance.new("UICorner")
speedCorner.CornerRadius = UDim.new(0, 8)
speedCorner.Parent = speedFrame

local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(1, 0, 0, 25)
speedTitle.Position = UDim2.new(0, 10, 0, 5)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "🏃 Walk-Speed"
speedTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
speedTitle.TextSize = 14
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextXAlignment = Enum.TextXAlignment.Left
speedTitle.Parent = speedFrame

local speedStatus = Instance.new("TextLabel")
speedStatus.Size = UDim2.new(1, 0, 0, 20)
speedStatus.Position = UDim2.new(0, 10, 0, 30)
speedStatus.BackgroundTransparency = 1
speedStatus.Text = "Status: Inactive - Speed: " .. currentSpeed
speedStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
speedStatus.TextSize = 11
speedStatus.Font = Enum.Font.Gotham
speedStatus.TextXAlignment = Enum.TextXAlignment.Left
speedStatus.Parent = speedFrame

local speedInputFrame = Instance.new("Frame")
speedInputFrame.Size = UDim2.new(1, -20, 0, 30)
speedInputFrame.Position = UDim2.new(0, 10, 0, 55)
speedInputFrame.BackgroundTransparency = 1
speedInputFrame.Parent = speedFrame

local speedInputLayout = Instance.new("UIListLayout")
speedInputLayout.FillDirection = Enum.FillDirection.Horizontal
speedInputLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
speedInputLayout.VerticalAlignment = Enum.VerticalAlignment.Center
speedInputLayout.Padding = UDim.new(0, 10)
speedInputLayout.Parent = speedInputFrame

-- TextBox value Speed
local speedInput = Instance.new("TextBox")
speedInput.Size = UDim2.new(0, 80, 1, 0)
speedInput.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
speedInput.Text = tostring(currentSpeed)
speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
speedInput.TextSize = 14
speedInput.Font = Enum.Font.Gotham
speedInput.BorderSizePixel = 0
speedInput.PlaceholderText = "Velocidade"
speedInput.TextXAlignment = Enum.TextXAlignment.Center
speedInput.Parent = speedInputFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 6)
inputCorner.Parent = speedInput

local applyBtn = Instance.new("TextButton")
applyBtn.Size = UDim2.new(0, 90, 1, 0)
applyBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
applyBtn.Text = "Apply"
applyBtn.TextSize = 12
applyBtn.Font = Enum.Font.GothamBold
applyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
applyBtn.BorderSizePixel = 0
applyBtn.Parent = speedInputFrame

local applyCorner = Instance.new("UICorner")
applyCorner.CornerRadius = UDim.new(0, 6)
applyCorner.Parent = applyBtn

local speedBtnFrame = Instance.new("Frame")
speedBtnFrame.Size = UDim2.new(1, -20, 0, 30)
speedBtnFrame.Position = UDim2.new(0, 10, 0, 90)
speedBtnFrame.BackgroundTransparency = 1
speedBtnFrame.Parent = speedFrame

local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(0, 180, 1, 0)
speedBtn.Position = UDim2.new(0.5, -90, 0, 0)
speedBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
speedBtn.Text = "ENABLE"
speedBtn.TextSize = 13
speedBtn.Font = Enum.Font.GothamBold
speedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedBtn.BorderSizePixel = 0
speedBtn.Parent = speedBtnFrame

local speedBtnCorner = Instance.new("UICorner")
speedBtnCorner.CornerRadius = UDim.new(0, 8)
speedBtnCorner.Parent = speedBtn

local antiLagFrame = Instance.new("Frame")
antiLagFrame.Size = UDim2.new(1, -10, 0, 80)
antiLagFrame.Position = UDim2.new(0, 5, 0, 230) 
antiLagFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
antiLagFrame.BorderSizePixel = 0
antiLagFrame.Parent = scrollFrame

local antiLagCorner = Instance.new("UICorner")
antiLagCorner.CornerRadius = UDim.new(0, 8)
antiLagCorner.Parent = antiLagFrame

local antiLagTitle = Instance.new("TextLabel")
antiLagTitle.Size = UDim2.new(1, 0, 0, 25)
antiLagTitle.Position = UDim2.new(0, 10, 0, 5)
antiLagTitle.BackgroundTransparency = 1
antiLagTitle.Text = "🚀 Anti-Lag"
antiLagTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
antiLagTitle.TextSize = 14
antiLagTitle.Font = Enum.Font.GothamBold
antiLagTitle.TextXAlignment = Enum.TextXAlignment.Left
antiLagTitle.Parent = antiLagFrame

local antiLagStatus = Instance.new("TextLabel")
antiLagStatus.Size = UDim2.new(0.6, 0, 0, 20)
antiLagStatus.Position = UDim2.new(0, 10, 0, 30)
antiLagStatus.BackgroundTransparency = 1
antiLagStatus.Text = "Status: DESATIVADO"
antiLagStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
antiLagStatus.TextSize = 11
antiLagStatus.Font = Enum.Font.Gotham
antiLagStatus.TextXAlignment = Enum.TextXAlignment.Left
antiLagStatus.Parent = antiLagFrame

local antiLagBtn = Instance.new("TextButton")
antiLagBtn.Size = UDim2.new(0, 120, 0, 35)
antiLagBtn.Position = UDim2.new(1, -130, 0, 25)
antiLagBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
antiLagBtn.Text = "ENABLE"
antiLagBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
antiLagBtn.TextSize = 12
antiLagBtn.Font = Enum.Font.GothamBold
antiLagBtn.BorderSizePixel = 0
antiLagBtn.Parent = antiLagFrame

local antiLagBtnCorner = Instance.new("UICorner")
antiLagBtnCorner.CornerRadius = UDim.new(0, 8)
antiLagBtnCorner.Parent = antiLagBtn

local hitboxFrame = Instance.new("Frame")
hitboxFrame.Size = UDim2.new(1, -10, 0, 120) 
hitboxFrame.Position = UDim2.new(0, 5, 0, 320) 
hitboxFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
hitboxFrame.BorderSizePixel = 0
hitboxFrame.Parent = scrollFrame

local hitboxCorner = Instance.new("UICorner")
hitboxCorner.CornerRadius = UDim.new(0, 8)
hitboxCorner.Parent = hitboxFrame

local hitboxTitle = Instance.new("TextLabel")
hitboxTitle.Size = UDim2.new(1, 0, 0, 25)
hitboxTitle.Position = UDim2.new(0, 10, 0, 5)
hitboxTitle.BackgroundTransparency = 1
hitboxTitle.Text = "🎯 Hitbox-Expander"
hitboxTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxTitle.TextSize = 14
hitboxTitle.Font = Enum.Font.GothamBold
hitboxTitle.TextXAlignment = Enum.TextXAlignment.Left
hitboxTitle.Parent = hitboxFrame

local hitboxStatus = Instance.new("TextLabel")
hitboxStatus.Size = UDim2.new(1, 0, 0, 20)
hitboxStatus.Position = UDim2.new(0, 10, 0, 30)
hitboxStatus.BackgroundTransparency = 1
hitboxStatus.Text = "Status: OFF - Tamanho: " .. hitboxSize
hitboxStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
hitboxStatus.TextSize = 11
hitboxStatus.Font = Enum.Font.Gotham
hitboxStatus.TextXAlignment = Enum.TextXAlignment.Left
hitboxStatus.Parent = hitboxFrame

local hitboxInputFrame = Instance.new("Frame")
hitboxInputFrame.Size = UDim2.new(1, -20, 0, 30)
hitboxInputFrame.Position = UDim2.new(0, 10, 0, 55)
hitboxInputFrame.BackgroundTransparency = 1
hitboxInputFrame.Parent = hitboxFrame

local hitboxInputLayout = Instance.new("UIListLayout")
hitboxInputLayout.FillDirection = Enum.FillDirection.Horizontal
hitboxInputLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
hitboxInputLayout.VerticalAlignment = Enum.VerticalAlignment.Center
hitboxInputLayout.Padding = UDim.new(0, 10)
hitboxInputLayout.Parent = hitboxInputFrame

-- TextBox value hitbox
local hitboxInput = Instance.new("TextBox")
hitboxInput.Size = UDim2.new(0, 80, 1, 0)
hitboxInput.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
hitboxInput.Text = tostring(hitboxSize)
hitboxInput.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxInput.TextSize = 14
hitboxInput.Font = Enum.Font.Gotham
hitboxInput.BorderSizePixel = 0
hitboxInput.PlaceholderText = "Value"
hitboxInput.TextXAlignment = Enum.TextXAlignment.Center
hitboxInput.Parent = hitboxInputFrame

local hitboxInputCorner = Instance.new("UICorner")
hitboxInputCorner.CornerRadius = UDim.new(0, 6)
hitboxInputCorner.Parent = hitboxInput

-- Button Apply Hitbox
local hitboxApplyBtn = Instance.new("TextButton")
hitboxApplyBtn.Size = UDim2.new(0, 90, 1, 0)
hitboxApplyBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
hitboxApplyBtn.Text = "Apply"
hitboxApplyBtn.TextSize = 12
hitboxApplyBtn.Font = Enum.Font.GothamBold
hitboxApplyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxApplyBtn.BorderSizePixel = 0
hitboxApplyBtn.Parent = hitboxInputFrame

local hitboxApplyCorner = Instance.new("UICorner")
hitboxApplyCorner.CornerRadius = UDim.new(0, 6)
hitboxApplyCorner.Parent = hitboxApplyBtn

local hitboxBtnFrame = Instance.new("Frame")
hitboxBtnFrame.Size = UDim2.new(1, -20, 0, 30)
hitboxBtnFrame.Position = UDim2.new(0, 10, 0, 90)
hitboxBtnFrame.BackgroundTransparency = 1
hitboxBtnFrame.Parent = hitboxFrame

local hitboxBtn = Instance.new("TextButton")
hitboxBtn.Size = UDim2.new(0, 180, 1, 0)
hitboxBtn.Position = UDim2.new(0.5, -90, 0, 0)
hitboxBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
hitboxBtn.Text = "ATIVAR HITBOX"
hitboxBtn.TextSize = 13
hitboxBtn.Font = Enum.Font.GothamBold
hitboxBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxBtn.BorderSizePixel = 0
hitboxBtn.Parent = hitboxBtnFrame

local hitboxBtnCorner = Instance.new("UICorner")
hitboxBtnCorner.CornerRadius = UDim.new(0, 8)
hitboxBtnCorner.Parent = hitboxBtn

-- Minimize Hub Functions
local function toggleMinimize()
    isMinimized = not isMinimized 
    if isMinimized then
        local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 350, 0, 40)
        })
        tween:Play() 
        scrollFrame.Visible = false
        minimizeBtn.Text = "□"      
    else
        local tween = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 350, 0, 280)
        })
        tween:Play() 
        tween.Completed:Connect(function()
            scrollFrame.Visible = true
        end) 
        minimizeBtn.Text = "_"
    end
end

-- ESP Functions 
local function createHighlight(character)
    if not character or highlights[character] then return end 
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESPHighlight"
    highlight.FillColor = Color3.fromRGB(100, 255, 100)
    highlight.FillTransparency = 0.8
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.OutlineTransparency = 0
    highlight.Parent = character
    highlights[character] = highlight
end

local function removeHighlight(character)
    if highlights[character] then
        highlights[character]:Destroy()
        highlights[character] = nil
    end
end

local function updateESP()
    if espEnabled then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                createHighlight(otherPlayer.Character)
            end
        end
    else
        for character, highlight in pairs(highlights) do
            highlight:Destroy()
        end
        highlights = {}
    end
end

-- Player distance meters and name functionality 
local function createESP(targetPlayer)
    if not targetPlayer or targetPlayer == player then return end 
    local function setupESPForCharacter(character)
        if not character or not espEnabled then return end 
        local head = character:FindFirstChild("Head")
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if not head or not rootPart then return end 
        createHighlight(character) 
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESPBillboard"
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = head 
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = targetPlayer.Name
        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLabel.TextSize = 14
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.TextStrokeTransparency = 0
        nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        nameLabel.Parent = billboard 
        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Size = UDim2.new(1, 0, 0.5, 0)
        distanceLabel.Position = UDim2.new(0, 0, 0.5, 0)
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.Text = "0m"
        distanceLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        distanceLabel.TextSize = 12
        distanceLabel.Font = Enum.Font.Gotham
        distanceLabel.TextStrokeTransparency = 0
        distanceLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        distanceLabel.Parent = billboard   
        local connection = RunService.Heartbeat:Connect(function()
            if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or
               not targetPlayer.Character or not targetPlayer.Character:FindFirstChild("HumanoidRootPart") or
               not espEnabled then
                return
            end  
            local distance = (player.Character.HumanoidRootPart.Position - targetPlayer.Character.HumanoidRootPart.Position).Magnitude
            distanceLabel.Text = math.floor(distance) .. "m"
        end)  
        espConnections[targetPlayer] = {connection, billboard, character}
    end  
    if targetPlayer.Character then
        setupESPForCharacter(targetPlayer.Character)
    end  
    local characterConnection = targetPlayer.CharacterAdded:Connect(function(character)
        if espEnabled then
            character:WaitForChild("Head")
            character:WaitForChild("HumanoidRootPart")
            wait(0.5)
            setupESPForCharacter(character)
        end
    end)
    local characterRemovedConnection = targetPlayer.CharacterRemoving:Connect(function(character)
        if espConnections[targetPlayer] then
            local connection, billboard = unpack(espConnections[targetPlayer])
            if connection then connection:Disconnect() end
            if billboard then billboard:Destroy() end
        end
        removeHighlight(character)
    end) 
    if not espConnections[targetPlayer] then
        espConnections[targetPlayer] = {nil, nil, nil, characterConnection, characterRemovedConnection}
    else
        espConnections[targetPlayer][4] = characterConnection
        espConnections[targetPlayer][5] = characterRemovedConnection
    end
end

local function removeESP(targetPlayer)
    if espConnections[targetPlayer] then
        local connection, billboard, character, charConn, charRemConn = unpack(espConnections[targetPlayer]) 
        if connection then connection:Disconnect() end
        if charConn then charConn:Disconnect() end
        if charRemConn then charRemConn:Disconnect() end
        if billboard then billboard:Destroy() end
        if character then removeHighlight(character) end
        espConnections[targetPlayer] = nil
    end
end

local function removeAllESP()
    for targetPlayer, _ in pairs(espConnections) do
        removeESP(targetPlayer)
    end
    for character, highlight in pairs(highlights) do
        highlight:Destroy()
    end
    highlights = {}
end

local function createESPForAllPlayers()
    for _, targetPlayer in pairs(Players:GetPlayers()) do
        if targetPlayer ~= player then
            createESP(targetPlayer)
        end
    end
end

local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        createESPForAllPlayers()  
        espUpdateConnection = RunService.Heartbeat:Connect(function()
            updateESP()
        end)  
        espStatus.Text = "Status: Active"
        espStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
        espBtn.Text = "Inactive"
        espBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        espFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 15)
    else
        removeAllESP()   
        if espUpdateConnection then
            espUpdateConnection:Disconnect()
            espUpdateConnection = nil
        end 
        espStatus.Text = "Status: Inactive"
        espStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
        espBtn.Text = "Active"
        espBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
        espFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)       
    end
end

-- Antikick functions but with Speed 
local function setupSpeedProtection()
    local getgenv, getnamecallmethod, hookmetamethod, hookfunction, newcclosure, checkcaller, gsub = getgenv, getnamecallmethod, hookmetamethod, hookfunction, newcclosure, checkcaller, string.gsub
    if getgenv and getgenv().SpeedProtection then
        return -- Já está ativo
    end
    local cloneref = cloneref or function(...) 
        return ...
    end
    local clonefunction = clonefunction or function(...)
        return ...
    end
    if not (getgenv and hookmetamethod and hookfunction) then
        print("SpeedProtection não suportado neste executor")
        return false
    end
    local Players, LocalPlayer = cloneref(game:GetService("Players")), cloneref(game:GetService("Players").LocalPlayer)
    local FindFirstChild = clonefunction(game.FindFirstChild)
    local CompareInstances = function(Instance1, Instance2)
        return (typeof(Instance1) == "Instance" and typeof(Instance2) == "Instance")
    end
    local CanCastToSTDString = function(...)
        return pcall(FindFirstChild, game, ...)
    end
    getgenv().SpeedProtection = {
        Enabled = true,
        CheckCaller = true
    }
    local OldNamecall; OldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(...)
        local self, message = ...
        local method = getnamecallmethod()
        if ((getgenv().SpeedProtection.CheckCaller and not checkcaller()) or true) and CompareInstances(self, LocalPlayer) and gsub(method, "^%l", string.upper) == "Kick" and getgenv().SpeedProtection.Enabled then
            if CanCastToSTDString(message) then
                return
            end
        end
        return OldNamecall(...)
    end))
    local OldFunction; OldFunction = hookfunction(LocalPlayer.Kick, function(...)
        local self, Message = ...
        if ((getgenv().SpeedProtection.CheckCaller and not checkcaller()) or true) and CompareInstances(self, LocalPlayer) and getgenv().SpeedProtection.Enabled then
            if CanCastToSTDString(Message) then
                return
            end
        end
    end) 
    return true
end

local function updateSpeedStatus()
    speedStatus.Text = "Status: " .. (speedEnabled and "ON" or "OFF") .. " - Velocidade: " .. currentSpeed
    speedStatus.TextColor3 = speedEnabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
    speedInput.Text = tostring(currentSpeed)
end

local function applySpeed(speed)
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.WalkSpeed = speed
    end
end

local function toggleSpeed()
    speedEnabled = not speedEnabled 
    if speedEnabled then
        -- SpeedProtection + Speed
        local speedProtectionSuccess = setupSpeedProtection() 
        applySpeed(currentSpeed)
        speedBtn.Text = "DISABLE"
        speedBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        speedFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 15)
        if speedProtectionSuccess then     
    else           
        end
    else
        applySpeed(normalSpeed)
        speedBtn.Text = "ENABLE"
        speedBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
        speedFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)   
    end
    updateSpeedStatus()
end

local function saveOriginalSettings()
    originalSettings = {
        Brightness = Lighting.Brightness,
        GlobalShadows = Lighting.GlobalShadows,
        FogEnd = Lighting.FogEnd,
        FogStart = Lighting.FogStart,
        Ambient = Lighting.Ambient,
        OutdoorAmbient = Lighting.OutdoorAmbient,
        ColorShift_Bottom = Lighting.ColorShift_Bottom,
        ColorShift_Top = Lighting.ColorShift_Top,
        ShadowSoftness = Lighting.ShadowSoftness,
        ExposureCompensation = Lighting.ExposureCompensation
    }
    pcall(function()
        originalRenderSettings = {
            QualityLevel = settings().Rendering.QualityLevel,
            MeshPartDetailLevel = settings().Rendering.MeshPartDetailLevel,
            TextureQuality = settings().Rendering.TextureQuality,
            ShadowQuality = settings().Rendering.ShadowQuality,
            AnisotropicFiltering = settings().Rendering.AnisotropicFiltering
        }
    end)
    
end

-- Reduce Performance
local function applyAntiLagSettings()
    pcall(function() 
        Lighting.Brightness = 11.3
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
        Lighting.Ambient = Color3.fromRGB(150, 150, 150)
        Lighting.OutdoorAmbient = Color3.fromRGB(150, 150, 150)
        Lighting.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)
        Lighting.ColorShift_Top = Color3.fromRGB(0, 0, 0)
        Lighting.ShadowSoftness = 0
        Lighting.ExposureCompensation = 0 
        local removed = 0
        local disabled = 0  
        for _, obj in pairs(workspace:GetDescendants()) do
            pcall(function()
                if obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Explosion") then
                    obj:Destroy()
                    removed = removed + 1
                elseif obj:IsA("SpotLight") or obj:IsA("PointLight") or obj:IsA("SurfaceLight") then
                    if obj.Enabled then
                        obj.Enabled = false
                        table.insert(disabledObjects, {obj = obj, property = "Enabled", originalValue = true})
                        disabled = disabled + 1
                    end
                elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then
                    if obj.Enabled then
                        obj.Enabled = false
                        table.insert(disabledObjects, {obj = obj, property = "Enabled", originalValue = true})
                        disabled = disabled + 1
                    end
                elseif obj:IsA("Decal") or obj:IsA("Texture") then
                    if obj.Transparency < 0.9 then
                        originalTextures[obj] = {transparency = obj.Transparency, texture = obj.Texture}
                        obj.Transparency = 0.9
                        obj.Texture = ""
                        disabled = disabled + 1
                    end
                elseif obj:IsA("SurfaceGui") or obj:IsA("BillboardGui") then
                    if obj.Enabled and obj ~= screenGui and not obj:IsDescendantOf(screenGui) then
                        obj.Enabled = false
                        table.insert(disabledObjects, {obj = obj, property = "Enabled", originalValue = true})
                        disabled = disabled + 1
                    end
                elseif obj:IsA("MeshPart") or obj:IsA("Part") or obj:IsA("UnionOperation") then
                    if obj.Material ~= Enum.Material.Plastic then
                        originalMaterials[obj] = obj.Material
                        obj.Material = Enum.Material.Plastic
                        disabled = disabled + 1
                    end
                    if obj.Reflectance > 0 then
                        obj.Reflectance = 0
                    end
                end
            end)
        end  
        pcall(function()
            local renderingSettings = settings().Rendering
            renderingSettings.QualityLevel = Enum.QualityLevel.Level01
            renderingSettings.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level01
            renderingSettings.TextureQuality = Enum.SavedQualitySetting.QualityLevel01
            renderingSettings.ShadowQuality = Enum.SavedQualitySetting.QualityLevel01
            renderingSettings.AnisotropicFiltering = Enum.SavedQualitySetting.QualityLevel01
        end)
    end)
end

local function restoreOriginalSettings()
    pcall(function()
        for property, value in pairs(originalSettings) do
            if Lighting[property] ~= nil then
                Lighting[property] = value
            end
        end
        pcall(function()
            local renderingSettings = settings().Rendering
            for property, value in pairs(originalRenderSettings) do
                if renderingSettings[property] ~= nil then
                    renderingSettings[property] = value
                end
            end
        end) 
        local restored = 0
        for _, data in pairs(disabledObjects) do
            pcall(function()
                if data.obj and data.obj.Parent then
                    data.obj[data.property] = data.originalValue
                    restored = restored + 1
                end
            end)
        end 
        for obj, data in pairs(originalTextures) do
            pcall(function()
                if obj and obj.Parent then
                    obj.Transparency = data.transparency
                    obj.Texture = data.texture
                    restored = restored + 1
                end
            end)
        end
        for obj, material in pairs(originalMaterials) do
            pcall(function()
                if obj and obj.Parent then
                    obj.Material = material
                    restored = restored + 1
                end
            end)
        end
        disabledObjects = {}
        originalTextures = {}
        originalMaterials = {}
    end)
end

local function toggleAntiLag()
    antiLagEnabled = not antiLagEnabled
    if antiLagEnabled then
        saveOriginalSettings()
        applyAntiLagSettings() 
        antiLagStatus.Text = "Status: Active"
        antiLagStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
        antiLagBtn.Text = "Inactive"
        antiLagBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        antiLagFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 15)            
    else
        restoreOriginalSettings()
        antiLagStatus.Text = "Status: Inactive"
        antiLagStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
        antiLagBtn.Text = "Active"
        antiLagBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
        antiLagFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)       
    end
end

-- Hitbox Features (BETA) 
local function updateHitboxStatus()
    hitboxStatus.Text = "Status: " .. (hitboxEnabled and "ON" or "OFF") .. " - Tamanho: " .. hitboxSize
    hitboxStatus.TextColor3 = hitboxEnabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
    hitboxInput.Text = tostring(hitboxSize)
end

local function createHitbox(targetPlayer)
    if not targetPlayer or targetPlayer == player or not targetPlayer.Character then return end    
    local character = targetPlayer.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end    
    if not hitboxParts[targetPlayer] then
        hitboxParts[targetPlayer] = {
            originalSize = humanoidRootPart.Size,
            originalTransparency = humanoidRootPart.Transparency,
            originalColor = humanoidRootPart.BrickColor
        }
    end  
    -- Expand Hitbox to combat, this is in Beta, may not work
    humanoidRootPart.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
    humanoidRootPart.Transparency = 0.8 -- Tornar visível para mostrar expansão
    humanoidRootPart.BrickColor = BrickColor.new("Really red")
end

local function removeHitbox(targetPlayer)
    if hitboxParts[targetPlayer] and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local humanoidRootPart = targetPlayer.Character.HumanoidRootPart
        humanoidRootPart.Size = hitboxParts[targetPlayer].originalSize
        humanoidRootPart.Transparency = hitboxParts[targetPlayer].originalTransparency
        humanoidRootPart.BrickColor = hitboxParts[targetPlayer].originalColor
     end
    hitboxParts[targetPlayer] = nil
end

local function removeAllHitboxes()
    for targetPlayer, data in pairs(hitboxParts) do
        ifr targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoidRootPart = targetPlayer.Character.HumanoidRootPart
            humanoidRootPart.Size = data.originalSize
            humanoidRootPart.Transparency = data.originalTransparency
            humanoidRootPart.BrickColor = data.originalColor
        end
    end
    hitboxParts = {}
end

local function createHitboxForAllPlayers()
    for _, targetPlayer in pairs(Players:GetPlayers()) do
        if targetPlayer ~= player and targetPlayer.Character then
            createHitbox(targetPlayer)
        end
    end
end

local function updateHitboxSizes()
    if hitboxEnabled then
        for targetPlayer, data in pairs(hitboxParts) do
            if targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                targetPlayer.Character.HumanoidRootPart.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
            end
        end
    end
end

local function toggleHitbox()
    hitboxEnabled = not hitboxEnabled
    if hitboxEnabled then
        createHitboxForAllPlayers()
        hitboxBtn.Text = "DISABLE"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
        hitboxFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 15)
    else
        removeAllHitboxes()
        hitboxBtn.Text = "ENABLE"
        hitboxBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
        hitboxFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    end
    updateHitboxStatus()
end

-- EVENTS
espBtn.MouseButton1Click:Connect(toggleESP)
speedBtn.MouseButton1Click:Connect(toggleSpeed)
antiLagBtn.MouseButton1Click:Connect(toggleAntiLag)
hitboxBtn.MouseButton1Click:Connect(toggleHitbox)
minimizeBtn.MouseButton1Click:Connect(toggleMinimize)

-- Speed Apply
applyBtn.MouseButton1Click:Connect(function()
    local value = tonumber(speedInput.Text)
    if value and value >= 1 and value <= 200 then
        currentSpeed = value
        if speedEnabled and player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid.WalkSpeed = currentSpeed
        end
        updateSpeedStatus()
    else
        speedInput.Text = tostring(currentSpeed)
        showErrorNotification("Valor Inválido", value and value > 200 and "Máximo permitido: 200" or "Use números entre 1-200")
    end
end)

-- Hitbox Apply
hitboxApplyBtn.MouseButton1Click:Connect(function()
    local value = tonumber(hitboxInput.Text)
    if value and value >= 1 and value <= 30 then
        hitboxSize = value
        updateHitboxSizes()
        updateHitboxStatus()
    else
        hitboxInput.Text = tostring(hitboxSize)
        showErrorNotification("Invalid Value", value and value > 25 and "Máximo permitido: 25" or "Use números entre 1-25")
    end
end)

-- Inputs com Enter
speedInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        applyBtn.MouseButton1Click:Fire()
    end
end)

hitboxInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        hitboxApplyBtn.MouseButton1Click:Fire()
    end
end)

-- Destroy functionality after removing the Hub 
closeBtn.MouseButton1Click:Connect(function()
    if espEnabled then
        espEnabled = false
        removeAllESP()
        if espUpdateConnection then
            espUpdateConnection:Disconnect()
            espUpdateConnection = nil
        end
    end
    if speedEnabled then
        speedEnabled = false
        applySpeed(normalSpeed)
    end 
    if antiLagEnabled then
        antiLagEnabled = false
        restoreOriginalSettings()
    end
    if hitboxEnabled then
        hitboxEnabled = false
        removeAllHitboxes()
    end
    espConnections = {}
    highlights = {}
    hitboxParts = {}
    disabledObjects = {}
    originalTextures = {}
    originalMaterials = {}
    originalSettings = {}
    originalRenderSettings = {}
    -- Destroy Gui 
    screenGui:Destroy()
end)

-- Shortcuts
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.E then
        toggleESP()
    elseif input.KeyCode == Enum.KeyCode.Q then
        toggleSpeed()
    elseif input.KeyCode == Enum.KeyCode.R then
        toggleAntiLag()
    elseif input.KeyCode == Enum.KeyCode.T then
        toggleHitbox()
    elseif input.KeyCode == Enum.KeyCode.M then
        toggleMinimize()
    end
end)

Players.PlayerAdded:Connect(function(newPlayer)    
    newPlayer.CharacterAdded:Connect(function(character)
        if espEnabled then
            wait(0.1)
            createHighlight(character)
            createESP(newPlayer)
        end
        if hitboxEnabled then
            wait(0.5)
            createHitbox(newPlayer)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(leavingPlayer)
    if espConnections[leavingPlayer] then
        removeESP(leavingPlayer)
    end
    if leavingPlayer.Character and highlights[leavingPlayer.Character] then
        removeHighlight(leavingPlayer.Character)
    end
    if hitboxParts[leavingPlayer] then
        removeHitbox(leavingPlayer)
    end
end)

player.CharacterAdded:Connect(function()
    wait(1)
    if speedEnabled then
        player.Character.Humanoid.WalkSpeed = currentSpeed
    end
    if hitboxEnabled then
        wait(2)
        createHitboxForAllPlayers()
    end
end)

-- Always check that the Speed has not been changed by the game
RunService.Heartbeat:Connect(function()
    if speedEnabled and player.Character and player.Character:FindFirstChild("Humanoid") then
        local humanoid = player.Character.Humanoid
        if humanoid.WalkSpeed ~= currentSpeed then
            humanoid.WalkSpeed = currentSpeed
        end
    end
end)

-- status
updateSpeedStatus()
updateHitboxStatus()

-- Ignore this
print("============================")
print("HUB LOADED!")
print("E - ESP | Q - Speed | R - Anti-Lag")
print("T - Hitbox | M - Minimizar")
print("============================")

-- Notification
spawn(function()
    wait(1)
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0, 320, 0, 40)
    notif.Position = UDim2.new(0.5, -160, 0, 50)
    notif.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    notif.Text = "😄Commands Hub loaded!"
    notif.TextColor3 = Color3.fromRGB(100, 255, 100)
    notif.TextSize = 12
    notif.Font = Enum.Font.GothamBod
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 8)
    notifCorner.Parent = notif
    local notifStroke = Instance.new("UIStroke")
    notifStroke.Color = Color3.fromRGB(100, 255, 100)
    notifStroke.Thickness = 1
    notifStroke.Parent = notif
    wait(3)
    notif:Destroy()
end) 
