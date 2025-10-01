-- Papagaio Hub (Dev/Admin) - by Rotieh
-- LocalScript em StarterPlayerScripts

-- =========================
-- CONFIGURAÇÕES INICIAIS
-- =========================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local Teams = game:GetService("Teams")

local LocalPlayer = Players.LocalPlayer

-- Quem pode ver/usar o hub
local ADMINS = {
	["Rotieh"] = true,           -- troque / adicione nomes
	[LocalPlayer.Name] = true,   -- garante o dono do script
}

-- Posição base (exemplo: "Shop") — ajuste se quiser
local SHOP_CFRAME = CFrame.new(
	-378.420959, -6.25197887, 59.4905052,
	0.106838159, -1.85177775e-08, 0.994276404,
	1.08971683e-07, 1, 6.91502233e-09,
	-0.994276404, 1.07609189e-07, 0.106838159
)

-- =========================
-- ESTADO DO HUB
-- =========================
local state = {
	uiOpen = true,
	moveForwardStuds = 10,      -- teleporte “mais à frente” (ajustável no slider)
	boostEnabled = false,
	boostSpeed = 42,
	infiniteJump = false,
	espEnabled = false,
	espShowNames = true,
	espShowHealth = true,
	espTeamColor = true,
	espFillTransparency = 0.5,
	espOutlineTransparency = 0,
}

-- =========================
-- FERRAMENTAS AUXILIARES
-- =========================
local function getCharacter(player)
	local char = player.Character or player.CharacterAdded:Wait()
	return char
end

local function getHumanoid(player)
	local char = getCharacter(player)
	return char:WaitForChild("Humanoid")
end

local function getHRP(player)
	local char = getCharacter(player)
	return char:WaitForChild("HumanoidRootPart")
end

local function notify(title, text)
	pcall(function()
		StarterGui:SetCore("SendNotification", {
			Title = title,
			Text = text,
			Duration = 4
		})
	end)
end

-- =========================
-- GUI BÁSICA
-- =========================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PapagaioHub"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.new(0, 320, 0, 420)
main.Position = UDim2.new(0, 20, 0, 120)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
main.BorderSizePixel = 0
main.Parent = screenGui

local corner = Instance.new("UICorner", main)
corner.CornerRadius = UDim.new(0, 14)

local shadow = Instance.new("ImageLabel", main)
shadow.Name = "Shadow"
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://5028857084"
shadow.ImageTransparency = 0.45
shadow.ScaleType = Enum.ScaleType.Slice
shadow.SliceCenter = Rect.new(24, 24, 276, 276)
shadow.Size = UDim2.new(1, 30, 1, 30)
shadow.Position = UDim2.new(0, -15, 0, -15)
shadow.ZIndex = 0

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, -16, 0, 36)
title.Position = UDim2.new(0, 8, 0, 8)
title.BackgroundTransparency = 1
title.Text = "Papagaio Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextXAlignment = Enum.TextXAlignment.Left
title.Font = Enum.Font.GothamBold
title.TextSize = 20

local subtitle = Instance.new("TextLabel", main)
subtitle.Size = UDim2.new(1, -16, 0, 20)
subtitle.Position = UDim2.new(0, 8, 0, 40)
subtitle.BackgroundTransparency = 1
subtitle.Text = "by Rotieh — Admin/Dev tools (para o seu jogo)"
subtitle.TextColor3 = Color3.fromRGB(200, 200, 200)
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 14

-- Drag do painel
do
	local dragging, dragInput, startPos, startMousePos
	main.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			startPos = main.Position
			startMousePos = input.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	main.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input == dragInput then
			local delta = input.Position - startMousePos
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

-- Abas
local tabsBar = Instance.new("Frame", main)
tabsBar.Size = UDim2.new(1, -16, 0, 36)
tabsBar.Position = UDim2.new(0, 8, 0, 66)
tabsBar.BackgroundTransparency = 1

local content = Instance.new("Frame", main)
content.Name = "Content"
content.Size = UDim2.new(1, -16, 1, -118)
content.Position = UDim2.new(0, 8, 0, 110)
content.BackgroundColor3 = Color3.fromRGB(26, 26, 36)
content.BorderSizePixel = 0
local contentCorner = Instance.new("UICorner", content)
contentCorner.CornerRadius = UDim.new(0, 10)

