-- ============================================
-- SCRIPT UNIFICADO OTIMIZADO - VERIFICAÇÃO DE ADMINS + PUXADA + ESP + OTIMIZAÇÃO
-- ============================================

local player = game.Players.LocalPlayer
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local camera = workspace.CurrentCamera
local TextChatService = game:GetService("TextChatService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

-- ============================================
-- CONFIGURAÇÃO GLOBAL
-- ============================================
_G.AimbotEnabled = false
_G.EspEnabled = false
_G.NoclipEnabled = false
_G.FlyEnabled = false
_G.FlyLeveledEnabled = false
_G.FlyFrozen = false
_G.Smoothness = 0.15
_G.AimbotFOV = 120
_G.ChatMaxMessages = 15
_G.FlySpeed = 100
_G.EspDistance = 500
_G.PullEnabled = false
_G.OptimizationLevel = 0 -- 0 = Normal, 1 = Médio, 2 = Máximo

_G.SelectedQuickOption = "FlyEnabled"

-- CONFIG PUXADA
local DISTANCIA_MAXIMA = 500 
local OFFSET_POSICAO = CFrame.new(1.5, 0, -5)
local COR_ESP = Color3.fromRGB(255, 0, 0)

-- ============================================
-- SISTEMA DE OTIMIZAÇÃO
-- ============================================

local function otimizarJogo(nivel)
	print("🔧 Otimizando jogo - Nível: " .. nivel)
	
	if nivel == 0 then
		-- NORMAL - Sem mudanças
		print("✅ Modo Normal ativado")
		
	elseif nivel == 1 then
		-- OTIMIZAÇÃO MÉDIA
		print("⚡ Otimização Média ativada")
		
		-- Reduz qualidade das texturas
		settings().Rendering.QualityLevel = Enum.QualityLevel.Level03
		
		-- Desativa sombras
		game.Lighting.GlobalShadows = false
		
		-- Reduz distância de renderização
		workspace.Terrain.MaximumDistance = 256
		
		-- Desativa partículas desnecessárias
		for _, obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("ParticleEmitter") then
				obj.Enabled = false
			end
		end
		
		print("✅ Texturas reduzidas, sombras desativadas")
		
	elseif nivel == 2 then
		-- OTIMIZAÇÃO MÁXIMA
		print("🚀 Otimização Máxima ativada")
		
		-- Qualidade mínima
		settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
		
		-- Desativa sombras completamente
		game.Lighting.GlobalShadows = false
		game.Lighting.Brightness = 2
		
		-- Reduz distância de renderização drasticamente
		workspace.Terrain.MaximumDistance = 128
		
		-- Desativa todas as partículas
		for _, obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("ParticleEmitter") then
				obj.Enabled = false
			end
			if obj:IsA("PointLight") or obj:IsA("SurfaceGui") then
				obj.Enabled = false
			end
		end
		
		-- Remove texturas de terreno complexas
		for _, part in pairs(workspace:FindPartBoundsInRadius(player.Character.HumanoidRootPart.Position, 200)) do
			if part:IsA("BasePart") then
				part.Material = Enum.Material.Plastic
				part.CanCollide = true
			end
		end
		
		print("✅ Qualidade mínima, máximo desempenho")
	end
	
	_G.OptimizationLevel = nivel
end

-- ============================================
-- LISTA DE ADMINS
-- ============================================
local ADMIN_LIST = {
	"Owner1",
	"Dev1",
	"Staff1",
	"Mod1",
	"Admin1"
}

local function isAdmin(playerName)
	if not playerName then return false end
	local lowerPlayerName = playerName:lower()
	for _, adminName in pairs(ADMIN_LIST) do
		if lowerPlayerName == adminName:lower() then
			return true
		end
	end
	return false
end

local function getAdminsOnline()
	local adminsOnline = {}
	for _, player_ in pairs(Players:GetPlayers()) do
		if isAdmin(player_.Name) then
			table.insert(adminsOnline, player_.Name)
		end
	end
	return adminsOnline
end

local function displayAdminsOnline()
	local admins = getAdminsOnline()
	local count = #admins
	print("═══════════════════════════════════")
	print("👨‍💼 VERIFICAÇÃO DE ADMINS NO SERVIDOR")
	print("═══════════════════════════════════")
	print("Total de jogadores: " .. #Players:GetPlayers())
	print("Admins encontrados: " .. count)
	if count > 0 then
		print("\n✅ ADMINS ONLINE:")
		for i, adminName in pairs(admins) do
			print("  [" .. i .. "] " .. adminName .. " 🟢")
		end
	else
		print("\n❌ Nenhum admin online no servidor")
	end
	print("═══════════════════════════════════")
	return count, admins
end

local function displayAdminList()
	print("\n📋 LISTA DE ADMINS CADASTRADOS:")
	for i, adminName in pairs(ADMIN_LIST) do
		print("  [" .. i .. "] " .. adminName)
	end
	print("")
end

print("[ADMIN CHECK] Seu rank: User")
displayAdminList()
displayAdminsOnline()

-- ============================================
-- PAINEL RÁPIDO
-- ============================================
local guiQuick = Instance.new("ScreenGui", player.PlayerGui)
guiQuick.ResetOnSpawn = false
guiQuick.Name = "QuickAccessPanel"

local quickBtn = Instance.new("TextButton", guiQuick)
quickBtn.Size = UDim2.new(0, 60, 0, 60)
quickBtn.Position = UDim2.new(0, 10, 0, 20)
quickBtn.Text = ""
quickBtn.BackgroundTransparency = 1
quickBtn.BorderSizePixel = 0

quickBtn.MouseButton1Click:Connect(function()
	_G[_G.SelectedQuickOption] = not _G[_G.SelectedQuickOption]
	print("✅ " .. _G.SelectedQuickOption .. " " .. (_G[_G.SelectedQuickOption] and "ATIVADO" or "DESATIVADO"))
end)

-- ============================================
-- MENU PRINCIPAL
-- ============================================

local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.ResetOnSpawn = false
gui.Name = "ExploitMenu"

local openBtn = Instance.new("TextButton", gui)
openBtn.Size = UDim2.new(0, 60, 0, 60)
openBtn.Position = UDim2.new(1, -70, 0, 20)
openBtn.Text = ""
openBtn.BackgroundTransparency = 1
openBtn.BorderSizePixel = 0

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 400)
frame.Position = UDim2.new(0.5, -125, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
frame.Visible = false
Instance.new("UICorner", frame)

openBtn.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

local nav = Instance.new("Frame", frame)
nav.Size = UDim2.new(1, 0, 0, 30)
nav.BackgroundTransparency = 1
Instance.new("UIListLayout", nav).FillDirection = "Horizontal"

local pages = Instance.new("Frame", frame)
pages.Size = UDim2.new(1, 0, 1, -35)
pages.Position = UDim2.new(0, 0, 0, 35)
pages.BackgroundTransparency = 1

local function createPage(n)
	local p = Instance.new("ScrollingFrame", pages)
	p.Name = n
	p.Size = UDim2.new(1, 0, 1, 0)
	p.BackgroundTransparency = 1
	p.Visible = false
	p.ScrollBarThickness = 6
	Instance.new("UIListLayout", p).Padding = UDim.new(0, 5)
	p.UIListLayout.HorizontalAlignment = "Center"
	return p
end

local pgLuta = createPage("Luta")
local pgExtra = createPage("Extra")
local pgChat = createPage("Chat")
local pgAdmins = createPage("Admins")
local pgConfig = createPage("Config")
pgLuta.Visible = true

local function tabBtn(t, p)
	local b = Instance.new("TextButton", nav)
	b.Size = UDim2.new(0.166, 0, 1, 0)
	b.Text = t
	b.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
	b.TextColor3 = Color3.new(1, 1, 1)
	b.BorderSizePixel = 0
	b.MouseButton1Click:Connect(function()
		pgLuta.Visible = false
		pgExtra.Visible = false
		pgChat.Visible = false
		pgAdmins.Visible = false
		pgConfig.Visible = false
		p.Visible = true
	end)
end

tabBtn("⚔️", pgLuta)
tabBtn("➕", pgExtra)
tabBtn("💬", pgChat)
tabBtn("👨‍💼", pgAdmins)
tabBtn("⚙️", pgConfig)

local function createToggle(text, parent, var, color)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(0.9, 0, 0, 35)
	b.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
	b.Text = text .. ": OFF"
	b.TextColor3 = Color3.new(1, 1, 1)
	Instance.new("UICorner", b)

	b.MouseButton1Click:Connect(function()
		_G[var] = not _G[var]
		b.Text = text .. (_G[var] and ": ON" or ": OFF")
		b.BackgroundColor3 = _G[var] and color or Color3.fromRGB(40, 40, 45)
		_G.SelectedQuickOption = var
		print("✅ Opção selecionada: " .. text)
	end)
end

createToggle("Smooth Aimbot", pgLuta, "AimbotEnabled", Color3.fromRGB(200, 50, 50))
createToggle("ESP BOX + Nick", pgLuta, "EspEnabled", Color3.fromRGB(200, 50, 50))
createToggle("Puxada (Pull)", pgLuta, "PullEnabled", Color3.fromRGB(100, 200, 255))
createToggle("Noclip", pgExtra, "NoclipEnabled", Color3.fromRGB(100, 100, 255))
createToggle("Fly Normal", pgExtra, "FlyEnabled", Color3.fromRGB(0, 255, 200))
createToggle("Fly Nivelado", pgExtra, "FlyLeveledEnabled", Color3.fromRGB(255, 165, 0))

-- ============================================
-- SLIDERS COM INPUT TEXT
-- ============================================

local function createInputSlider(text, parent, var, minVal, maxVal)
	local container = Instance.new("Frame", parent)
	container.Size = UDim2.new(0.9, 0, 0, 50)
	container.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
	container.BorderSizePixel = 0
	Instance.new("UICorner", container)

	local label = Instance.new("TextLabel", container)
	label.Size = UDim2.new(0.5, 0, 0, 20)
	label.Position = UDim2.new(0, 5, 0, 5)
	label.BackgroundTransparency = 1
	label.Font = Enum.Font.SourceSans
	label.TextSize = 13
	label.TextColor3 = Color3.new(1, 1, 1)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Text = text .. ":"

	local inputBox = Instance.new("TextBox", container)
	inputBox.Size = UDim2.new(0.4, 0, 0, 25)
	inputBox.Position = UDim2.new(0.55, 0, 0, 5)
	inputBox.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
	inputBox.TextColor3 = Color3.new(1, 1, 1)
	inputBox.Font = Enum.Font.SourceSans
	inputBox.TextSize = 14
	inputBox.Text = tostring(_G[var])
	inputBox.PlaceholderText = "Digite o valor"
	Instance.new("UICorner", inputBox)

	local btnAplicar = Instance.new("TextButton", container)
	btnAplicar.Size = UDim2.new(0.35, 0, 0, 25)
	btnAplicar.Position = UDim2.new(0.57, 0, 0, 23)
	btnAplicar.Text = "✓"
	btnAplicar.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
	btnAplicar.TextColor3 = Color3.new(1, 1, 1)
	btnAplicar.Font = Enum.Font.GothamBold
	btnAplicar.TextSize = 16
	btnAplicar.BorderSizePixel = 0
	Instance.new("UICorner", btnAplicar)

	btnAplicar.MouseButton1Click:Connect(function()
		local value = tonumber(inputBox.Text)
		if value then
			value = math.clamp(value, minVal, maxVal)
			_G[var] = value
			inputBox.Text = tostring(value)
			print("✅ " .. text .. " alterado para: " .. value)
		else
			print("❌ Valor inválido! Use um número entre " .. minVal .. " e " .. maxVal)
			inputBox.Text = tostring(_G[var])
		end
	end)

	inputBox.FocusLost:Connect(function(enterPressed)
		if enterPressed then
			btnAplicar:TriggerEvent("MouseButton1Click")
		end
	end)
end

createInputSlider("Aimbot Force", pgLuta, "Smoothness", 0.01, 0.5)
createInputSlider("Aimbot FOV", pgLuta, "AimbotFOV", 50, 300)
createInputSlider("Fly Speed", pgExtra, "FlySpeed", 10, 500)

local rejoinBtn = Instance.new("TextButton", pgExtra)
rejoinBtn.Size = UDim2.new(0.9, 0, 0, 35)
rejoinBtn.Text = "REJOIN SERVER"
rejoinBtn.BackgroundColor3 = Color3.fromRGB(70, 120, 255)
rejoinBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", rejoinBtn)

rejoinBtn.MouseButton1Click:Connect(function()
	TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, player)
end)

local freezeFlyBtn = Instance.new("TextButton", pgExtra)
freezeFlyBtn.Size = UDim2.new(0.9, 0, 0, 35)
freezeFlyBtn.Text = "❄️ CONGELAR FLY: OFF"
freezeFlyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
freezeFlyBtn.TextColor3 = Color3.new(1, 1, 1)
freezeFlyBtn.Font = Enum.Font.GothamBold
freezeFlyBtn.TextSize = 13
Instance.new("UICorner", freezeFlyBtn)

freezeFlyBtn.MouseButton1Click:Connect(function()
	_G.FlyFrozen = not _G.FlyFrozen
	freezeFlyBtn.Text = "❄️ CONGELAR FLY: " .. (_G.FlyFrozen and "ON" or "OFF")
	freezeFlyBtn.BackgroundColor3 = _G.FlyFrozen and Color3.fromRGB(0, 150, 255) or Color3.fromRGB(40, 40, 45)
	print("❄️ Fly congelado: " .. (_G.FlyFrozen and "ATIVADO" or "DESATIVADO"))
end)

-- ============================================
-- OTIMIZAÇÃO NO MENU
-- ============================================

local optimizationTitle = Instance.new("TextLabel", pgExtra)
optimizationTitle.Size = UDim2.new(0.9, 0, 0, 25)
optimizationTitle.BackgroundColor3 = Color3.fromRGB(50, 100, 150)
optimizationTitle.TextColor3 = Color3.new(1, 1, 1)
optimizationTitle.Font = Enum.Font.GothamBold
optimizationTitle.TextSize = 12
optimizationTitle.Text = "🔧 OTIMIZAÇÃO DE DESEMPENHO"
Instance.new("UICorner", optimizationTitle)

local optNormalBtn = Instance.new("TextButton", pgExtra)
optNormalBtn.Size = UDim2.new(0.9, 0, 0, 30)
optNormalBtn.Text = "🟢 NORMAL"
optNormalBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
optNormalBtn.TextColor3 = Color3.new(1, 1, 1)
optNormalBtn.Font = Enum.Font.GothamBold
optNormalBtn.TextSize = 12
Instance.new("UICorner", optNormalBtn)

optNormalBtn.MouseButton1Click:Connect(function()
	otimizarJogo(0)
end)

local optMediumBtn = Instance.new("TextButton", pgExtra)
optMediumBtn.Size = UDim2.new(0.9, 0, 0, 30)
optMediumBtn.Text = "⚡ OTIMIZAÇÃO MÉDIA"
optMediumBtn.BackgroundColor3 = Color3.fromRGB(200, 150, 50)
optMediumBtn.TextColor3 = Color3.new(1, 1, 1)
optMediumBtn.Font = Enum.Font.GothamBold
optMediumBtn.TextSize = 12
Instance.new("UICorner", optMediumBtn)

optMediumBtn.MouseButton1Click:Connect(function()
	otimizarJogo(1)
end)

local optMaxBtn = Instance.new("TextButton", pgExtra)
optMaxBtn.Size = UDim2.new(0.9, 0, 0, 30)
optMaxBtn.Text = "🚀 OTIMIZAÇÃO MÁXIMA"
optMaxBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
optMaxBtn.TextColor3 = Color3.new(1, 1, 1)
optMaxBtn.Font = Enum.Font.GothamBold
optMaxBtn.TextSize = 12
Instance.new("UICorner", optMaxBtn)

optMaxBtn.MouseButton1Click:Connect(function()
	otimizarJogo(2)
end)

-- ============================================
-- CONFIGURAÇÕES
-- ============================================

local configTitle = Instance.new("TextLabel", pgConfig)
configTitle.Size = UDim2.new(0.9, 0, 0, 30)
configTitle.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
configTitle.TextColor3 = Color3.new(1, 1, 1)
configTitle.TextScaled = true
configTitle.Font = Enum.Font.GothamBold
configTitle.Text = "CONFIGURAÇÕES GERAIS"
Instance.new("UICorner", configTitle)

createInputSlider("Aimbot Force", pgConfig, "Smoothness", 0.01, 0.5)
createInputSlider("Aimbot FOV", pgConfig, "AimbotFOV", 50, 300)
createInputSlider("Fly Speed", pgConfig, "FlySpeed", 10, 500)
createInputSlider("ESP Distance", pgConfig, "EspDistance", 50, 2000)

local adminListLabel = Instance.new("TextLabel", pgConfig)
adminListLabel.Size = UDim2.new(0.9, 0, 0, 20)
adminListLabel.BackgroundTransparency = 1
adminListLabel.TextColor3 = Color3.new(1, 1, 1)
adminListLabel.TextSize = 12
adminListLabel.TextXAlignment = Enum.TextXAlignment.Left
adminListLabel.Font = Enum.Font.SourceSans
adminListLabel.Text = "Lista de Admins (separados por vírgula)"

local adminListInput = Instance.new("TextBox", pgConfig)
adminListInput.Size = UDim2.new(0.9, 0, 0, 80)
adminListInput.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
adminListInput.TextColor3 = Color3.new(1, 1, 1)
adminListInput.Font = Enum.Font.SourceSans
adminListInput.TextSize = 11
adminListInput.TextXAlignment = Enum.TextXAlignment.Left
adminListInput.TextYAlignment = Enum.TextYAlignment.Top
adminListInput.MultiLine = true
adminListInput.Text = table.concat(ADMIN_LIST, ", ")
adminListInput.PlaceholderText = "Owner1, Dev1, Staff1, Mod1, Admin1"
Instance.new("UICorner", adminListInput)

local saveAdminBtn = Instance.new("TextButton", pgConfig)
saveAdminBtn.Size = UDim2.new(0.9, 0, 0, 35)
saveAdminBtn.BackgroundColor3 = Color3.fromRGB(50, 150, 100)
saveAdminBtn.TextColor3 = Color3.new(1, 1, 1)
saveAdminBtn.Text = "SALVAR ADMINS"
saveAdminBtn.Font = Enum.Font.GothamBold
saveAdminBtn.TextSize = 14
Instance.new("UICorner", saveAdminBtn)

local reloadAdminBtn = Instance.new("TextButton", pgConfig)
reloadAdminBtn.Size = UDim2.new(0.9, 0, 0, 35)
reloadAdminBtn.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
reloadAdminBtn.TextColor3 = Color3.new(1, 1, 1)
reloadAdminBtn.Text = "🔄 VERIFICAR ADMINS"
reloadAdminBtn.Font = Enum.Font.GothamBold
reloadAdminBtn.TextSize = 14
Instance.new("UICorner", reloadAdminBtn)

local resetBtn = Instance.new("TextButton", pgConfig)
resetBtn.Size = UDim2.new(0.9, 0, 0, 35)
resetBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
resetBtn.TextColor3 = Color3.new(1, 1, 1)
resetBtn.Text = "RESETAR CONFIGURAÇÕES"
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 14
Instance.new("UICorner", resetBtn)

resetBtn.MouseButton1Click:Connect(function()
	_G.Smoothness = 0.15
	_G.AimbotFOV = 120
	_G.FlySpeed = 100
	_G.EspDistance = 500
	_G.ChatMaxMessages = 15
	print("✅ Configurações resetadas!")
end)

local copyBtn = Instance.new("TextButton", pgConfig)
copyBtn.Size = UDim2.new(0.9, 0, 0, 35)
copyBtn.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
copyBtn.TextColor3 = Color3.new(1, 1, 1)
copyBtn.Text = "COPIAR CONFIGURAÇÃO"
copyBtn.Font = Enum.Font.GothamBold
copyBtn.TextSize = 14
Instance.new("UICorner", copyBtn)

copyBtn.MouseButton1Click:Connect(function()
	local config = string.format([[
Smoothness: %.2f
AimbotFOV: %.0f
FlySpeed: %.0f
EspDistance: %.0f
Otimização: %s
]], _G.Smoothness, _G.AimbotFOV, _G.FlySpeed, _G.EspDistance, _G.OptimizationLevel == 0 and "Normal" or (_G.OptimizationLevel == 1 and "Média" or "Máxima"))
	print("📋 Configuração copiada para console:")
	print(config)
end)

-- ============================================
-- CHAT
-- ============================================

local scroll = Instance.new("ScrollingFrame", pgChat)
scroll.Size = UDim2.new(0.95, 0, 0.85, 0)
scroll.Position = UDim2.new(0.025, 0, 0, 0)
scroll.BackgroundTransparency = 1
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.ScrollBarThickness = 6

local lay = Instance.new("UIListLayout", scroll)
lay.Padding = UDim.new(0, 2)

local messages = {}

local function msg(t)
	table.insert(messages, t)
	
	if #messages > _G.ChatMaxMessages then
		table.remove(messages, 1)
		local firstChild = scroll:GetChildren()[1]
		if firstChild and firstChild:IsA("TextLabel") then
			firstChild:Destroy()
		end
	end
	
	local l = Instance.new("TextLabel", scroll)
	l.Size = UDim2.new(1, 0, 0, 16)
	l.BackgroundTransparency = 1
	l.Font = Enum.Font.SourceSans
	l.TextSize = 15
	l.TextColor3 = Color3.new(1, 1, 1)
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.Text = t
	scroll.CanvasSize = UDim2.new(0, 0, 0, lay.AbsoluteContentSize.Y + 10)
	scroll.CanvasPosition = Vector2.new(0, 9999)
end

local clearBtn = Instance.new("TextButton", pgChat)
clearBtn.Size = UDim2.new(0.9, 0, 0.12, 0)
clearBtn.Position = UDim2.new(0.05, 0, 0.85, 0)
clearBtn.Text = "LIMPAR CHAT"
clearBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
clearBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", clearBtn)

clearBtn.MouseButton1Click:Connect(function()
	for _, v in pairs(scroll:GetChildren()) do
		if v:IsA("TextLabel") then v:Destroy() end
	end
	messages = {}
	scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
end)

TextChatService.OnIncomingMessage = function(m)
	local s = m.TextSource
	if s then
		local pl = game.Players:GetPlayerByUserId(s.UserId)
		if pl then msg(pl.Name .. ": " .. m.Text) end
	end
end

-- ============================================
-- PÁGINA ADMINS
-- ============================================

local function updateAdminsPage()
	for _, v in pairs(pgAdmins:GetChildren()) do
		if v:IsA("Frame") and v.Name ~= "UIListLayout" then
			v:Destroy()
		end
	end
	
	local admins = getAdminsOnline()
	local count = #admins
	
	local header = Instance.new("TextLabel", pgAdmins)
	header.Size = UDim2.new(0.9, 0, 0, 40)
	header.BackgroundColor3 = count > 0 and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(50, 150, 50)
	header.TextColor3 = Color3.new(1, 1, 1)
	header.TextScaled = true
	header.Font = Enum.Font.GothamBold
	header.Text = count > 0 and "🚨 ADMINS ONLINE: " .. count or "✅ NENHUM ADMIN ONLINE"
	Instance.new("UICorner", header)
	
	if count == 0 then
		local noAdmins = Instance.new("TextLabel", pgAdmins)
		noAdmins.Size = UDim2.new(0.9, 0, 0.3, 0)
		noAdmins.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
		noAdmins.TextColor3 = Color3.new(1, 1, 1)
		noAdmins.TextScaled = true
		noAdmins.Text = "Servidor seguro! 🟢"
		Instance.new("UICorner", noAdmins)
	else
		for i, adminName in pairs(admins) do
			local adminFrame = Instance.new("Frame", pgAdmins)
			adminFrame.Size = UDim2.new(0.9, 0, 0, 45)
			adminFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
			Instance.new("UICorner", adminFrame)
			
			local borderStroke = Instance.new("UIStroke", adminFrame)
			borderStroke.Color = Color3.fromRGB(255, 100, 100)
			borderStroke.Thickness = 2
			
			local nameLabel = Instance.new("TextLabel", adminFrame)
			nameLabel.Size = UDim2.new(1, 0, 1, 0)
			nameLabel.BackgroundTransparency = 1
			nameLabel.TextColor3 = Color3.new(1, 1, 1)
			nameLabel.TextXAlignment = Enum.TextXAlignment.Left
			nameLabel.Font = Enum.Font.GothamBold
			nameLabel.TextSize = 14
			nameLabel.Text = "  🚨 " .. i .. ". " .. adminName
		end
	end
end

saveAdminBtn.MouseButton1Click:Connect(function()
	ADMIN_LIST = {}
	local adminText = adminListInput.Text
	for adminName in adminText:gmatch("[^,]+") do
		local trimmed = adminName:match("^%s*(.-)%s*$")
		if trimmed ~= "" then
			table.insert(ADMIN_LIST, trimmed)
		end
	end
	print("✅ Lista de admins atualizada!")
	displayAdminList()
	displayAdminsOnline()
	updateAdminsPage()
end)

reloadAdminBtn.MouseButton1Click:Connect(function()
	print("\n🔄 Recarregando verificação de admins...")
	displayAdminList()
	displayAdminsOnline()
	updateAdminsPage()
	print("✅ Verificação concluída!\n")
end)

Players.PlayerAdded:Connect(function(plr)
	if isAdmin(plr.Name) then
		print("🚨 ATENÇÃO! Admin entrou: " .. plr.Name)
		displayAdminsOnline()
		updateAdminsPage()
	end
end)

Players.PlayerRemoving:Connect(function(plr)
	if isAdmin(plr.Name) then
		print("✅ Admin saiu: " .. plr.Name)
		displayAdminsOnline()
		updateAdminsPage()
	end
end)

updateAdminsPage()
task.wait(0.5)
updateAdminsPage()

-- ============================================
-- AIMBOT
-- ============================================

RunService.RenderStepped:Connect(function()
	if _G.AimbotEnabled then
		local target = nil
		local shortest = _G.AimbotFOV
		for _, v in pairs(game.Players:GetPlayers()) do
			if v ~= player and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
				local pos, vis = camera:WorldToViewportPoint(v.Character.Head.Position)
				if vis then
					local dist = (Vector2.new(pos.X, pos.Y) - camera.ViewportSize / 2).Magnitude
					if dist < shortest then
						shortest = dist
						target = v
					end
				end
			end
		end
		if target then
			local cf = CFrame.lookAt(camera.CFrame.Position, target.Character.Head.Position)
			camera.CFrame = camera.CFrame:Lerp(cf, _G.Smoothness)
		end
	end
end)

-- ============================================
-- ESP BOX
-- ============================================

local function esp(p)
	local box = Drawing.new("Square")
	box.Thickness = 1
	box.Filled = false
	box.Color = Color3.fromRGB(0, 255, 255)

	local name = Drawing.new("Text")
	name.Size = 13
	name.Center = true
	name.Outline = true

	RunService.RenderStepped:Connect(function()
		if _G.EspEnabled and p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local root = p.Character.HumanoidRootPart
			local head = p.Character:FindFirstChild("Head")

			if head then
				local r, vis1 = camera:WorldToViewportPoint(root.Position)
				local h, vis2 = camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))

				if vis1 and vis2 then
					local height = math.abs(h.Y - r.Y) * 2
					local width = height / 1.6

					box.Size = Vector2.new(width, height)
					box.Position = Vector2.new(r.X - width / 2, r.Y - height / 2)
					box.Visible = true

					name.Position = Vector2.new(r.X, r.Y - height / 2 - 14)
					name.Text = p.Name
					name.Visible = true
				else
					box.Visible = false
					name.Visible = false
				end
			end
		else
			box.Visible = false
			name.Visible = false
		end
	end)
