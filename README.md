local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
   Name = "papagaio hub",
   Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
   LoadingTitle = "pagaio hub",
   LoadingSubtitle = "by rotieh",
   ShowText = "papagaio hub", -- for mobile users to unhide rayfield, change if you'd like
   Theme = "Default", -- Check https://docs.sirius.menu/rayfield/configuration/themes

   ToggleUIKeybind = "K", -- The keybind to toggle the UI visibility (string like "K" or Enum.KeyCode)

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil, -- Create a custom folder for your hub/game
      FileName = "Big Hub"
   },
   }
})
local Button = Tab:CreateButton({
   Name = "tp base",
   Callback = function(
   -- Botão: Teleportar para a BASE (suave)
buttonRow(pageMove, "Teleportar para BASE (suave)", function()
	teleportBase(0.35)
end)
-- Função: faz o teleporte para a base
function teleportBase(tweenTime)
	local hrp = getHRP(LocalPlayer)
	local hum = getHum(LocalPlayer)
	local target = BASE_CF + (BASE_CF.LookVector * S.tpForward)
	if hum then hum.PlatformStand = true end
	if tweenTime and tweenTime > 0 then
		local tw = TweenService:Create(hrp, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = target})
		tw:Play(); tw.Completed:Wait()
	else
		hrp.CFrame = target
	end
	task.delay(0.25, function() if hum then hum.PlatformStand = false end end)
end
   )
})
local Button = Tab:CreateButton({
   Name = "boost speed",
   Callback = function(

-- Funções: ligar/desligar o boost
function startBoost()
	local hum = getHum(LocalPlayer)
	if boostConn then boostConn:Disconnect() end
	boostConn = RunService.Heartbeat:Connect(function()
		if hum and hum.Parent and hum.WalkSpeed ~= S.boostSpeed then
			hum.WalkSpeed = S.boostSpeed
		end
	end)
end

function stopBoost()
	if boostConn then boostConn:Disconnect() end
	boostConn = nil
	local hum = getHum(LocalPlayer)
	if hum then hum.WalkSpeed = 16 end
end

   )
})
local Button = Tab:CreateButton({
   Name = "pulo infinito",
   Callback = function(
   -- Função: pular sempre que apertar espaço
UserInputService.JumpRequest:Connect(function()
	if S.infiniteJump then
		local hum = getHum(LocalPlayer)
		if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end)
   )
   end,
})
