local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaScannerUI"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 350, 0, 250)
frame.Position = UDim2.new(0, 30, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)

local header = Instance.new("TextLabel", frame)
header.Size = UDim2.new(1, 0, 0, 30)
header.Text = "🧪 Scanner de Valores — Tianta"
header.TextColor3 = Color3.new(1,1,1)
header.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
header.Font = Enum.Font.GothamBold
header.TextSize = 14

local box = Instance.new("TextBox", frame)
box.Position = UDim2.new(0, 5, 0, 35)
box.Size = UDim2.new(1, -10, 1, -40)
box.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
box.TextColor3 = Color3.fromRGB(0, 255, 0)
box.ClearTextOnFocus = false
box.MultiLine = true
box.TextWrapped = false
box.TextXAlignment = Enum.TextXAlignment.Left
box.TextYAlignment = Enum.TextYAlignment.Top
box.TextSize = 13
box.Font = Enum.Font.Code
box.Text = "🔍 Aguardando valores...\n"

local function log(text)
    box.Text = "["..os.date("%H:%M:%S").."] "..text.."\n" .. box.Text
end

local function isLikelyMoney(str)
    local s = tostring(str):gsub("[,%$ ]", "") -- remove vírgulas e $
    local num = tonumber(s)
    return num and num > 1000 -- valor mínimo razoável para dinheiro
end

local function monitorText(obj)
    local last = obj.Text
    if isLikelyMoney(last) then
        log("💰 Observando: "..obj:GetFullName().." = "..last)
        obj:GetPropertyChangedSignal("Text"):Connect(function()
            if obj.Text ~= last and isLikelyMoney(obj.Text) then
                last = obj.Text
                log("📈 Novo valor: "..obj.Text)
            end
        end)
    end
end

local function scanGuiTree()
    for _, obj in ipairs(player.PlayerGui:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
            pcall(function() monitorText(obj) end)
        end
    end
end

scanGuiTree()

player.PlayerGui.DescendantAdded:Connect(function(obj)
    wait(0.2)
    if obj:IsA("TextLabel") or obj:IsA("TextButton") then
        pcall(function() monitorText(obj) end)
    end
end)
