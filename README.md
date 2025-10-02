local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "papagaio hub",
   Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
   LoadingTitle = "papagaio hub carregando...",
   LoadingSubtitle = "by rotieh",
   ShowText = "Rayfield", -- for mobile users to unhide rayfield, change if you'd like
   Theme = "Amethyst", -- Check https://docs.sirius.menu/rayfield/configuration/themes

   ToggleUIKeybind = "K", -- The keybind to toggle the UI visibility (string like "K" or Enum.KeyCode)

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil, -- Create a custom folder for your hub/game
      FileName = "Big Hub"
   },

   Discord = {
      Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
      Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ ABCD would be ABCD
      RememberJoins = true -- Set this to false to make them join the discord every time they load it up
   },

   KeySystem = false, -- Set this to true to use our key system
   KeySettings = {
      Title = "Untitled",
      Subtitle = "Key System",
      Note = "No method of obtaining the key is provided", -- Use this to tell the user how to get a key
      FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
      SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
      GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
      Key = {"Hello"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
   }
})
local Tab = Window:CreateTab("roubo", 4483362458) -- Title, Image

Rayfield:Notify({
   Title = "roubou",
   Content = "Notification Content",
   Duration = 6.5,
   Image = 4483362458,
})
local Button = roubo:CreateButton({
   Name = "roubar",
   Callback = function(
      -- =========================
-- LÓGICA: TELEPORTE
-- =========================
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
   )


   local Button = roubo:CreateButton({
   Name = "boost speed",
   Callback = function(
      if state.infiniteJump then
		local hum = getHumanoid(LocalPlayer)
		if hum then
			hum:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)
   )
   -- The function that takes place when the button is pressed
   end,
})
local Button = roubo:CreateButton({
   Name = "Button Example",
   Callback = function(
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

   )
   -- The function that takes place when the button is pressed
   end,
})
