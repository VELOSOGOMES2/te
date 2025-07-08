-- 🚀 Boost de velocidade melhorado
boostButton.MouseButton1Click:Connect(function()
    boostRunning = not boostRunning
    boostButton.Text = boostRunning and "🚀 Boost de Velocidade ON" or "🚀 Boost de Velocidade OFF"

    if boostRunning then
        notify("⚡ Boost ativado")
        boostConnection = RunService.Heartbeat:Connect(function()
            if not isInCar() then
                stopBoost()
                return
            end
            local car = getCar()
            if car and car.PrimaryPart then
                local velocity = car.PrimaryPart.AssemblyLinearVelocity
                local direction = car.PrimaryPart.CFrame.LookVector

                -- Impulso gradual com limite
                local boostForce = direction * 3
                local newVelocity = velocity + boostForce

                -- Limite de velocidade máxima (opcional: aumenta se quiser mais)
                local maxSpeed = 200
                if newVelocity.Magnitude < maxSpeed then
                    car.PrimaryPart.AssemblyLinearVelocity = newVelocity
                end
            end
        end)
    else
        stopBoost()
    end
end)
