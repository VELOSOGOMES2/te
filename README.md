local player = game.Players.LocalPlayer

local function boostSpeed()
    local gui = player:FindFirstChild("PlayerGui")
    if not gui then return warn("❗ PlayerGui não encontrado") end

    local chassisGui = gui:FindFirstChild("ChassisGui")
    if not chassisGui then return warn("❗ ChassisGui não encontrado") end

    -- Percorre os LocalScripts e valores internos
    for _, descendant in pairs(chassisGui:GetDescendants()) do
        if descendant:IsA("NumberValue") or descendant:IsA("IntValue") then
            if descendant.Value > 0 then
                print("🔧 Alterando:", descendant.Name, "de", descendant.Value, "para", descendant.Value * 3)
                descendant.Value = descendant.Value * 3
            end
        end
    end

    print("🚀 Boost de velocidade aplicado com sucesso!")
end

boostSpeed()
