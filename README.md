local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TiantaLogUI"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 350, 0, 230)
frame.Position = UDim2.new(0, 30, 0.5, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local header = Instance.new("TextLabel", frame)
header.Size = UDim2.new(1, 0, 0, 30)
header.Text = "📊 Log de Ganhos — Tianta"
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
box.Text = "🔍 Aguardando ganhos...\n"

local function log(text)
    box.Text = "["..os.date("%H:%M:%S").."] "..text.."\n" .. box.Text
end

-- 🔎 Detectar mudanças numéricas com nome visual
local keywords = {"cash", "money", "$", "coin", "saldo"}

local function trackGuiMoney()
    for _, gui in ipairs(player.PlayerGui:GetDescendants()) do
        if gui:IsA("TextLabel") or gui:IsA("TextButton") then
            local txt = gui.Text:lower()
            for _, word in ipairs(keywords) do
                if txt:find(word) or txt:find("%$") then
                    log("🎯 Monitorando: " .. gui:GetFullName())
                    local lastText = gui.Text
                    gui:GetPropertyChangedSignal("Text"):Connect(function()
                        if gui.Text ~= lastText then
                            log("💸 Ganhou algo: " .. gui.Text)
                            lastText = gui.Text
                        end
                    end)
                    break
                end
            end
        end
    end
end

trackGuiMoney()

-- Detecta novas GUIs que surgirem
player.PlayerGui.DescendantAdded:Connect(function(obj)
    wait(0.1)
    if obj:IsA("TextLabel") or obj:IsA("TextButton") then
        local txt = obj.Text:lower()
        for _, word in ipairs(keywords) do
            if txt:find(word) or txt:find("%$") then
                log("🎯 Novo valor detectado: " .. obj:GetFullName())
                local lastText = obj.Text
                obj:GetPropertyChangedSignal("Text"):Connect(function()
                    if obj.Text ~= lastText then
                        log("💸 Mudança: " .. obj.Text)
                        lastText = obj.Text
                    end
                end)
                break
            end
        end
    end
end)
