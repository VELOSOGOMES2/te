local player = game.Players.LocalPlayer

local function getCar()
    local character = player.Character
    if not character then return nil end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.SeatPart then
        local seat = humanoid.SeatPart
        local vehicle = seat:FindFirstAncestorOfClass("Model")
        return vehicle
    end
    return nil
end

local function boostCarSpeed(car)
    for _, obj in ipairs(car:GetDescendants()) do
        if obj:IsA("BodyVelocity") then
            obj.Velocity = Vector3.new(0, 0, 200) -- Boost direto
        elseif obj.Name:lower():find("engine") or obj.Name:lower():find("torque") or obj.Name:lower():find("speed") then
            if obj:IsA("NumberValue") then
                obj.Value = obj.Value * 3
            end
        elseif obj:IsA("VehicleSeat") then
            obj.MaxSpeed = 300
            obj.Torque = 10000
            obj.TurnSpeed = 100
        end
    end
end

local car = getCar()
if car then
    boostCarSpeed(car)
    print("🚀 Velocidade aumentada com sucesso!")
else
    warn("❗ Entra no carro antes de usar o boost.")
end
