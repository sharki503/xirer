-- ZM MENU V1 (corrigido)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local LP = Players.LocalPlayer

pcall(function()
	local oldGui = game.CoreGui:FindFirstChild("ZM_MENU_V1")
	if oldGui then
		oldGui:Destroy()
	end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "ZM_MENU_V1"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = game.CoreGui

local mobileBtn = Instance.new("TextButton")
mobileBtn.Name = "OpenButton"
mobileBtn.Parent = gui
mobileBtn.Size = UDim2.new(0, 120, 0, 35)
mobileBtn.Position = UDim2.new(0.5, -60, 0, 5)
mobileBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mobileBtn.BorderColor3 = Color3.fromRGB(255, 0, 0)
mobileBtn.Text = "ZM MENU"
mobileBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
mobileBtn.Font = Enum.Font.SourceSansBold
mobileBtn.TextSize = 22

local frame = Instance.new("Frame")
frame.Name = "MainFrame"
frame.Parent = gui
frame.Size = UDim2.new(0, 520, 0, 360)
frame.Position = UDim2.new(0.5, -260, 0.5, -180)
frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
frame.BorderColor3 = Color3.fromRGB(255, 0, 0)
frame.Visible = false
frame.Active = true

local title = Instance.new("TextLabel")
title.Parent = frame
title.Size = UDim2.new(1, 0, 0, 44)
title.BackgroundTransparency = 1
title.Text = "ZM MENU V1"
title.TextColor3 = Color3.fromRGB(255, 0, 0)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 28

local listHolder = Instance.new("Frame")
listHolder.Parent = frame
listHolder.Size = UDim2.new(0, 150, 1, -58)
listHolder.Position = UDim2.new(0, 10, 0, 48)
listHolder.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
listHolder.BorderColor3 = Color3.fromRGB(50, 50, 50)

local listTitle = Instance.new("TextLabel")
listTitle.Parent = listHolder
listTitle.Size = UDim2.new(1, 0, 0, 28)
listTitle.BackgroundTransparency = 1
listTitle.Text = "PLAYERS"
listTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
listTitle.Font = Enum.Font.SourceSansBold
listTitle.TextSize = 20

local listFrame = Instance.new("ScrollingFrame")
listFrame.Parent = listHolder
listFrame.Size = UDim2.new(1, -8, 1, -36)
listFrame.Position = UDim2.new(0, 4, 0, 30)
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.ScrollBarThickness = 4
listFrame.BackgroundTransparency = 1
listFrame.BorderSizePixel = 0

local listLayout = Instance.new("UIListLayout")
listLayout.Parent = listFrame
listLayout.Padding = UDim.new(0, 6)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder

local rightPanel = Instance.new("Frame")
rightPanel.Parent = frame
rightPanel.Size = UDim2.new(1, -180, 1, -58)
rightPanel.Position = UDim2.new(0, 170, 0, 48)
rightPanel.BackgroundTransparency = 1

local statusLabel = Instance.new("TextLabel")
statusLabel.Parent = rightPanel
statusLabel.Size = UDim2.new(1, 0, 0, 28)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Status: aguardando"
statusLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 22

local function setStatus(text)
	statusLabel.Text = "Status: " .. text
end

local dragging = false
local dragStart = nil
local startPos = nil

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

frame.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

UIS.InputChanged:Connect(function(input)
	if not dragging then
		return
	end

	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

local function toggleMenu()
	frame.Visible = not frame.Visible
	setStatus(frame.Visible and "menu aberto" or "menu fechado")
end

UIS.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then
		return
	end

	if input.KeyCode == Enum.KeyCode.K then
		toggleMenu()
	end
end)

mobileBtn.MouseButton1Click:Connect(toggleMenu)

local followTarget = nil
local aimEnabled = false
local espEnabled = false
local flying = false
local noclip = false

local aimFOV = 150
local aimSmooth = 0.2
local flySpeed = 60

local drawings = {}

local function hideESP(esp)
	if not esp then
		return
	end

	esp.box.Visible = false
	esp.name.Visible = false
	esp.tracer.Visible = false
	esp.hp.Visible = false
	esp.dist.Visible = false
	esp.highlight.Parent = nil
end

local function createESP(player)
	if player == LP or drawings[player] then
		return
	end

	local box = Drawing.new("Square")
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Thickness = 1.5
	box.Filled = false
	box.Visible = false

	local name = Drawing.new("Text")
	name.Color = Color3.fromRGB(255, 255, 255)
	name.Size = 14
	name.Center = true
	name.Outline = true
	name.Visible = false

	local tracer = Drawing.new("Line")
	tracer.Color = Color3.fromRGB(255, 0, 0)
	tracer.Thickness = 1.5
	tracer.Visible = false

	local hp = Drawing.new("Text")
	hp.Color = Color3.fromRGB(0, 255, 0)
	hp.Size = 13
	hp.Center = true
	hp.Outline = true
	hp.Visible = false

	local dist = Drawing.new("Text")
	dist.Color = Color3.fromRGB(255, 255, 0)
	dist.Size = 13
	dist.Center = true
	dist.Outline = true
	dist.Visible = false

	local highlight = Instance.new("Highlight")
	highlight.FillColor = Color3.fromRGB(255, 0, 0)
	highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
	highlight.FillTransparency = 0.75
	highlight.OutlineTransparency = 0.2
	highlight.Enabled = true

	drawings[player] = {
		box = box,
		name = name,
		tracer = tracer,
		hp = hp,
		dist = dist,
		highlight = highlight
	}
end

local function removeESP(player)
	local esp = drawings[player]
	if not esp then
		return
	end

	hideESP(esp)
	esp.box:Remove()
	esp.name:Remove()
	esp.tracer:Remove()
	esp.hp:Remove()
	esp.dist:Remove()
	esp.highlight:Destroy()
	drawings[player] = nil
end

local function refreshPlayers()
	for _, child in ipairs(listFrame:GetChildren()) do
		if child:IsA("TextButton") then
			child:Destroy()
		end
	end

	local count = 0
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LP then
			count = count + 1

			local btn = Instance.new("TextButton")
			btn.Parent = listFrame
			btn.Size = UDim2.new(1, -4, 0, 30)
			btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
			btn.BorderColor3 = Color3.fromRGB(60, 60, 60)
			btn.TextColor3 = Color3.fromRGB(255, 255, 255)
			btn.Font = Enum.Font.SourceSans
			btn.TextSize = 20
			btn.Text = player.Name

			btn.MouseButton1Click:Connect(function()
				if followTarget == player then
					followTarget = nil
					setStatus("follow desligado")
				else
					followTarget = player
					setStatus("seguindo " .. player.Name)
				end
			end)
		end
	end

	listFrame.CanvasSize = UDim2.new(0, 0, 0, count * 36)
end

local nextY = 40

local function createToggle(label, defaultState, onToggle)
	local button = Instance.new("TextButton")
	button.Parent = rightPanel
	button.Size = UDim2.new(0, 155, 0, 34)
	button.Position = UDim2.new(0, ((nextY - 40) % 2 == 0) and 0 or 165, 0, nextY)
	button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	button.BorderColor3 = Color3.fromRGB(255, 0, 0)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.SourceSansBold
	button.TextSize = 21

	local state = defaultState

	local function render()
		button.Text = string.format("%s: %s", label, state and "ON" or "OFF")
		button.BackgroundColor3 = state and Color3.fromRGB(80, 15, 15) or Color3.fromRGB(35, 35, 35)
	end

	button.MouseButton1Click:Connect(function()
		state = not state
		render()
		onToggle(state)
	end)

	render()
	nextY = nextY + 42
	return button
end

local function createSlider(label, min, max, step, defaultValue, onChange)
	local button = Instance.new("TextButton")
	button.Parent = rightPanel
	button.Size = UDim2.new(0, 320, 0, 34)
	button.Position = UDim2.new(0, 0, 0, nextY)
	button.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
	button.BorderColor3 = Color3.fromRGB(75, 75, 75)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.SourceSans
	button.TextSize = 21

	local value = defaultValue

	local function render()
		button.Text = string.format("%s: %s", label, tostring(value))
	end

	button.MouseButton1Click:Connect(function()
		value = value + step
		if value > max then
			value = min
		end

		render()
		onChange(value)
	end)

	render()
	nextY = nextY + 42
end

createToggle("ESP", false, function(state)
	espEnabled = state
	setStatus(state and "ESP ligado" or "ESP desligado")
	if not state then
		for _, esp in pairs(drawings) do
			hideESP(esp)
		end
	end
end)

createToggle("AIM", false, function(state)
	aimEnabled = state
	setStatus(state and "AIM ligado" or "AIM desligado")
end)

createToggle("FLY", false, function(state)
	flying = state
	setStatus(state and "FLY ligado" or "FLY desligado")
end)

createToggle("NOCLIP", false, function(state)
	noclip = state
	setStatus(state and "NOCLIP ligado" or "NOCLIP desligado")
end)

createSlider("Aim FOV", 50, 300, 25, 150, function(value)
	aimFOV = value
	setStatus("Aim FOV = " .. tostring(value))
end)

createSlider("Smooth x10", 1, 10, 1, 2, function(value)
	aimSmooth = value / 10
	setStatus("Smooth = " .. tostring(aimSmooth))
end)

createSlider("Fly Speed", 20, 150, 10, 60, function(value)
	flySpeed = value
	setStatus("Fly Speed = " .. tostring(value))
end)

Players.PlayerAdded:Connect(function(player)
	createESP(player)
	refreshPlayers()
end)

Players.PlayerRemoving:Connect(function(player)
	if followTarget == player then
		followTarget = nil
	end
	removeESP(player)
	refreshPlayers()
end)

for _, player in ipairs(Players:GetPlayers()) do
	createESP(player)
end

refreshPlayers()

local function getCharacterRoot(player)
	local character = player and player.Character
	if not character then
		return nil
	end

	return character:FindFirstChild("HumanoidRootPart")
end

local function getClosestTarget()
	local closest = nil
	local bestDistance = math.huge
	local mousePos = UIS:GetMouseLocation()

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LP then
			local root = getCharacterRoot(player)
			local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
			if root and humanoid and humanoid.Health > 0 then
				local pos, visible = Camera:WorldToViewportPoint(root.Position)
				if visible then
					local distance = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
					if distance < bestDistance and distance <= aimFOV then
						bestDistance = distance
						closest = player
					end
				end
			end
		end
	end

	return closest
end

RunService.RenderStepped:Connect(function()
	if followTarget then
		local root = getCharacterRoot(followTarget)
		if root then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, root.Position)
		end
	end

	if aimEnabled then
		local target = getClosestTarget()
		local root = target and getCharacterRoot(target)
		if root then
			local goal = CFrame.new(Camera.CFrame.Position, root.Position)
			Camera.CFrame = Camera.CFrame:Lerp(goal, aimSmooth)
		end
	end

	for player, esp in pairs(drawings) do
		if not espEnabled then
			hideESP(esp)
		else
			local root = getCharacterRoot(player)
			local localRoot = getCharacterRoot(LP)
			local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")

			if root and localRoot and humanoid and humanoid.Health > 0 then
				local pos, visible = Camera:WorldToViewportPoint(root.Position)
				if visible and pos.Z > 0 then
					local size = math.clamp(2000 / pos.Z, 18, 180)

					esp.box.Size = Vector2.new(size, size * 1.45)
					esp.box.Position = Vector2.new(pos.X - size / 2, pos.Y - size * 0.72)
					esp.box.Visible = true

					esp.name.Text = player.Name
					esp.name.Position = Vector2.new(pos.X, pos.Y - size)
					esp.name.Visible = true

					esp.hp.Text = "HP: " .. math.floor(humanoid.Health)
					esp.hp.Position = Vector2.new(pos.X, pos.Y + size * 0.75)
					esp.hp.Visible = true

					esp.dist.Text = math.floor((localRoot.Position - root.Position).Magnitude) .. "m"
					esp.dist.Position = Vector2.new(pos.X, pos.Y + size * 0.75 + 14)
					esp.dist.Visible = true

					esp.tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y - 8)
					esp.tracer.To = Vector2.new(pos.X, pos.Y)
					esp.tracer.Visible = true

					esp.highlight.Parent = player.Character
				else
					hideESP(esp)
				end
			else
				hideESP(esp)
			end
		end
	end
end)

RunService.RenderStepped:Connect(function()
	if flying then
		local root = getCharacterRoot(LP)
		if root then
			local direction = Vector3.new(0, 0, 0)

			if UIS:IsKeyDown(Enum.KeyCode.W) then
				direction = direction + Camera.CFrame.LookVector
			end
			if UIS:IsKeyDown(Enum.KeyCode.S) then
				direction = direction - Camera.CFrame.LookVector
			end
			if UIS:IsKeyDown(Enum.KeyCode.A) then
				direction = direction - Camera.CFrame.RightVector
			end
			if UIS:IsKeyDown(Enum.KeyCode.D) then
				direction = direction + Camera.CFrame.RightVector
			end

			if direction.Magnitude > 0 then
				root.Velocity = direction.Unit * flySpeed
			else
				root.Velocity = Vector3.new(0, 0, 0)
			end
		end
	end
end)

RunService.Stepped:Connect(function()
	local character = LP.Character
	if not character then
		return
	end

	for _, part in ipairs(character:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = not noclip
		end
	end
end)

setStatus("script carregado")
