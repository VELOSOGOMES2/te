-- 📜 Log de Ganhos (Base Tianta Menu)
local player = game.Players.LocalPlayer

-- 🧠 Memória de ganhos
local trackedValues = {}

-- 📊 UI Estilo AutoFarm
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaLogUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 300, 0, 200)
mainFrame.Position = UDim2.new(0, 280, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
header.Text = "📋 LOG DE GANHOS"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- 🖱️ Arrastar menu
local dragging = false
local dragStart, startPos
header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)
header.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- 📦 Área de texto
local logBox = Instance.new("TextLabel", mainFrame)
logBox.Size = UDim2.new(1, -20, 1, -50)
logBox.Position = UDim2.new(0, 10, 0, 40)
logBox.BackgroundTransparency = 1
logBox.TextColor3 = Color3.new(1, 1, 1)
logBox.Font = Enum.Font.Gotham
logBox.TextSize = 14
logBox.TextXAlignment = Enum.TextXAlignment.Left
logBox.TextYAlignment = Enum.TextYAlignment.Top
logBox.TextWrapped = true
logBox.Text = "Aguardando mudanças..."
logBox.TextScaled = false
logBox.TextTruncate = Enum.TextTruncate.AtEnd
logBox.TextStrokeTransparency = 1
logBox.TextYAlignment = Enum.TextYAlignment.Top
logBox.Text = ""

-- ❌ Botão fechar
local close = Instance.new("TextButton", mainFrame)
close.Size = UDim2.new(0, 25, 0, 25)
close.Position = UDim2.new(1, -30, 0, 2)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(120, 0, 0)
close.TextColor3 = Color3.new(1, 1, 1)
close.Font = Enum.Font.GothamBold
close.TextSize = 16
close.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- 🔍 Monitora valores do jogador
for _, stat in ipairs(player:WaitForChild("leaderstats"):GetChildren()) do
    if stat:IsA("IntValue") or stat:IsA("NumberValue") then
        trackedValues[stat.Name] = stat.Value
        stat:GetPropertyChangedSignal("Value"):Connect(function()
            local old = trackedValues[stat.Name]
            local new = stat.Value
            local diff = new - old
            trackedValues[stat.Name] = new

            if diff ~= 0 then
                local txt = os.date("[%H:%M:%S] ") .. stat.Name .. ": " .. (diff > 0 and "+" or "") .. tostring(diff)
                logBox.Text = txt .. "\n" .. logBox.Text
            end
        end)
    end
end
