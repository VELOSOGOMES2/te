-- Serviços
local RunService = game:GetService("RunService")
local player = game.Players.LocalPlayer

-- Detecta o carro atual
local function getCar()
	local char = player.Character or player.CharacterAdded:Wait()
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if humanoid and humanoid.SeatPart then
		local seat = humanoid.SeatPart
		local car = seat:FindFirstAncestorOfClass("Model")
		if car and car.PrimaryPart then
			return car
		end
	end
	return nil
end

-- Verifica se o jogador está no carro
local function isInCar()
	local char = player.Character
	local humanoid = char and char:FindFirstChildOfClass("Humanoid")
	return humanoid and humanoid.SeatPart ~= nil
end

-- UI
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "BoostVelocidadeUI"

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 260, 0, 80)
mainFrame.Position = UDim2.new(0, 20, 0.6, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Text = "🚀 Boost de Velocidade"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14

-- Arrastar menu
local dragging = false
local dragStart, startPos
title.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
	end
end)
title.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(
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

-- Botão
local boostButton = Instance.new("TextButton", mainFrame)
boostButton.Size = UDim2.new(1, -20, 0, 40)
boostButton.Position = UDim2.new(0, 10, 0, 35)
boostButton.Text = "🚀 Boost de Velocidade OFF"
boostButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
boostButton.TextColor3 = Color3.new(1, 1, 1)
boostButton.Font = Enum.Font.GothamBold
boostButton.TextSize = 16

-- Lógica
local boostAtivo = false
local loop = nil

local function ativarBoost()
	if loop then loop:Disconnect() end
	boostAtivo = true
	boostButton.Text = "🚀 Boost de Velocidade ON"

	loop = RunService.Heartbeat:Connect(function()
		if not isInCar() then
			boostButton.Text = "🚀 Boost de Velocidade OFF"
			boostAtivo = false
			loop:Disconnect()
			return
		end

		local car = getCar()
		if car and car.PrimaryPart then
			local dir = car.PrimaryPart.CFrame.LookVector
			local velocidadeAlvo = 1 -- força do impulso (ajusta se quiser)

			-- Aplica impulso muito forte na direção que o carro está virado
			car.PrimaryPart.AssemblyLinearVelocity = dir * velocidadeAlvo
		end
	end)
end

local function desativarBoost()
	boostAtivo = false
	boostButton.Text = "🚀 Boost de Velocidade OFF"
	if loop then loop:Disconnect() end
end

-- Clique do botão
boostButton.MouseButton1Click:Connect(function()
	if boostAtivo then
		desativarBoost()
	else
		ativarBoost()
	end
end)
