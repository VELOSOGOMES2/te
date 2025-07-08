local VirtualInputManager = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local car = nil

-- 🛡️ Anti-Cheat, Anti-Kick e Anti-Ban
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🚫 Kick bloqueado!")
            return nil
        end
        return oldNamecall(self, ...)
    end)
end)

-- 🚘 Detectar carro
local function getCar()
    if not player.Character then return nil end
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        return seat:FindFirstAncestorOfClass("Model")
    end
    return nil
end

-- 📊 Mostrar stats do carro
local function showStats()
    task.spawn(function()
        while wait(0.2) do
            car = getCar()
            if car and car:FindFirstChild("Engine") then
                local speed = math.floor((car.PrimaryPart.Velocity.Magnitude * 3.6))
                local rpm = car.Engine:FindFirstChild("CurrentRPM") and math.floor(car.Engine.CurrentRPM.Value) or 0
                local gear = car.Engine:FindFirstChild("CurrentGear") and car.Engine.CurrentGear.Value or "?"
                print("🚗 Velocidade:", speed, "KM/H | RPM:", rpm, "| Marcha:", gear)
            end
        end
    end)
end

-- 🚀 Boost de velocidade seguro
local function toggleSpeedBoost(state)
    if not car then return end
    local bodyVelocity = Instance.new("BodyVelocity", car.PrimaryPart)
    bodyVelocity.MaxForce = Vector3.new(1e5, 0, 1e5)
    bodyVelocity.Velocity = car.PrimaryPart.CFrame.LookVector * (state and 250 or 0)
    wait(2)
    bodyVelocity:Destroy()
end

-- ✨ Teleport fixo
local teleportPosition = CFrame.new(100, 10, 1000)
local function teleportCar()
    car = getCar()
    if car then
        car:SetPrimaryPartCFrame(teleportPosition)
    end
end

-- 💰 AutoFarm simples
local autofarm = false
local function startAutoFarm()
    autofarm = true
    task.spawn(function()
        while autofarm do
            car = getCar()
            if car then
                VirtualInputManager:SendKeyEvent(true, "W", false, game)
                wait(2)
                VirtualInputManager:SendKeyEvent(false, "W", false, game)
                wait(1)
                teleportCar()
                wait(1)
            else
                warn("🚗 Entra no carro para usar AutoFarm")
                wait(3)
            end
        end
    end)
end

local function stopAutoFarm()
    autofarm = false
    VirtualInputManager:SendKeyEvent(false, "W", false, game)
end

-- 🖼️ Interface gráfica
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "TiantaModMenu"

local frame = Instance.new("Frame", gui)
frame.Position = UDim2.new(0, 30, 0.3, 0)
frame.Size = UDim2.new(0, 250, 0, 220)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Text = "🌸 Tianta — Mod Menu"
title.Size = UDim2.new(1, 0, 0, 30)
title.TextColor3 = Color3.new(1, 0.6, 0.8)
title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
title.Font = Enum.Font.GothamBold
title.TextSize = 16

local function newButton(txt, order, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, -20, 0, 35)
    btn.Position = UDim2.new(0, 10, 0, 40 + (order * 40))
    btn.Text = txt
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Botões
local farmBtn = newButton("💰 AutoFarm", 0, function()
    if autofarm then stopAutoFarm() else startAutoFarm() end
end)

local speedBtn = newButton("🚀 Boost de Velocidade", 1, function()
    toggleSpeedBoost(true)
end)

local tpBtn = newButton("📍 Teleport Fixo", 2, teleportCar)

local statsBtn = newButton("📊 Stats do Carro", 3, showStats)

local closeBtn = newButton("❌ Fechar Menu", 4, function()
    gui:Destroy()
end)