end

for _, p in pairs(game.Players:GetPlayers()) do
	esp(p)
end
game.Players.PlayerAdded:Connect(esp)

-- ============================================
-- NOCLIP
-- ============================================

RunService.Stepped:Connect(function()
	if _G.NoclipEnabled and player.Character then
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
			end
		end
	elseif player.Character then
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = true
			end
		end
	end
end)

-- ============================================
-- FLY (NORMAL + NIVELADO)
-- ============================================

local frozenCFrame = nil

RunService.RenderStepped:Connect(function(dt)
	if (_G.FlyEnabled or _G.FlyLeveledEnabled) and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		hrp.Anchored = true

		if _G.FlyFrozen then
			if not frozenCFrame then
				frozenCFrame = hrp.CFrame
			end
			hrp.CFrame = frozenCFrame
		else
			frozenCFrame = nil
			
			if _G.FlyLeveledEnabled then
				local direction = camera.CFrame.LookVector
				direction = Vector3.new(direction.X, 0, direction.Z).Unit
				hrp.CFrame = hrp.CFrame + direction * _G.FlySpeed * dt
			elseif _G.FlyEnabled then
				hrp.CFrame = hrp.CFrame + camera.CFrame.LookVector * _G.FlySpeed * dt
			end
		end
	elseif not _G.FlyEnabled and not _G.FlyLeveledEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		if hrp.Anchored then
			hrp.Anchored = false
		end
		frozenCFrame = nil
	end
end)

