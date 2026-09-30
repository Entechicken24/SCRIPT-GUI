# SCRIPT-GUI
--========================================================
-- PIET SCRIPT HUB V3
-- Single LocalScript
-- StarterPlayer > StarterPlayerScripts
--========================================================

--// SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--========================================================
-- CORE SETTINGS
--========================================================

local VERSION = "V3.0"

local COLORS = {
	Dark = {
		BG = Color3.fromRGB(14, 16, 22),
		PANEL = Color3.fromRGB(21, 24, 32),
		PANEL2 = Color3.fromRGB(29, 33, 43),
		TEXT = Color3.fromRGB(240, 243, 250),
		MUTED = Color3.fromRGB(150, 158, 173)
	},

	Light = {
		BG = Color3.fromRGB(235, 238, 244),
		PANEL = Color3.fromRGB(250, 251, 253),
		PANEL2 = Color3.fromRGB(224, 228, 236),
		TEXT = Color3.fromRGB(25, 28, 35),
		MUTED = Color3.fromRGB(95, 102, 115)
	}
}

local ACCENTS = {
	Blue = Color3.fromRGB(70, 140, 255),
	Purple = Color3.fromRGB(150, 90, 255),
	Cyan = Color3.fromRGB(50, 210, 220),
	Green = Color3.fromRGB(65, 200, 120),
	Red = Color3.fromRGB(230, 75, 85),
	Orange = Color3.fromRGB(240, 145, 55),
	Pink = Color3.fromRGB(240, 80, 170)
}

local themeName = "Dark"
local accentName = "Blue"
local ACCENT = ACCENTS[accentName]

local function theme()
	return COLORS[themeName]
end

local function BG()
	return theme().BG
end

local function PANEL()
	return theme().PANEL
end

local function PANEL2()
	return theme().PANEL2
end

local function TEXT()
	return theme().TEXT
end

local function MUTED()
	return theme().MUTED
end

--========================================================
-- STATE
--========================================================

local currentPage = "Home"
local guiDestroyed = false

local walkSpeed = 16
local jumpPower = 50

local flySpeed = 60
local flying = false

local infiniteJump = false
local noclip = false
local antiAFK = false

local soundsEnabled = true
local animationsEnabled = true

local spectatingPlayer = nil

local waypoints = {}
local savedWaypointSets = {}

local playerSearch = ""

-- keyboard
local keys = {
	W = false,
	A = false,
	S = false,
	D = false,
	Up = false,
	Down = false
}

-- mobile
local mobileMove = Vector2.zero
local mobileUp = false
local mobileDown = false

--========================================================
-- CHARACTER
--========================================================

local function getCharacter()
	return LocalPlayer.Character
end

local function getHumanoid()
	local char = getCharacter()
	return char and char:FindFirstChildOfClass("Humanoid")
end

local function getRoot()
	local char = getCharacter()
	return char and char:FindFirstChild("HumanoidRootPart")
end

--========================================================
-- GUI
--========================================================

local GUI = Instance.new("ScreenGui")
GUI.Name = "PietScriptHubV3"
GUI.ResetOnSpawn = false
GUI.IgnoreGuiInset = true
GUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
GUI.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.fromOffset(780, 520)
Main.Position = UDim2.new(0.5, -390, 0.5, -260)
Main.BackgroundColor3 = BG()
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
Main.Parent = GUI

Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = ACCENT
MainStroke.Transparency = 0.7
MainStroke.Parent = Main

--========================================================
-- SOUND
--========================================================

local function clickSound()
	if not soundsEnabled then
		return
	end

	local sound = Instance.new("Sound")
	sound.SoundId = "rbxassetid://6895079853"
	sound.Volume = 0.18
	sound.Parent = GUI
	sound:Play()

	task.delay(1, function()
		if sound then
			sound:Destroy()
		end
	end)
end

--========================================================
-- TOP BAR
--========================================================

local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 56)
TopBar.BackgroundColor3 = PANEL()
TopBar.BorderSizePixel = 0
TopBar.Parent = Main

local Title = Instance.new("TextLabel")
Title.BackgroundTransparency = 1
Title.Position = UDim2.fromOffset(18, 7)
Title.Size = UDim2.fromOffset(400, 25)
Title.Font = Enum.Font.GothamBold
Title.Text = "PIET SCRIPT HUB"
Title.TextColor3 = TEXT()
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

local Version = Instance.new("TextLabel")
Version.BackgroundTransparency = 1
Version.Position = UDim2.fromOffset(19, 31)
Version.Size = UDim2.fromOffset(200, 18)
Version.Font = Enum.Font.Gotham
Version.Text = VERSION
Version.TextColor3 = MUTED()
Version.TextSize = 11
Version.TextXAlignment = Enum.TextXAlignment.Left
Version.Parent = TopBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.fromOffset(38, 38)
CloseButton.Position = UDim2.new(1, -47, 0, 9)
CloseButton.BackgroundColor3 = PANEL2()
CloseButton.Text = "×"
CloseButton.TextColor3 = TEXT()
CloseButton.TextSize = 24
CloseButton.Font = Enum.Font.GothamBold
CloseButton.BorderSizePixel = 0
CloseButton.Parent = TopBar

Instance.new("UICorner", CloseButton).CornerRadius = UDim.new(0, 9)

--========================================================
-- SIDEBAR
--========================================================

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.fromOffset(180, 464)
Sidebar.Position = UDim2.fromOffset(0, 56)
Sidebar.BackgroundColor3 = PANEL()
Sidebar.BorderSizePixel = 0
Sidebar.Parent = Main

local SidePadding = Instance.new("UIPadding")
SidePadding.PaddingTop = UDim.new(0, 12)
SidePadding.PaddingLeft = UDim.new(0, 9)
SidePadding.PaddingRight = UDim.new(0, 9)
SidePadding.Parent = Sidebar

local SideLayout = Instance.new("UIListLayout")
SideLayout.Padding = UDim.new(0, 6)
SideLayout.SortOrder = Enum.SortOrder.LayoutOrder
SideLayout.Parent = Sidebar