local list = Instance.new("UIListLayout", tabsBar)
list.FillDirection = Enum.FillDirection.Horizontal
list.Padding = UDim.new(0, 6)
list.VerticalAlignment = Enum.VerticalAlignment.Center

local function makeTab(name)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 90, 1, 0)
	btn.Text = name
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.BackgroundColor3 = Color3.fromRGB(34, 34, 46)
	btn.BorderSizePixel = 0
	local c = Instance.new("UICorner", btn); c.CornerRadius = UDim.new(0, 8)
	btn.Parent = tabsBar

	local page = Instance.new("ScrollingFrame")
	page.Name = "Page_"..name
	page.Size = UDim2.new(1, -16, 1, -16)
	page.Position = UDim2.new(0, 8, 0, 8)
	page.CanvasSize = UDim2.new(0,0,0,0)
	page.ScrollBarThickness = 6
	page.BackgroundTransparency = 1
	page.Visible = false
	page.Parent = content

	local grid = Instance.new("UIListLayout", page)
	grid.Padding = UDim.new(0, 8)
	grid.SortOrder = Enum.SortOrder.LayoutOrder

	btn.MouseButton1Click:Connect(function()
		for _,child in ipairs(content:GetChildren()) do
			if child:IsA("ScrollingFrame") then child.Visible = false end
		end
		for _,other in ipairs(tabsBar:GetChildren()) do
			if other:IsA("TextButton") then other.BackgroundColor3 = Color3.fromRGB(34,34,46) end
		end
		page.Visible = true
		btn.BackgroundColor3 = Color3.fromRGB(60,60,88)
	end)

	return btn, page
end

local function makeToggle(parent, label, getter, setter)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(1, -8, 0, 36)
	frame.BackgroundColor3 = Color3.fromRGB(32, 32, 44)
	frame.BorderSizePixel = 0
	local c = Instance.new("UICorner", frame); c.CornerRadius = UDim.new(0, 8)
	frame.Parent = parent

	local txt = Instance.new("TextLabel", frame)
	txt.BackgroundTransparency = 1
	txt.Position = UDim2.new(0, 10, 0, 0)
	txt.Size = UDim2.new(1, -90, 1, 0)
	txt.TextXAlignment = Enum.TextXAlignment.Left
	txt.Text = label
	txt.Font = Enum.Font.Gotham
	txt.TextSize = 14
	txt.TextColor3 = Color3.new(1,1,1)

	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(0, 64, 0, 26)
	btn.Position = UDim2.new(1, -74, 0.5, -13)
	btn.Text = getter() and "ON" or "OFF"
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = getter() and Color3.fromRGB(40, 140, 90) or Color3.fromRGB(100, 40, 40)
	btn.BorderSizePixel = 0
	local c2 = Instance.new("UICorner", btn); c2.CornerRadius = UDim.new(0, 8)

	btn.MouseButton1Click:Connect(function()
		local newVal = not getter()
		setter(newVal)
		btn.Text = newVal and "ON" or "OFF"
		btn.BackgroundColor3 = newVal and Color3.fromRGB(40, 140, 90) or Color3.fromRGB(100, 40, 40)
	end)

	return frame
end