-- ============================================
-- PUXADA + ESP ESQUELETO
-- ============================================

local conexaoLoop = nil
local conexaoESP = nil
local espObjects = {}

local function podePuxarOuMostrarESP(outroChar)
	local humanoid = outroChar:FindFirstChildOfClass("Humanoid")
	
	if humanoid then
		if humanoid.Health <= 0 or humanoid:GetState() == Enum.HumanoidStateType.Dead then
			return false
		end
		
		if humanoid.Sit or humanoid:GetState() == Enum.HumanoidStateType.Seated then
			return false
		end
		
		return true
	end
	return false
end

local function criarEsqueleto(char)
    local skeleton = {}
    local parts = {
        Head = char:FindFirstChild("Head"),
        Torso = char:FindFirstChild("Torso"),
        UpperTorso = char:FindFirstChild("UpperTorso"),
        LowerTorso = char:FindFirstChild("LowerTorso"),
        RightUpperArm = char:FindFirstChild("RightUpperArm"),
        RightLowerArm = char:FindFirstChild("RightLowerArm"),
        RightHand = char:FindFirstChild("RightHand"),
        LeftUpperArm = char:FindFirstChild("LeftUpperArm"),
        LeftLowerArm = char:FindFirstChild("LeftLowerArm"),
        LeftHand = char:FindFirstChild("LeftHand"),
        RightUpperLeg = char:FindFirstChild("RightUpperLeg"),
        RightLowerLeg = char:FindFirstChild("RightLowerLeg"),
        RightFoot = char:FindFirstChild("RightFoot"),
        LeftUpperLeg = char:FindFirstChild("LeftUpperLeg"),
        LeftLowerLeg = char:FindFirstChild("LeftLowerLeg"),
        LeftFoot = char:FindFirstChild("LeftFoot")
    }

    local connections = {
        {"Head", "UpperTorso"},
        {"UpperTorso", "LowerTorso"},
        {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
        {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
        {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"},
        {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"}
    }

    for _, conn in ipairs(connections) do
        local part1 = parts[conn[1]]
        local part2 = parts[conn[2]]

        if part1 and part2 then
            local line = Drawing.new("Line")
            line.Color = COR_ESP
            line.Thickness = 2
            line.Visible = false
            table.insert(skeleton, line)
        end
    end
    return skeleton, parts, connections
end

local function atualizarEsqueleto()
    for _, playerObjects in pairs(espObjects) do
        for _, obj in ipairs(playerObjects) do
            obj.Visible = false
            obj:Remove()
        end
    end
    espObjects = {}

    for _, outroPlayer in pairs(Players:GetPlayers()) do
        if outroPlayer ~= player then
            local outroChar = outroPlayer.Character
            if outroChar and outroChar:FindFirstChild("HumanoidRootPart") and podePuxarOuMostrarESP(outroChar) then
                local playerSkeleton, parts, connections = criarEsqueleto(outroChar)
                local currentESPObjects = {}

                for i, conn in ipairs(connections) do
                    local part1 = parts[conn[1]]
                    local part2 = parts[conn[2]]
                    
                    if part1 and part2 then
                        local screenPoint1, onScreen1 = camera:WorldToScreenPoint(part1.Position)
                        local screenPoint2, onScreen2 = camera:WorldToScreenPoint(part2.Position)

                        if onScreen1 and onScreen2 then
                            local line = playerSkeleton[i]
                            line.From = Vector2.new(screenPoint1.X, screenPoint1.Y)
                            line.To = Vector2.new(screenPoint2.X, screenPoint2.Y)
                            line.Visible = true
                            table.insert(currentESPObjects, line)
                        else
                            playerSkeleton[i].Visible = false
                        end
                    end
                end
                
                local head = parts.Head
                if head then
                    local headScreenPoint, headOnScreen = camera:WorldToScreenPoint(head.Position)
                    if headOnScreen then
                        local headBox = Drawing.new("Box")
                        headBox.Color = COR_ESP
                        headBox.Thickness = 1
                        headBox.Size = Vector2.new(30,30)
                        headBox.Position = Vector2.new(headScreenPoint.X - 15, headScreenPoint.Y - 15)
                        headBox.Visible = true
                        table.insert(currentESPObjects, headBox)
                    end
                end

                espObjects[outroPlayer] = currentESPObjects
            else
                if espObjects[outroPlayer] then
                    for _, obj in ipairs(espObjects[outroPlayer]) do
                        obj.Visible = false
                        obj:Remove()
                    end
                    espObjects[outroPlayer] = nil
                end
            end
        end
    end
end

local function IniciarPuxada()
	if conexaoLoop then conexaoLoop:Disconnect() end
	if conexaoESP then conexaoESP:Disconnect() end

	conexaoLoop = RunService.RenderStepped:Connect(function()
		local meuChar = player.Character
		if not meuChar or not meuChar:FindFirstChild("HumanoidRootPart") then return end
		
		local minhaPos = meuChar.HumanoidRootPart.CFrame
		
		for _, outroPlayer in pairs(Players:GetPlayers()) do
			if outroPlayer ~= player then
				local outroChar = outroPlayer.Character
				if outroChar and outroChar:FindFirstChild("HumanoidRootPart") then
					
					if podePuxarOuMostrarESP(outroChar) then
						local targetRP = outroChar.HumanoidRootPart
						local distancia = (meuChar.HumanoidRootPart.Position - targetRP.Position).Magnitude
						
						if distancia <= DISTANCIA_MAXIMA then
							targetRP.CFrame = minhaPos * OFFSET_POSICAO * CFrame.Angles(0, math.pi, 0)
							targetRP.AssemblyLinearVelocity = Vector3.new(0,0,0)
							targetRP.AssemblyAngularVelocity = Vector3.new(0,0,0)
						end
					end
				end
			end
		end
	end)

    conexaoESP = RunService.RenderStepped:Connect(atualizarEsqueleto)
end

local function PararPuxadaEESP()
	if conexaoLoop then
		conexaoLoop:Disconnect()
		conexaoLoop = nil
	end
    if conexaoESP then
        conexaoESP:Disconnect()
        conexaoESP = nil
    end

    for _, playerObjects in pairs(espObjects) do
        for _, obj in ipairs(playerObjects) do
            obj.Visible = false
            obj:Remove()
        end
    end
    espObjects = {}
end

-- Monitora mudança do toggle de puxada
local lastPullState = false
RunService.Heartbeat:Connect(function()
	if _G.PullEnabled and not lastPullState then
		print("🚀 Puxada ATIVADA!")
		IniciarPuxada()
		lastPullState = true
	elseif not _G.PullEnabled and lastPullState then
		print("⛔ Puxada DESATIVADA!")
		PararPuxadaEESP()
		lastPullState = false
	end
	
	if not player or not player.Parent then
		PararPuxadaEESP()
	end
end)

-- ============================================
-- MENSAGENS INICIAIS
-- ============================================

print("═══════════════════════════════════════════")
print("✅ SCRIPT UNIFICADO OTIMIZADO CARREGADO!")
print("═══════════════════════════════════════════")
print("⚔️  AIMBOT: Smooth aimbot com FOV customizável")
print("👁️  ESP: Box + Nick dos inimigos")
print("💨 PUXADA: Puxa inimigos (integrada ao menu)")
print("🎮 NOCLIP: Atravessa paredes")
print("✈️  FLY: Normal e Nivelado com congelamento")
print("🔧 OTIMIZAÇÃO: 3 Níveis (Normal, Médio, Máximo)")
print("👨‍💼 VERIFICAÇÃO DE ADMINS: Sistema completo")
print("💬 CHAT: Visualizador de mensagens")
print("═══════════════════════════════════════════\n")
displayAdminList()
displayAdminsOnline()