local pages = {
	"Home",
	"Players",
	"Fly",
	"Movement",
	"Waypoints",
	"Utilities",
	"Settings"
}

local pageButtons = {}

local function createSideButton(name)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, 0, 0, 40)
	button.BackgroundColor3 = PANEL()
	button.BorderSizePixel = 0
	button.Text = "  " .. name
	button.TextColor3 = MUTED()
	button.TextSize = 13
	button.Font = Enum.Font.GothamMedium
	button.TextXAlignment = Enum.TextXAlignment.Left
	button.AutoButtonColor = false
	button.Parent = Sidebar

	Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)

	pageButtons[name] = button

	button.MouseEnter:Connect(function()
		if currentPage ~= name then
			TweenService:Create(
				button,
				TweenInfo.new(0.12),
				{BackgroundColor3 = PANEL2()}
			):Play()
		end
	end)

	button.MouseLeave:Connect(function()
		if currentPage ~= name then
			TweenService:Create(
				button,
				TweenInfo.new(0.12),
				{BackgroundColor3 = PANEL()}
			):Play()
		end
	end)

	return button
end

for _, page in ipairs(pages) do
	createSideButton(page)
end

--========================================================
-- CONTENT
--========================================================

local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -180, 1, -56)
Content.Position = UDim2.fromOffset(180, 56)
Content.BackgroundColor3 = BG()
Content.BorderSizePixel = 0
Content.ClipsDescendants = true
Content.Parent = Main

local ContentPadding = Instance.new("UIPadding")
ContentPadding.PaddingTop = UDim.new(0, 15)
ContentPadding.PaddingLeft = UDim.new(0, 18)
ContentPadding.PaddingRight = UDim.new(0, 18)
ContentPadding.PaddingBottom = UDim.new(0, 15)
ContentPadding.Parent = Content

--========================================================
-- HELPERS
--========================================================

local function clearContent()
	for _, child in ipairs(Content:GetChildren()) do
		if not child:IsA("UIPadding") then
			child:Destroy()
		end
	end
end

local function label(parent, text, pos, size, font, textSize, color)
	local x = Instance.new("TextLabel")
	x.BackgroundTransparency = 1
	x.Position = pos
	x.Size = size
	x.Font = font or Enum.Font.Gotham
	x.Text = text
	x.TextColor3 = color or TEXT()
	x.TextSize = textSize or 14
	x.TextWrapped = true
	x.TextXAlignment = Enum.TextXAlignment.Left
	x.TextYAlignment = Enum.TextYAlignment.Center
	x.Parent = parent
	return x
end

local function button(parent, text, pos, size, color)
	local b = Instance.new("TextButton")
	b.Position = pos
	b.Size = size
	b.BackgroundColor3 = color or PANEL2()
	b.BorderSizePixel = 0
	b.Text = text
	b.TextColor3 = TEXT()
	b.TextSize = 13
	b.Font = Enum.Font.GothamMedium
	b.AutoButtonColor = false
	b.Parent = parent

	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

	b.MouseButton1Click:Connect(clickSound)

	b.MouseEnter:Connect(function()
		TweenService:Create(
			b,
			TweenInfo.new(0.12),
			{BackgroundColor3 = ACCENT}
		):Play()
	end)

	b.MouseLeave:Connect(function()
		TweenService:Create(
			b,
			TweenInfo.new(0.12),
			{BackgroundColor3 = color or PANEL2()}
		):Play()
	end)

	return b
end

local function textbox(parent, placeholder, pos, size)
	local b = Instance.new("TextBox")
	b.Position = pos
	b.Size = size
	b.BackgroundColor3 = PANEL2()
	b.BorderSizePixel = 0
	b.PlaceholderText = placeholder
	b.PlaceholderColor3 = MUTED()
	b.TextColor3 = TEXT()
	b.TextSize = 13
	b.Font = Enum.Font.Gotham
	b.ClearTextOnFocus = false
	b.Parent = parent

	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

	return b
end

local function notify(message)
	if guiDestroyed then
		return
	end

	local n = Instance.new("TextLabel")
	n.AnchorPoint = Vector2.new(1, 1)
	n.Position = UDim2.new(1, -18, 1, -18)
	n.Size = UDim2.fromOffset(270, 42)
	n.BackgroundColor3 = PANEL2()
	n.BorderSizePixel = 0
	n.Text = message
	n.TextColor3 = TEXT()
	n.TextSize = 13
	n.Font = Enum.Font.GothamMedium
	n.Parent = GUI

	Instance.new("UICorner", n).CornerRadius = UDim.new(0, 9)

	local stroke = Instance.new("UIStroke")
	stroke.Color = ACCENT
	stroke.Parent = n

	task.delay(2, function()
		if n and n.Parent then
			TweenService:Create(
				n,
				TweenInfo.new(0.2),
				{
					BackgroundTransparency = 1,
					TextTransparency = 1
				}
			):Play()

			task.wait(0.2)

			if n then
				n:Destroy()
			end
		end
	end)
end

local function toggleButton(parent, text, value, pos, callback)
	local b = button(
		parent,
		text .. ": " .. (value and "ON" or "OFF"),
		pos,
		UDim2.new(1, -10, 0, 42),
		value and ACCENT or PANEL2()
	)

	b.MouseButton1Click:Connect(function()
		callback()
	end)

	return b
end

--========================================================
-- HOME
--========================================================