local function makeSlider(parent, label, min, max, step, getter, setter, suffix)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(1, -8, 0, 54)
	frame.BackgroundColor3 = Color3.fromRGB(32, 32, 44)
	frame.BorderSizePixel = 0
	local c = Instance.new("UICorner", frame); c.CornerRadius = UDim.new(0, 8)
	frame.Parent = parent

	local txt = Instance.new("TextLabel", frame)
	txt.BackgroundTransparency = 1
	txt.Position = UDim2.new(0, 10, 0, 0)
	txt.Size = UDim2.new(1, -20, 0, 20)
	txt.TextXAlignment = Enum.TextXAlignment.Left
	txt.Text = label
	txt.Font = Enum.Font.Gotham
	txt.TextSize = 14
	txt.TextColor3 = Color3.new(1,1,1)

	local sliderBg = Instance.new("Frame", frame)
	sliderBg.Size = UDim2.new(1, -20, 0, 8)
	sliderBg.Position = UDim2.new(0, 10, 0, 28)
	sliderBg.BackgroundColor3 = Color3.fromRGB(45,45,60)
	sliderBg.BorderSizePixel = 0
	local c1 = Instance.new("UICorner", sliderBg); c1.CornerRadius = UDim.new(0, 4)

	local fill = Instance.new("Frame", sliderBg)
	fill.Size = UDim2.new(0, 0, 1, 0)
	fill.BackgroundColor3 = Color3.fromRGB(90, 120, 255)
	fill.BorderSizePixel = 0
	local c2 = Instance.new("UICorner", fill); c2.CornerRadius = UDim.new(0, 4)

	local knob = Instance.new("Frame", sliderBg)
	knob.Size = UDim2.new(0, 12, 0, 12)
	knob.Position = UDim2.new(0, -6, 0.5, -6)
	knob.BackgroundColor3 = Color3.fromRGB(200, 210, 255)
	knob.BorderSizePixel = 0
	local c3 = Instance.new("UICorner", knob); c3.CornerRadius = UDim.new(1, 0)

	local valLabel = Instance.new("TextLabel", frame)
	valLabel.BackgroundTransparency = 1
	valLabel.Position = UDim2.new(1, -80, 0, 0)
	valLabel.Size = UDim2.new(0, 70, 0, 20)
	valLabel.TextXAlignment = Enum.TextXAlignment.Right
	valLabel.Text = tostring(getter()) .. (suffix or "")
	valLabel.Font = Enum.Font.Gotham
	valLabel.TextSize = 14
	valLabel.TextColor3 = Color3.fromRGB(220,220,220)

	local function updateVisual(v)
		local t = math.clamp((v - min) / (max - min), 0, 1)
		fill.Size = UDim2.new(t, 0, 1, 0)
		knob.Position = UDim2.new(t, -6, 0.5, -6)
		valLabel.Text = tostring(v) .. (suffix or "")
	end

	updateVisual(getter())

	local dragging = false
	local function setFromX(x)
		local rel = (x - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X
		local raw = min + rel * (max - min)
		local stepped = math.floor((raw / step) + 0.5) * step
		stepped = math.clamp(stepped, min, max)
		setter(stepped)
		updateVisual(stepped)
	end

	sliderBg.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			setFromX(input.Position.X)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			setFromX(input.Position.X)
		end
	end)
	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)

	return frame
end

local function makeButton(parent, label, onClick)
	local frame = Instance.new("TextButton")
	frame.Size = UDim2.new(1, -8, 0, 36)
	frame.Text = label
	frame.Font = Enum.Font.GothamBold
	frame.TextSize = 14
	frame.TextColor3 = Color3.new(1,1,1)
	frame.BackgroundColor3 = Color3.fromRGB(45, 60, 90)
	frame.BorderSizePixel = 0
	local c = Instance.new("UICorner", frame); c.CornerRadius = UDim.new(0, 8)
	frame.Parent = parent

	frame.MouseButton1Click:Connect(function()
		pcall(onClick)
	end)

	return frame
end

-- =========================
-- LÓGICA: BOOST SPEED
-- =========================
local boostConnection
local function startBoost()
	local hum = getHumanoid(LocalPlayer)
	if boostConnection then boostConnection:Disconnect() end
	boostConnection = RunService.Heartbeat:Connect(function()
		if hum and hum.Parent then
			if hum.WalkSpeed ~= state.boostSpeed then
				hum.WalkSpeed = state.boostSpeed
			end
		end
	end)
end

local function stopBoost()
	if boostConnection then boostConnection:Disconnect() end
	boostConnection = nil
	local hum = getHumanoid(LocalPlayer)
	if hum then hum.WalkSpeed = 16 end
end

