local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = Players.LocalPlayer

-- 🛡️ Anti-Cheat e Anti-Kick
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall

    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(self) == "Kick" or method == "Kick" then
            warn("🛡️ Kick bloqueado!")
            return nil
        end
        return oldNamecall(self, ...)
    end)
end)

-- ⚙️ Variáveis
local autoFarm = false
local speedBoost = false
local teleportPoints = {
    ["🏁 Corrida"] = CFrame.new(1889, 30, -1609),
    ["🏠 Garagem"] = CFrame.new(1327, 9, -481),
    ["🌉 Ponte"] = CFrame.new(3985, 40, 200)
}
local startFarmPos = CFrame.new(1920.9, 30.8, -1610.7)
local endZ = -2598
local carForce = 50000
local shownGains = {}

-- 🛎️ Notificação
local function notify(txt)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "🧠 Tianta",
            Text = txt,
            Duration = 3
        })
    end)
end

-- 🚘 Detecta o carro atual
local function getCar()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        local car = seat:FindFirstAncestorOfClass("Model")
        if car and car.PrimaryPart then
            return car
        end
    end
end

-- 🧍‍♂️ Está no carro?
local function isInCar()
    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart
end

-- ⌨️ Tecla W
local function pressW(state)
    VirtualInputManager:SendKeyEvent(state, "W", false, game)
end

-- 💸 Espiar valores ganhos
local function spyGains()
    local old = {}
    RunService.Heartbeat:Connect(function()
        for _, obj in pairs(player:GetDescendants()) do
            if obj:IsA("NumberValue") and not shownGains[obj] then
                obj:GetPropertyChangedSignal("Value"):Connect(function()
                    local newVal = obj.Value
                    local oldVal = old[obj] or 0
                    if newVal ~= oldVal then
                        notify("💵 +" .. tostring(newVal - oldVal) .. " em: " .. obj.Name)
                        old[obj] = newVal
                    end
                end)
                old[obj] = obj.Value
                shownGains[obj] = true
            end
        end
    end)
end

-- 🚀 Força extra
local function applySpeedBoost(car)
    if not car then return end
    for _, v in pairs(car:GetDescendants()) do
        if v:IsA("BodyVelocity") or v:IsA("LinearVelocity") then
            v:Destroy()
        end
    end

    if speedBoost then
        local force = Instance.new("BodyVelocity")
        force.Velocity = Vector3.new(0, 0, -carForce)
        force.MaxForce = Vector3.new(0, 0, math.huge)
        force.P = 15000
        force.Parent = car.PrimaryPart
    end
end

-- 🗺️ Teleport
local function teleportTo(name)
    local cf = teleportPoints[name]
    if cf then
        local car = getCar()
        if car then
            car:SetPrimaryPartCFrame(cf)
        else
            (player.Character or player.CharacterAdded:Wait()):PivotTo(cf)
        end
    end
end

-- 💻 Interface
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "TiantaMod"

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 260, 0, 300)
main.Position = UDim2.new(0, 20, 0.4, 0)
main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
main.BorderSizePixel = 0

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "🎛️ Tianta Mod Menu"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundColor3 = Color3.fromRGB(50,50,50)
title.Font = Enum.Font.GothamBold
title.TextSize = 14

local function createButton(name, yPos, callback)
    local btn = Instance.new("TextButton", main)
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.Text = name
    btn.MouseButton1Click:Connect(callback)
end

-- Botões
createButton("💸 Iniciar AutoFarm", 40, function()
    autoFarm = not autoFarm
    notify(autoFarm and "AutoFarm Ligado" or "AutoFarm Desligado")

    if autoFarm then
        task.spawn(function()
            local car = getCar()
            if not car then repeat car = getCar() wait(1) until car end
            wait(0.3)
            car:SetPrimaryPartCFrame(startFarmPos)
            pressW(true)
            while autoFarm do
                if not isInCar() then
                    autoFarm = false
                    pressW(false)
                    notify("⛔ Saiu do carro")
                    return
                end
                local car = getCar()
                if car and car.PrimaryPart.Position.Z <= endZ then
                    car:SetPrimaryPartCFrame(startFarmPos)
                    wait(1)
                end
                wait(0.1)
            end
            pressW(false)
        end)
    else
        pressW(false)
    end
end)

createButton("🚀 Boost Velocidade", 80, function()
    speedBoost = not speedBoost
    notify(speedBoost and "🚀 Boost Ativado" or "⚠️ Boost Desativado")
    applySpeedBoost(getCar())
end)

createButton("🧭 Teleport: Corrida", 120, function() teleportTo("🏁 Corrida") end)
createButton("🧭 Teleport: Garagem", 160, function() teleportTo("🏠 Garagem") end)
createButton("🧭 Teleport: Ponte", 200, function() teleportTo("🌉 Ponte") end)
createButton("👀 Ativar Log de Ganhos", 240, spyGains)

-- Draggable
local dragging, dragStart, startPos
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
    end
end)
title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)
title.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Início
notify("🎮 Tianta Mod Ativado!")
