-- ⚙️ Interface
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "VelocidadeTurboMenu"

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 260, 0, 80)
frame.Position = UDim2.new(0, 20, 0.5, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Text = "🛞 Boost de Velocidade Suave"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 14

local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(1, -20, 0, 40)
btn.Position = UDim2.new(0, 10, 0, 35)
btn.Text = "Ativar Boost"
btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
btn.TextColor3 = Color3.new(1, 1, 1)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 16

-- 🎮 Função para encontrar o carro
local player = game.Players.LocalPlayer
local function getCar()
	local char = player.Character
	if not char then return nil end
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	if humanoid and humanoid.SeatPart then
		return humanoid.SeatPart:FindFirstAncestorOfClass("Model")
	end
end

-- 🚀 Ativar boost suave
local boostAtivo = false
btn.MouseButton1Click:Connect(function()
	local car = getCar()
	if not car then
		btn.Text = "❌ Entra num carro!"
		wait(2)
		btn.Text = "Ativar Boost"
		return
	end

	local seat = car:FindFirstChildWhichIsA("VehicleSeat", true)
	if not seat then
		btn.Text = "❌ Sem VehicleSeat"
		wait(2)
		btn.Text = "Ativar Boost"
		return
	end

	if not boostAtivo then
		boostAtivo = true
		btn.Text = "✅ Boost Ativo"

		-- 🚀 Aumentar velocidade máxima e torque com força mais alta
		pcall(function()
			if seat:FindFirstChild("MaxSpeed") then
				seat.MaxSpeed = 1200 -- Aumentado
			end
			seat.MaxSpeed = 1200
			seat.Torque = 250000 -- FORÇA MAIS ALTA
			seat.TorqueBoost = 100000 -- Se existir esse valor, aplica também
		end)

	else
		boostAtivo = false
		btn.Text = "Boost Desativado"

		-- 🔧 Volta aos valores padrão
		pcall(function()
			seat.MaxSpeed = 361
			seat.Torque = 3000
			if seat:FindFirstChild("TorqueBoost") then
				seat.TorqueBoost = 0
			end
		end)
	end
end)