-- =========================
-- LÓGICA: INFINITE JUMP
-- =========================
UserInputService.JumpRequest:Connect(function()
	if state.infiniteJump then
		local hum = getHumanoid(LocalPlayer)
		if hum then
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- =========================
-- LÓGICA: TELEPORTE
-- =========================
local function teleportForward(baseCF, forwardStuds, tweenTime)
	local hrp = getHRP(LocalPlayer)
	local forward = baseCF.LookVector * (forwardStuds or 10)
	local target = baseCF + forward
	local char = hrp.Parent
	local hum = char:FindFirstChildOfClass("Humanoid")

	if hum then hum.PlatformStand = true end
	if tweenTime and tweenTime > 0 then
		local tween = TweenService:Create(hrp, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = target})
		tween:Play()
		tween.Completed:Wait()
	else
		hrp.CFrame = target
	end
	task.delay(0.3, function()
		if hum then hum.PlatformStand = false end
	end)
end

-- =========================
-- LÓGICA: ESP JOGADORES
-- =========================
local function teamColorFor(player)
	if not state.espTeamColor then
		return Color3.fromRGB(255,255,255)
	end
	local team = player.Team
	if team and team.TeamColor then
		return team.TeamColor.Color
	end
	return Color3.fromRGB(255,255,255)
end

local function ensureESPForCharacter(player)
	if player == LocalPlayer then return end
	local char = player.Character
	if not char then return end

	-- Highlight
	local hl = char:FindFirstChildOfClass("Highlight")
	if not hl then
		hl = Instance.new("Highlight")
		hl.Parent = char
	end
	hl.Enabled = state.espEnabled
	hl.FillTransparency = state.espFillTransparency
	hl.OutlineTransparency = state.espOutlineTransparency
	hl.FillColor = teamColorFor(player)
	hl.OutlineColor = Color3.fromRGB(0,0,0)

	-- Billboard Name/Health
	local billboard = char:FindFirstChild("PapagaioESP")
	if not billboard then
		billboard = Instance.new("BillboardGui")
		billboard.Name = "PapagaioESP"
		billboard.Size = UDim2.new(0, 140, 0, 40)
		billboard.StudsOffset = Vector3.new(0, 3, 0)
		billboard.AlwaysOnTop = true
		billboard.Parent = char

		local lbl = Instance.new("TextLabel")
		lbl.Name = "Text"
		lbl.BackgroundTransparency = 1
		lbl.Size = UDim2.new(1, 0, 1, 0)
		lbl.TextColor3 = Color3.fromRGB(255,255,255)
		lbl.TextStrokeTransparency = 0.4
		lbl.Font = Enum.Font.GothamBold
		lbl.TextScaled = true
		lbl.Parent = billboard
	end

	local lbl = billboard:FindFirstChild("Text")
	local textParts = {}
	if state.espShowNames then
		table.insert(textParts, player.Name)
	end
	if state.espShowHealth then
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hum then
			table.insert(textParts, string.format("HP:%d", hum.Health))
		end
	end
	lbl.Text = table.concat(textParts, " | ")
	billboard.Enabled = state.espEnabled

	-- Atualiza cor continuamente (se trocar de time)
	RunService.Heartbeat:Connect(function()
		if hl and hl.Parent and state.espEnabled then
			hl.FillColor = teamColorFor(player)
			hl.FillTransparency = state.espFillTransparency
			hl.OutlineTransparency = state.espOutlineTransparency
		end
	end)
end

local function refreshESPForAll()
	for _,plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			if plr.Character then
				ensureESPForCharacter(plr)
			end
			plr.CharacterAdded:Connect(function()
				task.wait(0.5)
				ensureESPForCharacter(plr)
			end)
		end
	end
end

Players.PlayerAdded:Connect(function(plr)
	if plr ~= LocalPlayer then
		plr.CharacterAdded:Connect(function()
			task.wait(0.5)
			ensureESPForCharacter(plr)
		end)
	end
end)

-- =========================
-- PÁGINAS / ABAS
-- =========================
local tabMainBtn,   tabMainPage   = makeTab("Main")
local tabMoveBtn,   tabMovePage   = makeTab("Move")
local tabESPBtn,    tabESPPage    = makeTab("ESP")
local tabMiscBtn,   tabMiscPage   = makeTab("Misc")
local tabAboutBtn,  tabAboutPage  = makeTab("Sobre")

-- Seleciona aba inicial
tabMainBtn:Activate()
for _,child in ipairs(content:GetChildren()) do
	if child:IsA("ScrollingFrame") then child.Visible = false end
end
tabMainPage.Visible = true
tabMainBtn.BackgroundColor3 = Color3.fromRGB(60,60,88)

