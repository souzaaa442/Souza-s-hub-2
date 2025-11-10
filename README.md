
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Remove GUI antiga de demo se existir
local old = playerGui:FindFirstChild("GengarDemoGUI")
if old then old:Destroy() end

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "GengarDemoGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Container principal
local frame = Instance.new("Frame")
frame.Name = "MainFrame"
frame.Size = UDim2.new(0, 380, 0, 480)
frame.Position = UDim2.new(0.5, -190, 0.5, -240)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.BackgroundColor3 = Color3.fromRGB(40, 10, 60) -- roxo escuro
frame.BorderSizePixel = 0
frame.Parent = screenGui
frame.BackgroundTransparency = 0.05
frame.ClipsDescendants = true
frame.Active = true

-- Sombra / borda suave
local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0, 14)

-- Top bar com imagem do Gengar
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 120)
topBar.Position = UDim2.new(0, 0, 0, 0)
topBar.BackgroundTransparency = 1
topBar.Parent = frame

local gengarImage = Instance.new("ImageLabel")
gengarImage.Size = UDim2.new(0, 120, 0, 120)
gengarImage.Position = UDim2.new(0, 12, 0, 12)
gengarImage.BackgroundTransparency = 1
gengarImage.Image = "rbxassetid://PUT_GENGAR_ASSET_ID_AQUI" -- substitua aqui
gengarImage.Parent = topBar
gengarImage.ScaleType = Enum.ScaleType.Crop

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -156, 0, 120)
title.Position = UDim2.new(0, 144, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Gengar UI — Painel"
title.Font = Enum.Font.GothamBold
title.TextSize = 26
title.TextColor3 = Color3.fromRGB(230, 220, 255)
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = topBar

-- Área de opções
local optionsFrame = Instance.new("Frame")
optionsFrame.Size = UDim2.new(1, -24, 1, -156)
optionsFrame.Position = UDim2.new(0, 12, 0, 140)
optionsFrame.BackgroundTransparency = 1
optionsFrame.Parent = frame

local uiList = Instance.new("UIListLayout")
uiList.Padding = UDim.new(0, 10)
uiList.HorizontalAlignment = Enum.HorizontalAlignment.Center
uiList.SortOrder = Enum.SortOrder.LayoutOrder
uiList.Parent = optionsFrame

-- Função auxiliar pra criar botão estiloso
local function makeButton(text, layoutOrder)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 46)
    btn.BackgroundColor3 = Color3.fromRGB(95, 35, 150)
    btn.AutoButtonColor = true
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 18
    btn.TextColor3 = Color3.fromRGB(245, 240, 255)
    btn.Text = text
    btn.LayoutOrder = layoutOrder or 1
    btn.Parent = optionsFrame
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 8)
    return btn
end

-- Botões de exemplo (não fazem exploits)
local btnToggleESP = makeButton("Opção: ESP (simulada)", 1)
local btnToggleGlow = makeButton("Opção: Glow (simulada)", 2)
local btnFastWalk = makeButton("Opção: Vel. (simulada)", 3)
local btnSettings = makeButton("Configurações", 4)

-- Indicador de estado (label)
local stateLabel = Instance.new("TextLabel")
stateLabel.Size = UDim2.new(1, -20, 0, 36)
stateLabel.Text = "Estados: ESP: OFF  |  Glow: OFF  |  Vel: OFF"
stateLabel.TextColor3 = Color3.fromRGB(210, 200, 255)
stateLabel.Font = Enum.Font.Gotham
stateLabel.TextSize = 14
stateLabel.BackgroundTransparency = 1
stateLabel.LayoutOrder = 5
stateLabel.Parent = optionsFrame

-- Simple toggles (local & estéticos)
local states = {esp = false, glow = false, vel = false}
local function updateStateLabel()
    stateLabel.Text = string.format("Estados: ESP: %s  |  Glow: %s  |  Vel: %s",
        states.esp and "ON" or "OFF",
        states.glow and "ON" or "OFF",
        states.vel and "ON" or "OFF")
end

btnToggleESP.MouseButton1Click:Connect(function()
    states.esp = not states.esp
    updateStateLabel()
    -- aqui você pode tocar som ou animar um ícone, MAS NADA que interfira no jogo
end)

btnToggleGlow.MouseButton1Click:Connect(function()
    states.glow = not states.glow
    updateStateLabel()
end)

btnFastWalk.MouseButton1Click:Connect(function()
    states.vel = not states.vel
    updateStateLabel()
end)

btnSettings.MouseButton1Click:Connect(function()
    -- Exemplo: abrir modal de configurações (simulado)
    local modal = Instance.new("Frame")
    modal.Size = UDim2.new(0, 320, 0, 220)
    modal.Position = UDim2.new(0.5, -160, 0.5, -110)
    modal.AnchorPoint = Vector2.new(0.5, 0.5)
    modal.BackgroundColor3 = Color3.fromRGB(30, 8, 50)
    modal.Parent = screenGui
    local mc = Instance.new("UICorner", modal); mc.CornerRadius = UDim.new(0, 12)

    local lbl = Instance.new("TextLabel", modal)
    lbl.Size = UDim2.new(1, -24, 0, 44)
    lbl.Position = UDim2.new(0, 12, 0, 12)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 18
    lbl.TextColor3 = Color3.fromRGB(230,230,255)
    lbl.Text = "Configurações (demo)"

    local closeBtn = Instance.new("TextButton", modal)
    closeBtn.Size = UDim2.new(0, 80, 0, 36)
    closeBtn.Position = UDim2.new(1, -92, 1, -48)
    closeBtn.Text = "Fechar"
    closeBtn.Font = Enum.Font.Gotham
    closeBtn.TextSize = 14
    local cc = Instance.new("UICorner", closeBtn); cc.CornerRadius = UDim.new(0,8)
    closeBtn.MouseButton1Click:Connect(function()
        modal:Destroy()
    end)
end)
