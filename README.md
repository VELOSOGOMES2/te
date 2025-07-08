local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "TurboRSJ"

-- 🖼️ UI
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 260, 0, 80)
frame.Position = UDim2.new(0, 20, 0.5, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Text = "🔥 Turbo Veloz RSJGAMES"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14

local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(1, -20, 0, 40)
btn.Position = UDim2.new(0, 10, 0, 35)
btn.Text = "Ativar Turbo"
btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
btn.TextColor3 = Color3.new(1, 1, 1)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 16

-- 🔍 Função para detectar o carro
local function getCar()
	local char = player.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if hum and hum.SeatPart then
		return hum.SeatPart:FindFirstAncestorOfClass("Model")
	end
end

-- 🚀 Turbo via AssemblyVelocity
local turbo = false
local heartbeatConn

btn.MouseButton1Click:Connect(function()
	local car = getCar()
	if not car or not car.PrimaryPart then
		btn.Text = "❌ Sem carro!"
		wait(2)
		btn.Text = "Ativar Turbo"
		return
	end

	local seat = car:FindFirstChildWhichIsA("VehicleSeat", true)
	if not seat then
		btn.Text = "❌ Sem VehicleSeat"
		wait(2)
		btn.Text = "Ativar Turbo"
		return
	end

	if not turbo then
		turbo = true
		btn.Text = "✅ Turbo Ativo"

		-- ⚙️ Reduz atrito e aumenta resposta
		for _, part in pairs(car:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CustomPhysicalProperties = PhysicalProperties.new(0.1, 0, 0.1)
			end
		end

		-- 🚀 Aplica velocidade toda frame
		heartbeatConn = runService.Heartbeat:Connect(function()
			if not turbo then return end
			local cf = car.PrimaryPart.CFrame
			local direction = cf.LookVector
			car:ApplyImpulse(direction * 50000) -- força suave contínua
			car.PrimaryPart.AssemblyLinearVelocity = direction * 450 -- ultrapassa 361 controlado
		end)

	else
		turbo = false
		btn.Text = "Turbo Desativado"
		if heartbeatConn then heartbeatConn:Disconnect() end

		-- 🔧 Reset propriedades físicas
		for _, part in pairs(getCar():GetDescendants()) do
			if part:IsA("BasePart") then
				part.CustomPhysicalProperties = PhysicalProperties.new()
			end
		end
	end
end)
