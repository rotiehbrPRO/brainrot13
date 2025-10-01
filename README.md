local OrionLib 
loadstring(game:HttpGet(https://github.com/rotiehbrPRO/brainrot13/blob/main/README.md"))()
	-- Papagaio Hub (Admin/Dev) - para o SEU jogo no Roblox Studio
-- UI arrastável + mesmas funções do seu exemplo (versão segura, sem executor)
-- Créditos: Rotieh

--=============================
-- Serviços e utilidades
--=============================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer

local function notify(t, m)
	pcall(function()
		StarterGui:SetCore("SendNotification", {Title = t, Text = m, Duration = 4})
	end)
end

local function getChar(p)
	local c = p.Character or p.CharacterAdded:Wait()
	return c
end
local function getHum(p)
	return getChar(p):WaitForChild("Humanoid")
end
local function getHRP(p)
	return getChar(p):WaitForChild("HumanoidRootPart")
end

--=============================
-- Estado do Hub
--=============================
local STATE = {
	tpForwardStuds = 12,
	boostEnabled = false,
	boostSpeed = 42,
	infiniteJump = false,
	espPlayers = false,
	espShowNames = true,
	espShowHealth = true,
	espTeamColor = true,
	espLocked = false,              -- “ESP locked” (acha textos com ‘Xs’)
	fillT = 0.5,
	outT = 0.0,
}

-- Posição base (Shop) – use a sua
local SHOP_CFRAME = CFrame.new(
	-378.420959, -6.25197887, 59.4905052,
	0.106838159, -1.85177775e-08, 0.994276404,
	1.08971683e-07, 1, 6.91502233e-09,
	-0.994276404, 1.07609189e-07, 0.106838159
)

--=============================
-- GUI (arrastável) + Tabs
--=============================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PapagaioHub"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 340, 0, 440)
main.Position = UDim2.new(0, 20, 0, 120)
main.BackgroundColor3 = Color3.fromRGB(22, 22, 30)
main.BorderSizePixel = 0
main.Parent = screenGui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 44)
header.BackgroundTransparency = 1

local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -16, 1, 0)
title.Position = UDim2.new(0, 8, 0, 0)
title.BackgroundTransparency = 1
title.TextXAlignment = Enum.TextXAlignment.Left
title.Text = "🦜 Papagaio Hub — by Rotieh"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Drag
do
	local dragToggle, dragInput, dragStart, startPos
	header.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragToggle = true
			dragStart = input.Position
			startPos = main.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragToggle = false
				end
			end)
		end
	end)
	header.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragToggle then
			local delta = input.Position - dragStart
			main.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
		end
	end)
end

local tabsBar = Instance.new("Frame", main)
tabsBar.Size = UDim2.new(1, -16, 0, 36)
tabsBar.Position = UDim2.new(0, 8, 0, 48)
tabsBar.BackgroundTransparency = 1
local tabsLayout = Instance.new("UIListLayout", tabsBar)
tabsLayout.FillDirection = Enum.FillDirection.Horizontal
tabsLayout.Padding = UDim.new(0, 6)
tabsLayout.VerticalAlignment = Enum.VerticalAlignment.Center

local content = Instance.new("Frame", main)
content.Name = "Content"
content.Size = UDim2.new(1, -16, 1, -96)
content.Position = UDim2.new(0, 8, 0, 92)
content.BackgroundColor3 = Color3.fromRGB(28, 28, 40)
content.BorderSizePixel = 0
Instance.new("UICorner", content).CornerRadius = UDim.new(0, 10)

local function makeTab(name)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 90, 1, 0)
	btn.Text = name
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = Color3.fromRGB(34, 34, 46)
	btn.BorderSizePixel = 0
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
	btn.Parent = tabsBar

	local page = Instance.new("ScrollingFrame")
	page.Name = "Page_"..name
	page.Size = UDim2.new(1, -16, 1, -16)
	page.Position = UDim2.new(0, 8, 0, 8)
	page.ScrollBarThickness = 6
	page.BackgroundTransparency = 1
	page.Visible = false
	page.Parent = content
	local list = Instance.new("UIListLayout", page)
	list.Padding = UDim.new(0, 8)
	list.SortOrder = Enum.SortOrder.LayoutOrder

	btn.MouseButton1Click:Connect(function()
		for _,child in ipairs(content:GetChildren()) do
			if child:IsA("ScrollingFrame") then child.Visible = false end
		end
		for _,other in ipairs(tabsBar:GetChildren()) do
			if other:IsA("TextButton") then
				other.BackgroundColor3 = Color3.fromRGB(34,34,46)
			end
		end
		page.Visible = true
		btn.BackgroundColor3 = Color3.fromRGB(60,60,88)
	end)

	return btn, page
