
-- Zay Scripts Admin GUI
-- Place in StarterGui LocalScript

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local roleEvent = ReplicatedStorage:WaitForChild("RoleUpdate")

local gui = Instance.new("ScreenGui")
gui.Name = "ZayScripts"
gui.Parent = player.PlayerGui

-- Main window
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 350, 0, 420) -- Resized main frame to fit additional buttons
main.Position = UDim2.new(0.5, -175, 0.5, -210)
main.BackgroundColor3 = Color3.fromRGB(30, 0, 0)
main.BackgroundTransparency = 0.15
main.Parent = gui

local corner = Instance.new("UICorner")
corner.Parent = main

-- Shadow
local shadow = Instance.new("UIStroke")
shadow.Color = Color3.fromRGB(255, 0, 0)
shadow.Thickness = 3
shadow.Parent = main

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Zay Scripts"
title.TextColor3 = Color3.new(1, 0, 0)
title.BackgroundTransparency = 1
title.TextScaled = true
title.Parent = main

-- Toggle circle
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 45, 0, 45)
button.Position = UDim2.new(0, 20, 0.5, 0)
button.Text = "Z"
button.TextScaled = true
button.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
button.TextColor3 = Color3.new(1, 1, 1)
button.Parent = gui

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(1, 0)
buttonCorner.Parent = button

button.MouseButton1Click:Connect(function()
	main.Visible = not main.Visible
end)

-- Drag system
local function drag(obj)
	local dragging = false
	local start, pos

	obj.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			start = input.Position
			pos = obj.Position
		end
	end)

	obj.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - start
			obj.Position = UDim2.new(
				pos.X.Scale,
				pos.X.Offset + delta.X,
				pos.Y.Scale,
				pos.Y.Offset + delta.Y
			)
		end
	end)

	obj.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)
end

drag(main)
drag(button)

-- Advanced Admin ESP
local function addESP(plr, role)
	if not plr.Character then return end
	local character = plr.Character

	local old = character:FindFirstChild("ZayESP")
	if old then old:Destroy() end

	local highlight = Instance.new("Highlight")
	highlight.Name = "ZayESP"
	highlight.Parent = character

	local color = Color3.fromRGB(0, 255, 0)
	if role == "Murderer" then color = Color3.fromRGB(255, 0, 0)
	elseif role == "Sheriff" then color = Color3.fromRGB(0, 100, 255)
	elseif role == "Hero" then color = Color3.fromRGB(255, 255, 0) end

	highlight.FillColor = color

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "RoleESP"
	billboard.Size = UDim2.new(0, 200, 0, 50)
	billboard.StudsOffset = Vector3.new(0, 3, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = character

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, 0, 1, 0)
	text.BackgroundTransparency = 1
	text.TextScaled = true
	text.Font = Enum.Font.GothamBold
	text.Text = plr.Name .. " | " .. role
	text.TextColor3 = color
	text.Parent = billboard
end

roleEvent.OnClientEvent:Connect(function(roleTable)
	for plr, role in pairs(roleTable) do
		if plr ~= player then
			if plr.Character then addESP(plr, role) end
			plr.CharacterAdded:Connect(function()
				task.wait(1)
				addESP(plr, role)
			end)
		end
	end
end)

-- Auto Farm Setup
local autoFarm = false
local farmSpeed = 1

local farmButton = Instance.new("TextButton")
farmButton.Size = UDim2.new(0, 200, 0, 35)
farmButton.Position = UDim2.new(0, 75, 0, 50)
farmButton.Text = "Auto Farm: OFF"
farmButton.Parent = main

farmButton.MouseButton1Click:Connect(function()
	autoFarm = not autoFarm
	farmButton.Text = autoFarm and "Auto Farm: ON" or "Auto Farm: OFF"
end)

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 200, 0, 30)
speedBox.Position = UDim2.new(0, 75, 0, 90)
speedBox.Text = "1"
speedBox.PlaceholderText = "Speed 1-30"
speedBox.Parent = main

speedBox.FocusLost:Connect(function()
	local value = tonumber(speedBox.Text)
	if value then
		farmSpeed = math.clamp(value, 1, 30)
		speedBox.Text = tostring(farmSpeed)
	end
end)