local function showHome()
	clearContent()

	label(
		Content,
		"Welcome to Piet Script Hub",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 40),
		Enum.Font.GothamBold,
		24
	)

	label(
		Content,
		"Made by Piet and ChatGPT • " .. VERSION,
		UDim2.fromOffset(6, 40),
		UDim2.new(1, -10, 0, 25),
		Enum.Font.Gotham,
		13,
		MUTED()
	)

	local card = Instance.new("Frame")
	card.Position = UDim2.fromOffset(5, 82)
	card.Size = UDim2.new(1, -10, 0, 285)
	card.BackgroundColor3 = PANEL()
	card.BorderSizePixel = 0
	card.Parent = Content

	Instance.new("UICorner", card).CornerRadius = UDim.new(0, 12)

	label(
		card,
		"V3 FEATURES",
		UDim2.fromOffset(18, 15),
		UDim2.new(1, -36, 0, 25),
		Enum.Font.GothamBold,
		15,
		ACCENT
	)

	label(
		card,
		"✈ Cross-platform Fly\n👥 Player tools + Spectate\n📍 Waypoints + Compass\n⚡ Infinite Jump + Noclip\n🛡 Anti-AFK\n🎨 Themes + Accent Colors\n🔊 UI Sounds\n📱 Mobile optimized controls",
		UDim2.fromOffset(18, 48),
		UDim2.new(1, -36, 0, 210),
		Enum.Font.GothamMedium,
		14
	)
end

--========================================================
-- SPECTATE
--========================================================

local function stopSpectating()
	spectatingPlayer = nil
	Camera.CameraType = Enum.CameraType.Custom

	local humanoid = getHumanoid()

	if humanoid then
		Camera.CameraSubject = humanoid
	end
end

local function spectate(player)
	if not player or player == LocalPlayer then
		notify("Choose another player")
		return
	end

	local humanoid = player.Character
		and player.Character:FindFirstChildOfClass("Humanoid")

	if humanoid then
		spectatingPlayer = player
		Camera.CameraType = Enum.CameraType.Custom
		Camera.CameraSubject = humanoid
		notify("Spectating " .. player.DisplayName)
	end
end

--========================================================
-- PLAYERS
--========================================================

local function showPlayers()
	clearContent()

	label(
		Content,
		"Players",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	local search = textbox(
		Content,
		"Search player...",
		UDim2.fromOffset(5, 43),
		UDim2.new(1, -115, 0, 38)
	)

	search.Text = playerSearch

	local stopSpec = button(
		Content,
		"STOP",
		UDim2.new(1, -100, 0, 43),
		UDim2.fromOffset(95, 38),
		PANEL2()
	)

	stopSpec.MouseButton1Click:Connect(stopSpectating)

	local list = Instance.new("ScrollingFrame")
	list.Position = UDim2.fromOffset(5, 91)
	list.Size = UDim2.new(1, -10, 1, -96)
	list.BackgroundTransparency = 1
	list.BorderSizePixel = 0
	list.ScrollBarThickness = 4
	list.AutomaticCanvasSize = Enum.AutomaticSize.Y
	list.Parent = Content

	local layout = Instance.new("UIListLayout")
	layout.Padding = UDim.new(0, 7)
	layout.Parent = list

	local function populate()
		for _, child in ipairs(list:GetChildren()) do
			if child:IsA("Frame") then
				child:Destroy()
			end
		end

		local query = string.lower(search.Text)
		local count = 0

		for _, player in ipairs(Players:GetPlayers()) do
			local dn = string.lower(player.DisplayName)
			local un = string.lower(player.Name)

			if query == ""
				or string.find(dn, query, 1, true)
				or string.find(un, query, 1, true) then

				count += 1

				local card = Instance.new("Frame")
				card.Size = UDim2.new(1, -5, 0, 70)
				card.BackgroundColor3 = PANEL()
				card.BorderSizePixel = 0
				card.Parent = list

				Instance.new("UICorner", card).CornerRadius = UDim.new(0, 9)

				local avatar = Instance.new("ImageLabel")
				avatar.Position = UDim2.fromOffset(8, 9)
				avatar.Size = UDim2.fromOffset(52, 52)
				avatar.BackgroundColor3 = PANEL2()
				avatar.BorderSizePixel = 0
				avatar.Parent = card

				Instance.new("UICorner", avatar).CornerRadius = UDim.new(1, 0)

				task.spawn(function()
					local ok, image = pcall(function()
						return Players:GetUserThumbnailAsync(
							player.UserId,
							Enum.ThumbnailType.HeadShot,
							Enum.ThumbnailSize.Size100x100
						)
					end)

					if ok and avatar.Parent then
						avatar.Image = image
					end
				end)

				label(
					card,
					player.DisplayName,
					UDim2.fromOffset(70, 8),
					UDim2.new(1, -330, 0, 23),
					Enum.Font.GothamBold,
					14
				)

				label(
					card,
					"@" .. player.Name,
					UDim2.fromOffset(70, 34),
					UDim2.new(1, -330, 0, 20),
					Enum.Font.Gotham,
					11,
					MUTED()
				)

				local tp = button(
					card,
					"TELEPORT",
					UDim2.new(1, -255, 0.5, -18),
					UDim2.fromOffset(78, 36),
					PANEL2()
				)

				tp.MouseButton1Click:Connect(function()
					local root = getRoot()
					local targetRoot = player.Character
						and player.Character:FindFirstChild("HumanoidRootPart")

					if root and targetRoot then
						root.CFrame =
							targetRoot.CFrame + Vector3.new(0, 3, 0)

						notify("Teleported to " .. player.DisplayName)
					end
				end)

				local spec = button(
					card,
					"SPECTATE",
					UDim2.new(1, -170, 0.5, -18),
					UDim2.fromOffset(80, 36),
					PANEL2()
				)

				spec.MouseButton1Click:Connect(function()
					spectate(player)
				end)

				local bring = button(
					card,
					"TP",
					UDim2.new(1, -80, 0.5, -18),
					UDim2.fromOffset(70, 36),
					PANEL2()
				)

				bring.MouseButton1Click:Connect(function()
					local myRoot = getRoot()
					local targetRoot = player.Character
						and player.Character:FindFirstChild("HumanoidRootPart")

					if myRoot and targetRoot then
						targetRoot.CFrame =
							myRoot.CFrame + Vector3.new(0, 3, 0)

						notify("Teleported " .. player.DisplayName .. " to you")
					end
				end)
			end
		end

		if count == 0 then
			label(
				list,
				"No players found.",
				UDim2.fromOffset(10, 10),
				UDim2.new(1, -20, 0, 40),
				Enum.Font.GothamMedium,
				14,
				MUTED()
			)
		end
	end

	search:GetPropertyChangedSignal("Text"):Connect(function()
		playerSearch = search.Text
		populate()
	end)

	populate()
end

--========================================================
-- FLY
--========================================================

local flyVelocity
local flyGyro
local flyConnection

local function destroyMobileFlyControls()
	for _, obj in ipairs(GUI:GetChildren()) do
		if obj.Name == "MobileFlyControls" then
			obj:Destroy()
		end
	end

	mobileMove = Vector2.zero
	mobileUp = false
	mobileDown = false
end

local function createMobileFlyControls()
	destroyMobileFlyControls()

	if not UserInputService.TouchEnabled then
		return
	end

	local holder = Instance.new("ScreenGui")
	holder.Name = "MobileFlyControls"
	holder.IgnoreGuiInset = true
	holder.ResetOnSpawn = false
	holder.DisplayOrder = 100
	holder.Parent = GUI

	local joystick = Instance.new("Frame")
	joystick.AnchorPoint = Vector2.new(0.5, 0.5)
	joystick.Position = UDim2.new(0, 105, 1, -115)
	joystick.Size = UDim2.fromOffset(120, 120)
	joystick.BackgroundColor3 = PANEL2()
	joystick.BackgroundTransparency = 0.15
	joystick.BorderSizePixel = 0
	joystick.Parent = holder

	Instance.new("UICorner", joystick).CornerRadius = UDim.new(1, 0)

	local stroke = Instance.new("UIStroke")
	stroke.Color = ACCENT
	stroke.Thickness = 2
	stroke.Transparency = 0.25
	stroke.Parent = joystick

	local knob = Instance.new("Frame")
	knob.AnchorPoint = Vector2.new(0.5, 0.5)
	knob.Position = UDim2.fromScale(0.5, 0.5)
	knob.Size = UDim2.fromOffset(55, 55)
	knob.BackgroundColor3 = ACCENT
	knob.BorderSizePixel = 0
	knob.Parent = joystick

	Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

	local activeTouch

	local function updateJoystick(input)
		local center =
			joystick.AbsolutePosition
			+ joystick.AbsoluteSize / 2

		local offset = Vector2.new(
			input.Position.X - center.X,
			input.Position.Y - center.Y
		)

		local maxDistance = 45

		if offset.Magnitude > maxDistance then
			offset = offset.Unit * maxDistance
		end

		knob.Position = UDim2.new(
			0.5,
			offset.X,
			0.5,
			offset.Y
		)

		mobileMove = Vector2.new(
			offset.X / maxDistance,
			offset.Y / maxDistance
		)
	end

	joystick.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch
			or input.UserInputType == Enum.UserInputType.MouseButton1 then

			activeTouch = input
			updateJoystick(input)
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == activeTouch then
			updateJoystick(input)
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input == activeTouch then
			activeTouch = nil
			mobileMove = Vector2.zero

			if knob.Parent then
				knob.Position = UDim2.fromScale(0.5, 0.5)
			end
		end
	end)

	local up = button(
		holder,
		"▲",
		UDim2.new(1, -95, 1, -145),
		UDim2.fromOffset(70, 55),
		PANEL2()
	)

	local down = button(
		holder,
		"▼",
		UDim2.new(1, -95, 1, -78),
		UDim2.fromOffset(70, 55),
		PANEL2()
	)

	up.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			mobileUp = true
		end
	end)

	up.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			mobileUp = false
		end
	end)

	down.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			mobileDown = true
		end
	end)

	down.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			mobileDown = false
		end
	end)
