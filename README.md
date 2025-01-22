local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESPs = {}
local ESPEnabled = false
local AimbotEnabled = false
local NoRecoilEnabled = false
local FOVSize = 150
local AimSmoothness = 5
local AntiLagEnabled = false
local SkyRemoved = false
local PanelVisible = true

-- Criar GUI do Painel Futurista
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 350, 0, 400)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Frame.BackgroundTransparency = 0.1
Frame.BorderSizePixel = 0
Frame.BorderRadius = UDim.new(0, 10) -- Bordas arredondadas
Frame.Parent = ScreenGui

local function createToggle(yOffset, label, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0, 100, 0, 30)
    toggleFrame.Position = UDim2.new(0.5, -50, 0, yOffset)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.BorderRadius = UDim.new(0, 8) -- Bordas arredondadas
    toggleFrame.Parent = Frame

    local toggleButton = Instance.new("Frame")
    toggleButton.Size = UDim2.new(0, 20, 0, 20)
    toggleButton.Position = UDim2.new(0, 2, 0, 2)
    toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    toggleButton.BorderRadius = UDim.new(0, 6) -- Bordas arredondadas para o botão
    toggleButton.Parent = toggleFrame

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0, 200, 0, 30)
    textLabel.Position = UDim2.new(0, 10, 0, yOffset)
    textLabel.Text = label
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.BackgroundTransparency = 1
    textLabel.TextSize = 16
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.Parent = Frame

    local isActive = false

    toggleFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isActive = not isActive
            if isActive then
                toggleButton:TweenPosition(UDim2.new(1, -22, 0, 2), "Out", "Sine", 0.2, true)
                toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            else
                toggleButton:TweenPosition(UDim2.new(0, 2, 0, 2), "Out", "Sine", 0.2, true)
                toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            end
            callback(isActive)
        end
    end)
end

createToggle(20, "ESP", function(state) ESPEnabled = state end)
createToggle(60, "Aimbot", function(state) AimbotEnabled = state end)
createToggle(100, "No Recoil", function(state) NoRecoilEnabled = state end)
createToggle(140, "FOV", function(state) FOVSize = state and 150 or 0 end)
createToggle(180, "Anti-Lag", function(state) AntiLagEnabled = state end)

-- Aba de Créditos
local CreditsButton = Instance.new("TextButton")
CreditsButton.Size = UDim2.new(0, 100, 0, 30)
CreditsButton.Position = UDim2.new(0.5, -50, 0, 280)
CreditsButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
CreditsButton.Text = "Créditos"
CreditsButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CreditsButton.Font = Enum.Font.SourceSansBold
CreditsButton.TextSize = 16
CreditsButton.BorderSizePixel = 0
CreditsButton.BorderRadius = UDim.new(0, 8)
CreditsButton.Parent = Frame

local CreditsFrame = Instance.new("Frame")
CreditsFrame.Size = UDim2.new(0, 300, 0, 150)
CreditsFrame.Position = UDim2.new(0.5, -150, 0, 100)
CreditsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
CreditsFrame.Visible = false
CreditsFrame.BorderSizePixel = 0
CreditsFrame.BorderRadius = UDim.new(0, 10)
CreditsFrame.Parent = ScreenGui

local CreditsText = Instance.new("TextLabel")
CreditsText.Size = UDim2.new(0, 280, 0, 140)
CreditsText.Position = UDim2.new(0, 10, 0, 10)
CreditsText.Text = "Script feito por: \nSeu Nome Aqui \nVersão: 1.0\n\nEspecial agradecimento ao desenvolvedor XYZ."
CreditsText.TextColor3 = Color3.fromRGB(255, 255, 255)
CreditsText.TextSize = 16
CreditsText.Font = Enum.Font.SourceSans
CreditsText.TextWrapped = true
CreditsText.BackgroundTransparency = 1
CreditsText.Parent = CreditsFrame

CreditsButton.MouseButton1Click:Connect(function()
    CreditsFrame.Visible = not CreditsFrame.Visible
end)

-- Criar ESP
local function CreateESP(player)
    if player == LocalPlayer or ESPs[player] then return end

    local highlight = Instance.new("Highlight")
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    
    player.CharacterAdded:Connect(function(char)
        highlight.Parent = char
    end)
    
    if player.Character then
        highlight.Parent = player.Character
    end

    ESPs[player] = highlight
end

local function UpdateESP()
    if not ESPEnabled then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if not ESPs[player] then
                CreateESP(player)
            end
        end
    end
end

local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOVSize

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local RootPart = player.Character.HumanoidRootPart
            local ScreenPosition, OnScreen = Camera:WorldToViewportPoint(RootPart.Position)

            if OnScreen then
                local MousePos = UserInputService:GetMouseLocation()
                local Distance = (Vector2.new(ScreenPosition.X, ScreenPosition.Y) - MousePos).Magnitude

                if Distance < shortestDistance then
                    closestPlayer = RootPart
                    shortestDistance = Distance
                end
            end
        end
    end
    return closestPlayer
end

local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Visible = false

local function UpdateFOV()
    if FOVSize > 0 then
        local MousePosition = UserInputService:GetMouseLocation()
        FOVCircle.Position = Vector2.new(MousePosition.X, MousePosition.Y)
        FOVCircle.Radius = FOVSize
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end

RunService.RenderStepped:Connect(function()
    UpdateFOV()
    UpdateESP()

    if AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = GetClosestPlayer()
        if target then
            local targetPos = target.Position + Vector3.new(0, 1.5, 0)
            local newCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            Camera.CFrame = Camera.CFrame:Lerp(newCFrame, 0.1 * AimSmoothness)
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        PanelVisible = not PanelVisible
        Frame.Visible = PanelVisible
    end
end)