task.spawn(function()
	while task.wait(1) do
		if autoFarm then
			local coins = workspace:FindFirstChild("Coins")
			if coins then
				for _, coin in pairs(coins:GetChildren()) do
					if coin:IsA("BasePart") and player.Character then
						player.Character:MoveTo(coin.Position)
						task.wait(farmSpeed)
						ReplicatedStorage.CollectCoin:FireServer(coin)
					end
				end
			end
		end
	end
end)

-- Auto Shoot Setup
local autoShoot = false
local shootRange = 100

local shootButton = Instance.new("TextButton")
shootButton.Size = UDim2.new(0, 200, 0, 35)
shootButton.Position = UDim2.new(0, 75, 0, 130)
shootButton.Text = "Auto Shoot: OFF"
shootButton.Parent = main

shootButton.MouseButton1Click:Connect(function()
	autoShoot = not autoShoot
	shootButton.Text = autoShoot and "Auto Shoot: ON" or "Auto Shoot: OFF"
end)

local rangeBox = Instance.new("TextBox")
rangeBox.Size = UDim2.new(0, 200, 0, 30)
rangeBox.Position = UDim2.new(0, 75, 0, 170)
rangeBox.Text = "100"
rangeBox.Parent = main

rangeBox.FocusLost:Connect(function()
	local value = tonumber(rangeBox.Text)
	if value then
		shootRange = math.clamp(value, 10, 500)
		rangeBox.Text = tostring(shootRange)
	end
end)

task.spawn(function()
	while task.wait(0.2) do
		if autoShoot then
			local myChar = player.Character
			if myChar and myChar:FindFirstChild("HumanoidRootPart") then
				for _, target in pairs(Players:GetPlayers()) do
					if target ~= player and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
						local distance = (myChar.HumanoidRootPart.Position - target.Character.HumanoidRootPart.Position).Magnitude
						if distance <= shootRange then
							local role = target:FindFirstChild("Role")
							if role and role.Value == "Murderer" then
								ReplicatedStorage.ShootRequest:FireServer(target)
							end
						end
					end
				end
			end
		end
	end
end)

-- Infinite Jump
local infiniteJump = false

local jumpButton = Instance.new("TextButton")
jumpButton.Size = UDim2.new(0, 200, 0, 35)
jumpButton.Position = UDim2.new(0, 75, 0, 210)
jumpButton.Text = "Infinite Jump: OFF"
jumpButton.Parent = main

jumpButton.MouseButton1Click:Connect(function()
	infiniteJump = not infiniteJump
	jumpButton.Text = infiniteJump and "Infinite Jump: ON" or "Infinite Jump: OFF"
end)

UserInputService.JumpRequest:Connect(function()
	if infiniteJump and player.Character then
		local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

-- Noclip
local noclip = false

local noclipButton = Instance.new("TextButton")
noclipButton.Size = UDim2.new(0, 200, 0, 35)
noclipButton.Position = UDim2.new(0, 75, 0, 250)
noclipButton.Text = "Noclip: OFF"
noclipButton.Parent = main

noclipButton.MouseButton1Click:Connect(function()
	noclip = not noclip
	noclipButton.Text = noclip and "Noclip: ON" or "Noclip: OFF"
end)

RunService.Stepped:Connect(function()
	if noclip and player.Character then
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
			end
		end
	end
end)

-- NEW: Auto Pickup Feature
local autoPickup = false

local pickupButton = Instance.new("TextButton")
pickupButton.Size = UDim2.new(0, 200, 0, 35)
pickupButton.Position = UDim2.new(0, 75, 0, 290)
pickupButton.Text = "Auto Pickup: OFF"
pickupButton.Parent = main

pickupButton.MouseButton1Click:Connect(function()
	autoPickup = not autoPickup
	pickupButton.Text = autoPickup and "Auto Pickup: ON" or "Auto Pickup: OFF"
end)

task.spawn(function()
	while task.wait(0.5) do
		if autoPickup then
			local character = player.Character
			if character and character:FindFirstChild("HumanoidRootPart") then
				local root = character.HumanoidRootPart
				
				-- Look for dropped tools or objects in the workspace
				for _, item in pairs(workspace:GetChildren()) do
					if item:IsA("Tool") or item:IsA("Part") and item.Name == "GunDrop" then
						local handle = item:FindFirstChild("Handle") or (item:IsA("Part") and item)
						if handle then
							-- Move character root to item location to trigger pickup
							firetouchinterest(root, handle, 0)
							firetouchinterest(root, handle, 1)
						end
					end
				end
			end
		end
	end
end)