end

local function makeToggle(parent, label, getter, setter)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(1, -8, 0, 36)
	frame.BackgroundColor3 = Color3.fromRGB(32,32,44)
	frame.BorderSizePixel = 0
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
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
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

	btn.MouseButton1Click:Connect(function()
		local nv = not getter()
		setter(nv)
		btn.Text = nv and "ON" or "OFF"
		btn.BackgroundColor3 = nv and Color3.fromRGB(40, 140, 90) or Color3.fromRGB(100, 40, 40)
	end)

	return frame
end

local function makeSlider(parent, label, min, max, step, getter, setter, suffix)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(1, -8, 0, 54)
	frame.BackgroundColor3 = Color3.fromRGB(32, 32, 44)
	frame.BorderSizePixel = 0
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
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

	local bar = Instance.new("Frame", frame)
	bar.Size = UDim2.new(1, -20, 0, 8)
	bar.Position = UDim2.new(0, 10, 0, 28)
	bar.BackgroundColor3 = Color3.fromRGB(45,45,60)
	bar.BorderSizePixel = 0
	Instance.new("UICorner", bar).CornerRadius = UDim.new(0, 4)

	local fill = Instance.new("Frame", bar); fill.BackgroundColor3 = Color3.fromRGB(90, 120, 255)
	fill.Size = UDim2.new(0,0,1,0); fill.BorderSizePixel = 0
	Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 4)

	local knob = Instance.new("Frame", bar); knob.Size = UDim2.new(0, 12, 0, 12)
	knob.Position = UDim2.new(0, -6, 0.5, -6)
	knob.BackgroundColor3 = Color3.fromRGB(200,210,255); knob.BorderSizePixel = 0
	Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

	local val = Instance.new("TextLabel", frame)
	val.BackgroundTransparency = 1
	val.Position = UDim2.new(1, -80, 0, 0)
	val.Size = UDim2.new(0, 70, 0, 20)
	val.TextXAlignment = Enum.TextXAlignment.Right
	val.Font = Enum.Font.Gotham; val.TextSize = 14
	val.TextColor3 = Color3.fromRGB(220,220,220)

	local function updateVisual(v)
		local t = math.clamp((v - min) / (max - min), 0, 1)
		fill.Size = UDim2.new(t, 0, 1, 0)
		knob.Position = UDim2.new(t, -6, 0.5, -6)
		val.Text = tostring(v) .. (suffix or "")
	end
	updateVisual(getter())

	local dragging = false
	local function setFromX(x)
		local rel = (x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X
		local raw = min + rel * (max - min)
		local stepped = math.floor((raw / step) + 0.5) * step
		stepped = math.clamp(stepped, min, max)
		setter(stepped)
		updateVisual(stepped)
	end

	bar.InputBegan:Connect(function(input)
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
	local btn = Instance.new("TextButton", parent)
	btn.Size = UDim2.new(1, -8, 0, 36)
	btn.Text = label
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = Color3.fromRGB(45, 60, 90)
	btn.BorderSizePixel = 0
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
	btn.MouseButton1Click:Connect(function() pcall(onClick) end)
	return btn
end

--=============================
-- Funções: Boost / Jump / TP
--=============================
local boostConn
local function startBoost()
	local hum = getHum(LocalPlayer)
	if boostConn then boostConn:Disconnect() end
	boostConn = RunService.Heartbeat:Connect(function()
		if hum and hum.Parent and hum.WalkSpeed ~= STATE.boostSpeed then
			hum.WalkSpeed = STATE.boostSpeed
		end
	end)
end
local function stopBoost()
	if boostConn then boostConn:Disconnect() end
	boostConn = nil
	local hum = getHum(LocalPlayer)
	if hum then hum.WalkSpeed = 16 end
end

UserInputService.JumpRequest:Connect(function()
	if STATE.infiniteJump then
		local hum = getHum(LocalPlayer)
		if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end)

local function teleportForward(base, dist, tweenTime)
	local hrp = getHRP(LocalPlayer)
	local hum = getHum(LocalPlayer)
	local target = base + (base.LookVector * (dist or 12))
	if hum then hum.PlatformStand = true end
	if tweenTime and tweenTime > 0 then
		local tw = TweenService:Create(hrp, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = target})
		tw:Play(); tw.Completed:Wait()
	else
		hrp.CFrame = target
	end
	task.delay(0.25, function()
		if hum then hum.PlatformStand = false end
	 end)
end

--=============================
-- ESP Players (Highlight + Name/HP)
--=============================
local function teamColorFor(plr)
	if not STATE.espTeamColor then return Color3.fromRGB(255,255,255) end
	local team = plr.Team
	return (team and team.TeamColor and team.TeamColor.Color) or Color3.fromRGB(255,255,255)
end

local function ensureESP(plr)
	if plr == LocalPlayer then return end
	local char = plr.Character
	if not char then return end

	-- Highlight
	local hl = char:FindFirstChildOfClass("Highlight")
	if not hl and STATE.espPlayers then
		hl = Instance.new("Highlight")
		hl.Parent = char
	end
	if hl then
		hl.Enabled = STATE.espPlayers
		hl.FillTransparency = STATE.fillT
		hl.OutlineTransparency = STATE.outT
		hl.FillColor = teamColorFor(plr)
		hl.OutlineColor = Color3.fromRGB(0,0,0)
	end

	-- Billboard
	local bb = char:FindFirstChild("PapagaioESP")
	if not bb and STATE.espPlayers then
		bb = Instance.new("BillboardGui")
		bb.Name = "PapagaioESP"
		bb.AlwaysOnTop = true
		bb.StudsOffset = Vector3.new(0, 3, 0)
		bb.Size = UDim2.new(0, 140, 0, 40)
		bb.Parent = char

		local lbl = Instance.new("TextLabel", bb)
		lbl.Name = "Text"
		lbl.BackgroundTransparency = 1
		lbl.Size = UDim2.new(1, 0, 1, 0)
		lbl.TextColor3 = Color3.fromRGB(255,255,255)
		lbl.TextStrokeTransparency = 0.4
		lbl.Font = Enum.Font.GothamBold
		lbl.TextScaled = true
	end

	if bb then
		local lbl = bb:FindFirstChild("Text")
		local parts = {}
		if STATE.espShowNames then table.insert(parts, plr.Name) end
		if STATE.espShowHealth then
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hum then table.insert(parts, ("HP:%d"):format(hum.Health)) end
		end
		lbl.Text = table.concat(parts, " | ")
		bb.Enabled = STATE.espPlayers
	end
end

local function refreshESPAll()
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer then
			if p.Character then
				ensureESP(p)
			end
			p.CharacterAdded:Connect(function()
				task.wait(0.4)
				ensureESP(p)
			end)
		end
	end
end

Players.PlayerAdded:Connect(function(p)
	if p ~= LocalPlayer then
		p.CharacterAdded:Connect(function()
			task.wait(0.4)
			ensureESP(p)
		end)
	end
end)

--=============================
-- ESP "Locked" (acha textos com “Xs”)
--=============================
local function isJailText(part)
	for _,d in ipairs(part:GetDescendants()) do
		if d:IsA("TextLabel") and d.Text and d.Text:match("%d+s") then
			return d
		end
	end
	return nil
end

local function addLockedESP(part, label)
	if part:FindFirstChild("ESP_JailText") then return end
	local bb = Instance.new("BillboardGui")
	bb.Name = "ESP_JailText"
	bb.Adornee = part
	bb.Size = UDim2.new(0, 120, 0, 22)
	bb.StudsOffset = Vector3.new(0, 3, 0)
	bb.AlwaysOnTop = true
	bb.Parent = part

	local txt = Instance.new("TextLabel", bb)
	txt.BackgroundTransparency = 1
	txt.Size = UDim2.new(1,0,1,0)
	txt.Font = Enum.Font.GothamSemibold
	txt.TextSize = 14
	txt.TextColor3 = Color3.fromRGB(255, 230, 0)
	txt.TextStrokeTransparency = 0.4
	txt.Text = "⛓️ "..label.Text

	local conn
	conn = RunService.RenderStepped:Connect(function()
		if not STATE.espLocked or not label or not txt or not txt.Parent then
			if conn then conn:Disconnect() end
			return
		end
		if label.Text:match("%d+s") then
			txt.Text = "⛓️ "..label.Text
		else
			txt.Text = ""
		end
	end)
end

local function clearLockedESP()
	for _,obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			local g = obj:FindFirstChild("ESP_JailText")
			if g then g:Destroy() end
		end
	end
end

task.spawn(function()
	while true do
		if STATE.espLocked then
			for _,obj in ipairs(workspace:GetDescendants()) do
				if obj:IsA("BasePart") and not obj:FindFirstChild("ESP_JailText") then
					local lbl = isJailText(obj)
					if lbl then addLockedESP(obj, lbl) end
				end
			end
		end
		task.wait(2.5)
	end
end)

--=============================
-- Abas e Controles
--=============================
local bMain, pMain   = makeTab("Main")
local bMove, pMove   = makeTab("Move")
local bESP,  pESP    = makeTab("ESP")
local bMisc, pMisc   = makeTab("Misc")
local bAbout,pAbout  = makeTab("Sobre")

bMain:Activate(); pMain.Visible = true; bMain.BackgroundColor3 = Color3.fromRGB(60,60,88)

-- MAIN
makeToggle(pMain, "Infinite Jump", function() return STATE.infiniteJump end, function(v)
	STATE.infiniteJump = v
	notify("Papagaio Hub", v and "Infinite Jump ON" or "Infinite Jump OFF")
end)

makeToggle(pMain, "Boost Velocidade", function() return STATE.boostEnabled end, function(v)
	STATE.boostEnabled = v
	if v then startBoost() else stopBoost() end
end)

makeSlider(pMain, "Velocidade (boost)", 16, 100, 1, function() return STATE.boostSpeed end, function(v)
	STATE.boostSpeed = v
	if STATE.boostEnabled then startBoost() end
end, " u/s")

-- MOVE
makeSlider(pMove, "Distância à frente (TP)", 0, 50, 1, function() return STATE.tpForwardStuds end, function(v)
	STATE.tpForwardStuds = v
end, " studs")

makeButton(pMove, "Teleportar p/ Shop (suave)", function()
	teleportForward(SHOP_CFRAME, STATE.tpForwardStuds, 0.35)
end)
makeButton(pMove, "Teleportar p/ Shop (instant)", function()
	teleportForward(SHOP_CFRAME, STATE.tpForwardStuds, 0)
end)

-- ESP
makeToggle(pESP, "ESP Jogadores", function() return STATE.espPlayers end, function(v)
	STATE.espPlayers = v
	refreshESPAll()
end)
makeToggle(pESP, "Mostrar Nomes", function() return STATE.espShowNames end, function(v)
	STATE.espShowNames = v
	refreshESPAll()
end)
makeToggle(pESP, "Mostrar Vida", function() return STATE.espShowHealth end, function(v)
	STATE.espShowHealth = v
	refreshESPAll()
end)
makeToggle(pESP, "Cor por Time", function() return STATE.espTeamColor end, function(v)
	STATE.espTeamColor = v
	refreshESPAll()
end)
makeSlider(pESP, "Transparência Fill", 0, 1, 0.05, function() return STATE.fillT end, function(v)
	STATE.fillT = v
	refreshESPAll()
end)
makeSlider(pESP, "Transparência Outline", 0, 1, 0.05, function() return STATE.outT end, function(v)
	STATE.outT = v
	refreshESPAll()
end)

-- “ESP locked”
makeToggle(pESP, "ESP Locked (contagem 'Xs')", function() return STATE.espLocked end, function(v)
	STATE.espLocked = v
	if not v then clearLockedESP() end
end)

-- MISC
makeButton(pMisc, "Reentrar no mesmo servidor", function()
	TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
end)

makeButton(pMisc, "Créditos", function()
	notify("Papagaio Hub", "Feito por Rotieh 🦜 | UI própria | Sem executor")
end)

-- SOBRE
do
	local t = Instance.new("TextLabel", pAbout)
	t.Size = UDim2.new(1, -8, 0, 64)
	t.BackgroundTransparency = 1
	t.TextXAlignment = Enum.TextXAlignment.Left
	t.TextYAlignment = Enum.TextYAlignment.Top
	t.Font = Enum.Font.Gotham
	t.TextSize = 14
	t.TextColor3 = Color3.fromRGB(230,230,230)
	t.TextWrapped = true
	t.Text = "Papagaio Hub — mesmas funções do seu exemplo, porém versão segura para o SEU jogo (Studio). Teleporte com slider, Boost, Infinite Jump, ESP players (nome/vida/cor), ESP 'locked', Rejoin."
end

-- Respawn: restaura Boost/ESP
LocalPlayer.CharacterAdded:Connect(function()
	if STATE.boostEnabled then
		task.wait(0.25)
		startBoost()
	end
	if STATE.espPlayers then
		task.wait(0.4)
		refreshESPAll()
	end
end)

notify("Papagaio Hub", "Pronto! UI arrastável com as mesmas funções (versão segura).")