end

local function getFlyDirection()
	local d = Vector3.zero

	if keys.W then
		d += Vector3.new(0, 0, -1)
	end

	if keys.S then
		d += Vector3.new(0, 0, 1)
	end

	if keys.A then
		d += Vector3.new(-1, 0, 0)
	end

	if keys.D then
		d += Vector3.new(1, 0, 0)
	end

	if mobileMove.Magnitude > 0.05 then
		d += Vector3.new(
			mobileMove.X,
			0,
			-mobileMove.Y
		)
	end

	if keys.Up or mobileUp then
		d += Vector3.new(0, 1, 0)
	end

	if keys.Down or mobileDown then
		d += Vector3.new(0, -1, 0)
	end

	if d.Magnitude > 1 then
		d = d.Unit
	end

	return d
end

local function stopFlying()
	flying = false

	if flyConnection then
		flyConnection:Disconnect()
		flyConnection = nil
	end

	if flyVelocity then
		flyVelocity:Destroy()
		flyVelocity = nil
	end

	if flyGyro then
		flyGyro:Destroy()
		flyGyro = nil
	end

	destroyMobileFlyControls()

	local humanoid = getHumanoid()

	if humanoid then
		humanoid.AutoRotate = true
	end
end

local function startFlying()
	local root = getRoot()
	local humanoid = getHumanoid()

	if not root or not humanoid then
		notify("Character not ready")
		return
	end

	stopFlying()

	flying = true

	humanoid.AutoRotate = false

	flyVelocity = Instance.new("BodyVelocity")
	flyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
	flyVelocity.P = 25000
	flyVelocity.Velocity = Vector3.zero
	flyVelocity.Parent = root

	flyGyro = Instance.new("BodyGyro")
	flyGyro.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
	flyGyro.P = 30000
	flyGyro.D = 1000
	flyGyro.Parent = root

	flyConnection = RunService.RenderStepped:Connect(function()
		if not flying or not root.Parent then
			return
		end

		local d = getFlyDirection()
		local cam = Camera.CFrame

		local forward = Vector3.new(
			cam.LookVector.X,
			0,
			cam.LookVector.Z
		)

		local right = Vector3.new(
			cam.RightVector.X,
			0,
			cam.RightVector.Z
		)

		if forward.Magnitude > 0 then
			forward = forward.Unit
		end

		if right.Magnitude > 0 then
			right = right.Unit
		end

		local world =
			forward * -d.Z
			+ right * d.X
			+ Vector3.new(0, d.Y, 0)

		if world.Magnitude > 1 then
			world = world.Unit
		end

		local target = world * flySpeed

		flyVelocity.Velocity =
			flyVelocity.Velocity:Lerp(target, 0.18)

		local look = Vector3.new(
			cam.LookVector.X,
			0,
			cam.LookVector.Z
		)

		if look.Magnitude > 0.05 then
			flyGyro.CFrame = CFrame.lookAt(
				root.Position,
				root.Position + look.Unit
			)
		end
	end)

	createMobileFlyControls()

	notify("Fly enabled")
