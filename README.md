local player = game.Players.LocalPlayer
local VirtualInputManager = game:GetService("VirtualInputManager")

-- Anti-Cheat + Anti-Ban 🛡️
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local old = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🚫 Kick bloqueado")
            return
        elseif tostring(self) == "Ban" or method == "Ban" then
            warn("🚫 Ban detectado")
            return
        end
        return old(self, ...)
    end)
end)

-- UI 📦
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaLogUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 300, 0, 200)
mainFrame.Position = UDim2.new(0, 20, 0.55, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
header.Text = "💰 Log de Ganhos — Tianta"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

-- Arrastar 🖱️
local dragging, dragStart, startPos = false, nil, nil
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

-- Caixa de log 📄
local logBox = Instance.new("TextBox", mainFrame)
logBox.Size = UDim2.new(1, -10, 1, -40)
logBox.Position = UDim2.new(0, 5, 0, 35)
logBox.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
logBox.TextColor3 = Color3.new(0, 1, 0)
logBox.ClearTextOnFocus = false
logBox.MultiLine = true
logBox.TextWrapped = false
logBox.TextXAlignment = Enum.TextXAlignment.Left
logBox.TextYAlignment = Enum.TextYAlignment.Top
logBox.TextSize = 13
logBox.Font = Enum.Font.Code
logBox.Text = "🔍 Aguardando ganhos...\n"
logBox.RichText = true

-- Funções úteis
local function getCar()
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        return seat:FindFirstAncestorOfClass("Model")
    end
end

-- Monitorar valores dentro do jogador
local trackedValues = {}

local function monitorValue(val)
    if not (val:IsA("IntValue") or val:IsA("NumberValue")) then return end
    trackedValues[val] = val.Value
    val:GetPropertyChangedSignal("Value"):Connect(function()
        local old = trackedValues[val]
        local new = val.Value
        if new ~= old then
            local diff = new - old
            trackedValues[val] = new
            local path = val:GetFullName()
            local sign = (diff > 0 and "+" or "")
            local msg = os.date("[%H:%M:%S] ") .. path .. ": " .. sign .. tostring(diff)
            logBox.Text = msg .. "\n" .. logBox.Text
        end
    end)
end

-- Escanear jogador
local function scanAll(obj)
    for _, v in pairs(obj:GetDescendants()) do
        monitorValue(v)
    end
end

-- Ativar log apenas com carro andando
task.spawn(function()
    while true do
        local car = getCar()
        if car then
            local seat = player.Character and player.Character:FindFirstChildOfClass("Humanoid") and player.Character:FindFirstChildOfClass("Humanoid").SeatPart
            if seat and seat.Throttle ~= 0 then
                scanAll(player)
            end
        end
        wait(1)
    end
end)

-- Caso adicionem valores novos depois
player.DescendantAdded:Connect(function(child)
    wait(0.5)
    monitorValue(child)
end)
