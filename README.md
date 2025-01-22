local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variáveis do script
local ESPEnabled = true
local AimbotEnabled = true
local NoRecoilEnabled = true
local FOVSize = 150
local AimSmoothness = 5
local PanelVisible = true

-- Criar GUI do Painel
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 300, 0, 350)
MainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Visible = true

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 10)

-- Título do painel
local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundTransparency = 1
Title.Text = "Painel Futurista"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Botões para alternar abas
local TabFrame = Instance.new("Frame", MainFrame)
TabFrame.Size = UDim2.new(1, 0, 0, 40)
TabFrame.Position = UDim2.new(0, 0, 0, 50)
TabFrame.BackgroundTransparency = 1

local MainTabButton = Instance.new("TextButton", TabFrame)
MainTabButton.Size = UDim2.new(0.5, -5, 1, 0)
MainTabButton.Position = UDim2.new(0, 0, 0, 0)
MainTabButton.Text = "Configurações"
MainTabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MainTabButton.Font = Enum.Font.Gotham
MainTabButton.TextSize = 14
MainTabButton.TextColor3 = Color3.fromRGB(255, 255, 255)

local CreditsTabButton = Instance.new("TextButton", TabFrame)
CreditsTabButton.Size = UDim2.new(0.5, -5, 1, 0)
CreditsTabButton.Position = UDim2.new(0.5, 5, 0, 0)
CreditsTabButton.Text = "Créditos"
CreditsTabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
CreditsTabButton.Font = Enum.Font.Gotham
CreditsTabButton.TextSize = 14
CreditsTabButton.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Aba principal (Configurações)
local MainTab = Instance.new("Frame", MainFrame)
MainTab.Size = UDim2.new(1, 0, 1, -90)
MainTab.Position = UDim2.new(0, 0, 0, 90)
MainTab.BackgroundTransparency = 1
MainTab.Visible = true

local function createToggle(label, yOffset, callback)
    local ToggleFrame = Instance.new("Frame", MainTab)
    ToggleFrame.Size = UDim2.new(1, -20, 0, 40)
    ToggleFrame.Position = UDim2.new(0, 10, 0, yOffset)
    ToggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

    local UICorner = Instance.new("UICorner", ToggleFrame)
    UICorner.CornerRadius = UDim.new(0, 8)

    local ToggleLabel = Instance.new("TextLabel", ToggleFrame)
    ToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    ToggleLabel.Position = UDim2.new(0, 10, 0, 0)
    ToggleLabel.Text = label
    ToggleLabel.Font = Enum.Font.Gotham
    ToggleLabel.TextSize = 16
    ToggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleLabel.BackgroundTransparency = 1

    local ToggleButton = Instance.new("TextButton", ToggleFrame)
    ToggleButton.Size = UDim2.new(0.2, 0, 0.8, 0)
    ToggleButton.Position = UDim2.new(0.75, 0, 0.1, 0)
    ToggleButton.Text = "Off"
    ToggleButton.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    ToggleButton.Font = Enum.Font.GothamBold
    ToggleButton.TextSize = 14
    ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)

    local isActive = false
    ToggleButton.MouseButton1Click:Connect(function()
        isActive = not isActive
        ToggleButton.Text = isActive and "On" or "Off"
        ToggleButton.BackgroundColor3 = isActive and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(100, 0, 0)
        callback(isActive)
    end)
end

createToggle("ESP", 0, function(state) ESPEnabled = state end)
createToggle("Aimbot", 50, function(state) AimbotEnabled = state end)
createToggle("No Recoil", 100, function(state) NoRecoilEnabled = state end)

-- Aba de créditos
local CreditsTab = Instance.new("Frame", MainFrame)
CreditsTab.Size = UDim2.new(1, 0, 1, -90)
CreditsTab.Position = UDim2.new(0, 0, 0, 90)
CreditsTab.BackgroundTransparency = 1
CreditsTab.Visible = false

local CreditsText = Instance.new("TextLabel", CreditsTab)
CreditsText.Size = UDim2.new(1, -20, 1, -20)
CreditsText.Position = UDim2.new(0, 10, 0, 10)
CreditsText.Text = "Script feito por Octam\nObrigado por usar!"
CreditsText.Font = Enum.Font.Gotham
CreditsText.TextSize = 16
CreditsText.TextColor3 = Color3.fromRGB(255, 255, 255)
CreditsText.BackgroundTransparency = 1
CreditsText.TextWrapped = true

-- Alternar abas
MainTabButton.MouseButton1Click:Connect(function()
    MainTab.Visible = true
    CreditsTab.Visible = false
end)

CreditsTabButton.MouseButton1Click:Connect(function()
    MainTab.Visible = false
    CreditsTab.Visible = true
end)

-- Toggle do painel com a tecla Insert
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        PanelVisible = not PanelVisible
        MainFrame.Visible = PanelVisible
    end
end)
