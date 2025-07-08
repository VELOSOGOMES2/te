local player = game.Players.LocalPlayer
local VirtualInputManager = game:GetService("VirtualInputManager")

-- 🛡️ Anti-Kick / Anti-Ban
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

-- 🖼️ UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaLogUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 320, 0, 220)
mainFrame.Position = UDim2.new(0, 20, 0.55, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local header = Instance.new("TextLabel", mainFrame)
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
header.Text = "💰 Log de Ganhos — Tianta"
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

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
logBox.Text = "🔍 Monitorando valores relacionados a dinheiro...\n"
logBox.RichText = true

-- 🧠 Funções
local tracked = {}

local keywords = {"money", "cash", "coin", "coins", "reward", "balance", "points"}

local function isMoneyRelated(name)
    name = name:lower()
    for _, keyword in ipairs(keywords) do
        if name:find(keyword) then return true end
    end
    return false
end

local function safeGetPath(obj)
    local success, result = pcall(function()
        return obj:GetFullName()
    end)
    return success and result or "[Erro ao obter caminho]"
end

local function monitor(obj)
    if tracked[obj] or not isMoneyRelated(obj.Name) then return end
    if obj:IsA("IntValue") or obj:IsA("NumberValue") or obj:IsA("StringValue") then
        tracked[obj] = obj.Value
        obj:GetPropertyChangedSignal("Value"):Connect(function()
            local old = tracked[obj]
            local new = obj.Value
            if new ~= old then
                tracked[obj] = new
                local diff = (typeof(old) == "number" and typeof(new) == "number") and (new - old) or ""
                local msg = os.date("[%H:%M:%S] ") .. safeGetPath(obj) .. " ➜ " .. tostring(old) .. " → " .. tostring(new)
                if diff ~= "" then
                    msg ..= " (" .. (diff >= 0 and "+" or "") .. diff .. ")"
                end
                logBox.Text = msg .. "\n" .. logBox.Text
            end
        end)
    end
end

-- Escaneia o jogo
local function deepScan(root)
    for _, obj in ipairs(root:GetDescendants()) do
        monitor(obj)
    end
end

deepScan(game)

-- Novos objetos criados depois
game.DescendantAdded:Connect(function(obj)
    wait(0.1)
    monitor(obj)
end)
