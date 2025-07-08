-- Adiciona isso abaixo dos seus botões no menu principal
local turboAtivo = false
local turboForce

local turboButton = Instance.new("TextButton", mainFrame)
turboButton.Size = UDim2.new(1, -20, 0, 40)
turboButton.Position = UDim2.new(0, 10, 0, 85)
turboButton.Text = "🚀 Turbo OFF"
turboButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
turboButton.TextColor3 = Color3.new(1, 1, 1)
turboButton.Font = Enum.Font.GothamBold
turboButton.TextSize = 16

-- Função para aplicar/remover turbo
local function aplicarTurbo(estado)
    local car = getCar()
    if not car then
        notify("❗ Entra no carro primeiro!")
        return
    end

    if estado then
        if turboForce and turboForce.Parent then
            turboForce:Destroy()
        end

        turboForce = Instance.new("BodyVelocity")
        turboForce.Name = "TiantaTurbo"
        turboForce.MaxForce = Vector3.new(1e5, 0, 1e5)  -- Apenas força horizontal
        turboForce.Velocity = car.PrimaryPart.CFrame.LookVector * 300  -- Ajustável
        turboForce.P = 10000
        turboForce.Parent = car.PrimaryPart

        notify("🚀 Turbo ativado")
    else
        if turboForce and turboForce.Parent then
            turboForce:Destroy()
        end
        notify("⛔ Turbo desligado")
    end
end

-- Evento do botão
turboButton.MouseButton1Click:Connect(function()
    turboAtivo = not turboAtivo
    turboButton.Text = turboAtivo and "🚀 Turbo ON" or "🚀 Turbo OFF"
    aplicarTurbo(turboAtivo)
end)

-- Se sair do carro, desativa o turbo automaticamente
game:GetService("RunService").Heartbeat:Connect(function()
    if turboAtivo and not isInCar() then
        turboAtivo = false
        turboButton.Text = "🚀 Turbo OFF"
        aplicarTurbo(false)
    end
end)