-- MAIN
makeToggle(tabMainPage, "Infinite Jump", function() return state.infiniteJump end, function(v)
	state.infiniteJump = v
	notify("Papagaio Hub", v and "Infinite Jump ON" or "Infinite Jump OFF")
end)

makeToggle(tabMainPage, "Boost Velocidade", function() return state.boostEnabled end, function(v)
	state.boostEnabled = v
	if v then startBoost() else stopBoost() end
end)

makeSlider(tabMainPage, "Velocidade (boost)", 16, 100, 1, function() return state.boostSpeed end, function(newV)
	state.boostSpeed = newV
	if state.boostEnabled then startBoost() end
end, " u/s")

-- MOVE
makeSlider(tabMovePage, "Distância à frente (TP)", 0, 50, 1, function() return state.moveForwardStuds end, function(v)
	state.moveForwardStuds = v
end, " studs")

makeButton(tabMovePage, "Teleportar para Shop (suave)", function()
	teleportForward(SHOP_CFRAME, state.moveForwardStuds, 0.35)
end)

makeButton(tabMovePage, "Teleportar para Shop (instantâneo)", function()
	teleportForward(SHOP_CFRAME, state.moveForwardStuds, 0)
end)

-- ESP
makeToggle(tabESPPage, "ESP Jogadores (ON/OFF)", function() return state.espEnabled end, function(v)
	state.espEnabled = v
	refreshESPForAll()
end)
makeToggle(tabESPPage, "Mostrar Nomes", function() return state.espShowNames end, function(v)
	state.espShowNames = v
	refreshESPForAll()
end)
makeToggle(tabESPPage, "Mostrar Vida", function() return state.espShowHealth end, function(v)
	state.espShowHealth = v
	refreshESPForAll()
end)
makeToggle(tabESPPage, "Cor por Time", function() return state.espTeamColor end, function(v)
	state.espTeamColor = v
	refreshESPForAll()
end)
makeSlider(tabESPPage, "Transparência Fill", 0, 1, 0.05, function() return state.espFillTransparency end, function(v)
	state.espFillTransparency = v
	refreshESPForAll()
end)
makeSlider(tabESPPage, "Transparência Outline", 0, 1, 0.05, function() return state.espOutlineTransparency end, function(v)
	state.espOutlineTransparency = v
	refreshESPForAll()
end)

-- MISC
makeButton(tabMiscPage, "Reentrar no mesmo servidor", function()
	local placeId, jobId = game.PlaceId, game.JobId
	TeleportService:TeleportToPlaceInstance(placeId, jobId, LocalPlayer)
end)

makeButton(tabMiscPage, "Hop para outro servidor público", function()
	-- Observação: Em jogos próprios, você pode manter uma lista de servers.
	notify("Papagaio Hub", "Para server hop público, implemente uma API própria ou UI de places.")
end)

-- SOBRE
do
	local t1 = Instance.new("TextLabel", tabAboutPage)
	t1.Size = UDim2.new(1, -8, 0, 60)
	t1.TextWrapped = true
	t1.BackgroundTransparency = 1
	t1.TextXAlignment = Enum.TextXAlignment.Left
	t1.TextYAlignment = Enum.TextYAlignment.Top
	t1.Font = Enum.Font.Gotham
	t1.TextSize = 14
	t1.TextColor3 = Color3.fromRGB(230,230,230)
	t1.Text = "Papagaio Hub — Ferramentas de Admin/Dev para o SEU jogo.\nCréditos: Rotieh | UI própria | Sem exploits."
end

-- =========================
-- PERMISSÃO DE EXIBIÇÃO
-- =========================
if not ADMINS[LocalPlayer.Name] then
	screenGui.Enabled = false
end

-- =========================
-- RESPAWN HANDLING
-- =========================
LocalPlayer.CharacterAdded:Connect(function()
	if state.boostEnabled then
		task.wait(0.25)
		startBoost()
	end
	if state.espEnabled then
		task.wait(0.5)
		refreshESPForAll()
	end
end)

-- Inicial
task.defer(function()
	refreshESPForAll()
end)

notify("Papagaio Hub", "Pronto! Use as abas para ativar recursos.")