end

--========================================================
-- FLY PAGE
--========================================================

local function showFly()
	clearContent()

	label(
		Content,
		"Fly",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	label(
		Content,
		"PC + mobile flight",
		UDim2.fromOffset(5, 35),
		UDim2.new(1, -10, 0, 25),
		Enum.Font.Gotham,
		13,
		MUTED()
	)

	local toggle = button(
		Content,
		flying and "DISABLE FLY" or "ENABLE FLY",
		UDim2.fromOffset(5, 75),
		UDim2.new(1, -10, 0, 45),
		flying and ACCENT or PANEL2()
	)

	toggle.MouseButton1Click:Connect(function()
		if flying then
			stopFlying()
		else
			startFlying()
		end

		showFly()
	end)

	label(
		Content,
		"Fly Speed",
		UDim2.fromOffset(5, 135),
		UDim2.new(1, -10, 0, 25),
		Enum.Font.GothamMedium,
		14
	)

	local speed = textbox(
		Content,
		"Speed...",
		UDim2.fromOffset(5, 165),
		UDim2.new(1, -10, 0, 40)
	)

	speed.Text = tostring(flySpeed)

	local apply = button(
		Content,
		"APPLY SPEED",
		UDim2.fromOffset(5, 215),
		UDim2.new(1, -10, 0, 40),
		PANEL2()
	)

	apply.MouseButton1Click:Connect(function()
		local n = tonumber(speed.Text)

		if n then
			flySpeed = math.clamp(n, 1, 500)
			notify("Fly speed: " .. flySpeed)
		else
			notify("Invalid speed")
		end
	end)

	local info = Instance.new("Frame")
	info.Position = UDim2.fromOffset(5, 270)
	info.Size = UDim2.new(1, -10, 0, 130)
	info.BackgroundColor3 = PANEL()
	info.BorderSizePixel = 0
	info.Parent = Content

	Instance.new("UICorner", info).CornerRadius = UDim.new(0, 10)

	label(
		info,
		"PC",
		UDim2.fromOffset(15, 10),
		UDim2.fromOffset(80, 20),
		Enum.Font.GothamBold,
		13,
		ACCENT
	)

	label(
		info,
		"WASD • Space • Ctrl",
		UDim2.fromOffset(15, 34),
		UDim2.new(1, -30, 0, 22),
		Enum.Font.Gotham,
		12
	)

	label(
		info,
		"MOBILE",
		UDim2.fromOffset(15, 66),
		UDim2.fromOffset(80, 20),
		Enum.Font.GothamBold,
		13,
		ACCENT
	)

	label(
		info,
		"Joystick • ▲ • ▼",
		UDim2.fromOffset(15, 90),
		UDim2.new(1, -30, 0, 22),
		Enum.Font.Gotham,
		12
	)
end

--========================================================
-- MOVEMENT
--========================================================

local function applyMovement()
	local humanoid = getHumanoid()

	if humanoid then
		humanoid.WalkSpeed = walkSpeed
		humanoid.UseJumpPower = true
		humanoid.JumpPower = jumpPower
	end
end

local function showMovement()
	clearContent()

	label(
		Content,
		"Movement",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	local ws = textbox(
		Content,
		"WalkSpeed",
		UDim2.fromOffset(5, 55),
		UDim2.new(1, -10, 0, 40)
	)

	ws.Text = tostring(walkSpeed)

	local jp = textbox(
		Content,
		"JumpPower",
		UDim2.fromOffset(5, 105),
		UDim2.new(1, -10, 0, 40)
	)

	jp.Text = tostring(jumpPower)

	local apply = button(
		Content,
		"APPLY MOVEMENT",
		UDim2.fromOffset(5, 155),
		UDim2.new(1, -10, 0, 42),
		PANEL2()
	)

	apply.MouseButton1Click:Connect(function()
		local a = tonumber(ws.Text)
		local b = tonumber(jp.Text)

		if a then
			walkSpeed = math.clamp(a, 1, 500)
		end

		if b then
			jumpPower = math.clamp(b, 1, 500)
		end

		applyMovement()
		notify("Movement applied")
	end)

	local infinite = button(
		Content,
		"INFINITE JUMP: " .. (infiniteJump and "ON" or "OFF"),
		UDim2.fromOffset(5, 210),
		UDim2.new(1, -10, 0, 42),
		infiniteJump and ACCENT or PANEL2()
	)

	infinite.MouseButton1Click:Connect(function()
		infiniteJump = not infiniteJump
		showMovement()
	end)

	local noClipButton = button(
		Content,
		"NOCLIP: " .. (noclip and "ON" or "OFF"),
		UDim2.fromOffset(5, 262),
		UDim2.new(1, -10, 0, 42),
		noclip and ACCENT or PANEL2()
	)

	noClipButton.MouseButton1Click:Connect(function()
		noclip = not noclip

		if not noclip then
			local char = getCharacter()

			if char then
				for _, part in ipairs(char:GetDescendants()) do
					if part:IsA("BasePart") then
						part.CanCollide = true
					end
				end
			end
		end

		showMovement()
	end)
end

--========================================================
-- WAYPOINT COMPASS
--========================================================

local compassGui
local compassConnection

local function stopCompass()
	if compassConnection then
		compassConnection:Disconnect()
		compassConnection = nil
	end

	if compassGui then
		compassGui:Destroy()
		compassGui = nil
	end
end

local function startCompass()
	stopCompass()

	if next(waypoints) == nil then
		notify("Create a waypoint first")
		return
	end

	compassGui = Instance.new("ScreenGui")
	compassGui.Name = "WaypointCompass"
	compassGui.ResetOnSpawn = false
	compassGui.IgnoreGuiInset = true
	compassGui.DisplayOrder = 50
	compassGui.Parent = GUI

	local frame = Instance.new("Frame")
	frame.AnchorPoint = Vector2.new(0.5, 0)
	frame.Position = UDim2.new(0.5, 0, 0, 20)
	frame.Size = UDim2.fromOffset(360, 42)
	frame.BackgroundColor3 = PANEL2()
	frame.BackgroundTransparency = 0.1
	frame.BorderSizePixel = 0
	frame.Parent = compassGui

	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

	local text = Instance.new("TextLabel")
	text.BackgroundTransparency = 1
	text.Size = UDim2.fromScale(1, 1)
	text.TextColor3 = TEXT()
	text.TextSize = 13
	text.Font = Enum.Font.GothamBold
	text.Parent = frame

	compassConnection = RunService.RenderStepped:Connect(function()
		local root = getRoot()

		if not root then
			return
		end

		local nearestName
		local nearestDistance

		for name, data in pairs(waypoints) do
			local distance = (root.Position - data.Position).Magnitude

			if not nearestDistance or distance < nearestDistance then
				nearestDistance = distance
				nearestName = name
			end
		end

		if nearestName then
			text.Text = string.format(
				"📍 %s  •  %d studs",
				nearestName,
				math.floor(nearestDistance)
			)
		end
	end)

	notify("Waypoint compass enabled")
end

--========================================================
-- WAYPOINTS
--========================================================

local function showWaypoints()
	clearContent()

	label(
		Content,
		"Waypoints",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	local nameBox = textbox(
		Content,
		"Waypoint name...",
		UDim2.fromOffset(5, 48),
		UDim2.new(1, -120, 0, 38)
	)

	local create = button(
		Content,
		"CREATE",
		UDim2.new(1, -105, 0, 48),
		UDim2.fromOffset(100, 38),
		ACCENT
	)

	create.MouseButton1Click:Connect(function()
		local root = getRoot()
		local name = nameBox.Text

		if not root then
			notify("Character not ready")
			return
		end

		if name == "" then
			notify("Enter a name")
			return
		end

		if waypoints[name] then
			notify("Waypoint already exists")
			return
		end

		waypoints[name] = {
			CFrame = root.CFrame,
			Position = root.Position
		}

		notify("Created " .. name)
		showWaypoints()
	end)

	local compass = button(
		Content,
		"COMPASS",
		UDim2.fromOffset(5, 95),
		UDim2.new(0.48, -5, 0, 38),
		PANEL2()
	)

	compass.MouseButton1Click:Connect(function()
		if compassGui then
			stopCompass()
			notify("Compass disabled")
		else
			startCompass()
		end
	end)

	local save = button(
		Content,
		"SAVE SET",
		UDim2.new(0.52, 0, 0, 95),
		UDim2.new(0.48, -5, 0, 38),
		PANEL2()
	)

	save.MouseButton1Click:Connect(function()
		local set = {}

		for name, data in pairs(waypoints) do
			set[name] = {
				CFrame = data.CFrame,
				Position = data.Position
			}
		end

		table.insert(savedWaypointSets, set)

		notify("Waypoint set saved")
	end)

	local load = button(
		Content,
		"LOAD SET",
		UDim2.fromOffset(5, 140),
		UDim2.new(0.48, -5, 0, 38),
		PANEL2()
	)

	load.MouseButton1Click:Connect(function()
		local latest = savedWaypointSets[#savedWaypointSets]

		if not latest then
			notify("No saved set")
			return
		end

		waypoints = {}

		for name, data in pairs(latest) do
			waypoints[name] = {
				CFrame = data.CFrame,
				Position = data.Position
			}
		end

		notify("Waypoint set loaded")
		showWaypoints()
	end)

	local list = Instance.new("ScrollingFrame")
	list.Position = UDim2.fromOffset(5, 190)
	list.Size = UDim2.new(1, -10, 1, -195)
	list.BackgroundTransparency = 1
	list.BorderSizePixel = 0
	list.ScrollBarThickness = 4
	list.AutomaticCanvasSize = Enum.AutomaticSize.Y
	list.Parent = Content

	local layout = Instance.new("UIListLayout")
	layout.Padding = UDim.new(0, 7)
	layout.Parent = list

	local count = 0

	for name, data in pairs(waypoints) do
		count += 1

		local row = Instance.new("Frame")
		row.Size = UDim2.new(1, -5, 0, 64)
		row.BackgroundColor3 = PANEL()
		row.BorderSizePixel = 0
		row.Parent = list

		Instance.new("UICorner", row).CornerRadius = UDim.new(0, 9)

		label(
			row,
			name,
			UDim2.fromOffset(10, 6),
			UDim2.new(1, -230, 0, 22),
			Enum.Font.GothamBold,
			13
		)

		label(
			row,
			string.format(
				"%.1f, %.1f, %.1f",
				data.Position.X,
				data.Position.Y,
				data.Position.Z
			),
			UDim2.fromOffset(10, 31),
			UDim2.new(1, -230, 0, 18),
			Enum.Font.Gotham,
			10,
			MUTED()
		)

		local tp = button(
			row,
			"TP",
			UDim2.new(1, -210, 0.5, -17),
			UDim2.fromOffset(55, 34),
			PANEL2()
		)

		tp.MouseButton1Click:Connect(function()
			local root = getRoot()

			if root then
				root.CFrame =
					data.CFrame + Vector3.new(0, 3, 0)

				notify("Teleported to " .. name)
			end
		end)

		local del = button(
			row,
			"DELETE",
			UDim2.new(1, -145, 0.5, -17),
			UDim2.fromOffset(90, 34),
			PANEL2()
		)

		del.MouseButton1Click:Connect(function()
			waypoints[name] = nil
			showWaypoints()
		end)
	end

	if count == 0 then
		label(
			list,
			"No waypoints yet.",
			UDim2.fromOffset(10, 10),
			UDim2.new(1, -20, 0, 40),
			Enum.Font.GothamMedium,
			14,
			MUTED()
		)
	end
end

--========================================================
-- UTILITIES
--========================================================

local function showUtilities()
	clearContent()

	label(
		Content,
		"Utilities",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	label(
		Content,
		"Extra tools",
		UDim2.fromOffset(5, 36),
		UDim2.new(1, -10, 0, 25),
		Enum.Font.Gotham,
		13,
		MUTED()
	)

	local anti = button(
		Content,
		"ANTI-AFK: " .. (antiAFK and "ON" or "OFF"),
		UDim2.fromOffset(5, 75),
		UDim2.new(1, -10, 0, 42),
		antiAFK and ACCENT or PANEL2()
	)

	anti.MouseButton1Click:Connect(function()
		antiAFK = not antiAFK
		showUtilities()
	end)

	local stopSpec = button(
		Content,
		"STOP SPECTATING",
		UDim2.fromOffset(5, 125),
		UDim2.new(1, -10, 0, 42),
		PANEL2()
	)

	stopSpec.MouseButton1Click:Connect(function()
		stopSpectating()
	end)

	local stopCompassButton = button(
		Content,
		"STOP WAYPOINT COMPASS",
		UDim2.fromOffset(5, 175),
		UDim2.new(1, -10, 0, 42),
		PANEL2()
	)

	stopCompassButton.MouseButton1Click:Connect(function()
		stopCompass()
	end)

	local resetMovement = button(
		Content,
		"RESET MOVEMENT",
		UDim2.fromOffset(5, 225),
		UDim2.new(1, -10, 0, 42),
		PANEL2()
	)

	resetMovement.MouseButton1Click:Connect(function()
		walkSpeed = 16
		jumpPower = 50
		applyMovement()
		notify("Movement reset")
	end)

	local stopAll = button(
		Content,
		"STOP ALL SPECIAL MODES",
		UDim2.fromOffset(5, 275),
		UDim2.new(1, -10, 0, 42),
		PANEL2()
	)

	stopAll.MouseButton1Click:Connect(function()
		stopFlying()
		stopCompass()
		stopSpectating()

		noclip = false
		infiniteJump = false

		local char = getCharacter()

		if char then
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.CanCollide = true
				end
			end
		end

		notify("All special modes stopped")
	end)
end

--========================================================
-- SETTINGS
--========================================================

local function showSettings()
	clearContent()

	label(
		Content,
		"Settings",
		UDim2.fromOffset(5, 0),
		UDim2.new(1, -10, 0, 38),
		Enum.Font.GothamBold,
		23
	)

	label(
		Content,
		"Customize Piet Script Hub",
		UDim2.fromOffset(5, 36),
		UDim2.new(1, -10, 0, 25),
		Enum.Font.Gotham,
		13,
		MUTED()
	)

	-- Theme
	label(
		Content,
		"Theme",
		UDim2.fromOffset(5, 78),
		UDim2.new(1, -10, 0, 22),
		Enum.Font.GothamBold,
		14
	)

	local dark = button(
		Content,
		"DARK",
		UDim2.fromOffset(5, 108),
		UDim2.fromOffset(100, 38),
		themeName == "Dark" and ACCENT or PANEL2()
	)

	dark.MouseButton1Click:Connect(function()
		themeName = "Dark"
		showSettings()
	end)

	local light = button(
		Content,
		"LIGHT",
		UDim2.fromOffset(115, 108),
		UDim2.fromOffset(100, 38),
		themeName == "Light" and ACCENT or PANEL2()
	)

	light.MouseButton1Click:Connect(function()
		themeName = "Light"
		showSettings()
	end)

	-- Accent
	label(
		Content,
		"Accent Color",
		UDim2.fromOffset(5, 158),
		UDim2.new(1, -10, 0, 22),
		Enum.Font.GothamBold,
		14
	)

	local names = {
		"Blue",
		"Purple",
		"Cyan",
		"Green",
		"Red",
		"Orange",
		"Pink"
	}

	for i, name in ipairs(names) do
		local col = (i - 1) % 4
		local row = math.floor((i - 1) / 4)

		local b = button(
			Content,
			name,
			UDim2.fromOffset(
				5 + col * 90,
				188 + row * 44
			),
			UDim2.fromOffset(82, 36),
			accentName == name and ACCENTS[name] or PANEL2()
		)

		b.MouseButton1Click:Connect(function()
			accentName = name
			ACCENT = ACCENTS[name]

			MainStroke.Color = ACCENT

			showSettings()
		end)
	end

	-- Sounds
	local soundY = 282

	local sounds = button(
		Content,
		"UI SOUNDS: " .. (soundsEnabled and "ON" or "OFF"),
		UDim2.fromOffset(5, soundY),
		UDim2.new(1, -10, 0, 40),
		soundsEnabled and ACCENT or PANEL2()
	)

	sounds.MouseButton1Click:Connect(function()
		soundsEnabled = not soundsEnabled
		showSettings()
	end)

	-- Animations
	local anim = button(
		Content,
		"ANIMATIONS: " .. (animationsEnabled and "ON" or "OFF"),
		UDim2.fromOffset(5, soundY + 48),
		UDim2.new(1, -10, 0, 40),
		animationsEnabled and ACCENT or PANEL2()
	)

	anim.MouseButton1Click:Connect(function()
		animationsEnabled = not animationsEnabled
		showSettings()
	end)

	-- Unexecute
	local unexecute = button(
		Content,
		"UNEXECUTE SCRIPT",
		UDim2.fromOffset(5, soundY + 98),
		UDim2.new(1, -10, 0, 44),
		Color3.fromRGB(200, 55, 65)
	)

	unexecute.MouseEnter:Connect(function()
		TweenService:Create(
			unexecute,
			TweenInfo.new(0.1),
			{BackgroundColor3 = Color3.fromRGB(245, 70, 80)}
		):Play()
	end)

	unexecute.MouseButton1Click:Connect(function()
		guiDestroyed = true

		stopFlying()
		stopCompass()
		stopSpectating()

		if GUI then
			GUI:Destroy()
		end
	end)
end

--========================================================
-- PAGE SWITCHING
--========================================================

local function updateSidebar()
	for name, b in pairs(pageButtons) do
		if name == currentPage then
			b.BackgroundColor3 = ACCENT
			b.TextColor3 = TEXT()
		else
			b.BackgroundColor3 = PANEL()
			b.TextColor3 = MUTED()
		end
	end
end

local function showPage(page)
	if guiDestroyed then
		return
	end

	currentPage = page
	updateSidebar()

	if page == "Home" then
		showHome()
	elseif page == "Players" then
		showPlayers()
	elseif page == "Fly" then
		showFly()
	elseif page == "Movement" then
		showMovement()
	elseif page == "Waypoints" then
		showWaypoints()
	elseif page == "Utilities" then
		showUtilities()
	elseif page == "Settings" then
		showSettings()
	end
end

for name, b in pairs(pageButtons) do
	b.MouseButton1Click:Connect(function()
		showPage(name)
	end)
end

--========================================================
-- CLOSE / REOPEN
--========================================================

local Reopen = Instance.new("TextButton")
Reopen.Size = UDim2.fromOffset(120, 42)
Reopen.Position = UDim2.fromOffset(18, 18)
Reopen.BackgroundColor3 = ACCENT
Reopen.BorderSizePixel = 0
Reopen.Text = "PIET HUB"
Reopen.TextColor3 = TEXT()
Reopen.TextSize = 13
Reopen.Font = Enum.Font.GothamBold
Reopen.Visible = false
Reopen.Parent = GUI

Instance.new("UICorner", Reopen).CornerRadius = UDim.new(0, 9)

CloseButton.MouseButton1Click:Connect(function()
	Main.Visible = false
	Reopen.Visible = true
end)

Reopen.MouseButton1Click:Connect(function()
	Main.Visible = true
	Reopen.Visible = false
end)

--========================================================
-- DRAGGING
--========================================================

local dragging = false
local dragStart
local startPos

TopBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPos = Main.Position
	end
end)

TopBar.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (
		input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch
	) then

		local delta = input.Position - dragStart

		Main.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

--========================================================
-- KEYBOARD
--========================================================

UserInputService.InputBegan:Connect(function(input, processed)
	if processed then
		return
	end

	if input.KeyCode == Enum.KeyCode.W then
		keys.W = true
	elseif input.KeyCode == Enum.KeyCode.A then
		keys.A = true
	elseif input.KeyCode == Enum.KeyCode.S then
		keys.S = true
	elseif input.KeyCode == Enum.KeyCode.D then
		keys.D = true
	elseif input.KeyCode == Enum.KeyCode.Space then
		keys.Up = true
	elseif input.KeyCode == Enum.KeyCode.LeftControl
		or input.KeyCode == Enum.KeyCode.RightControl then
		keys.Down = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then
		keys.W = false
	elseif input.KeyCode == Enum.KeyCode.A then
		keys.A = false
	elseif input.KeyCode == Enum.KeyCode.S then
		keys.S = false
	elseif input.KeyCode == Enum.KeyCode.D then
		keys.D = false
	elseif input.KeyCode == Enum.KeyCode.Space then
		keys.Up = false
	elseif input.KeyCode == Enum.KeyCode.LeftControl
		or input.KeyCode == Enum.KeyCode.RightControl then
		keys.Down = false
	end
end)

--========================================================
-- INFINITE JUMP
--========================================================

UserInputService.JumpRequest:Connect(function()
	if infiniteJump then
		local humanoid = getHumanoid()

		if humanoid then
			humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

--========================================================
-- NOCLIP
--========================================================

RunService.Stepped:Connect(function()
	if noclip then
		local char = getCharacter()

		if char then
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.CanCollide = false
				end
			end
		end
	end
end)

--========================================================
-- ANTI-AFK
--========================================================

LocalPlayer.Idled:Connect(function()
	if antiAFK then
		VirtualUser:CaptureController()
		VirtualUser:ClickButton2(Vector2.new())
	end
end)

--========================================================
-- RESPAWN
--========================================================

LocalPlayer.CharacterAdded:Connect(function()
	stopFlying()

	task.wait(0.6)

	applyMovement()

	if spectatingPlayer then
		stopSpectating()
	end

	if currentPage == "Players" then
		showPlayers()
	end
end)

--========================================================
-- PLAYER EVENTS
--========================================================

Players.PlayerAdded:Connect(function()
	if currentPage == "Players" then
		task.wait(0.1)
		showPlayers()
	end
end)

Players.PlayerRemoving:Connect(function(player)
	if spectatingPlayer == player then
		stopSpectating()
	end

	if currentPage == "Players" then
		task.wait(0.05)
		showPlayers()
	end
end)

--========================================================
-- INITIALIZE
--========================================================

applyMovement()
showPage("Home")

Main.Size = UDim2.fromOffset(0, 0)
Main.Position = UDim2.new(0.5, 0, 0.5, 0)

if animationsEnabled then
	TweenService:Create(
		Main,
		TweenInfo.new(
			0.35,
			Enum.EasingStyle.Back,
			Enum.EasingDirection.Out
		),
		{
			Size = UDim2.fromOffset(780, 520),
			Position = UDim2.new(0.5, -390, 0.5, -260)
		}
	):Play()
else
	Main.Size = UDim2.fromOffset(780, 520)
	Main.Position = UDim2.new(0.5, -390, 0.5, -260)
end

task.delay(0.5, function()
	notify("Piet Script Hub " .. VERSION .. " loaded")
end)
