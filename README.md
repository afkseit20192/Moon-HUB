
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()


local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")


local ESP_ENABLED = false
local NPC_ESP = false
local AIMBOT_ENABLED = false

local COLORS = {
	Enemy = Color3.fromRGB(255, 60, 60),
	Team = Color3.fromRGB(60, 255, 60)
}


local Window = Rayfield:CreateWindow({
	Name = "Moon Hub V1 I EH",
	Theme = "Dark",
	LoadingTitle = "Loading...",
	LoadingSubtitle = "Rayfield UI",
	ConfigurationSaving = {
		Enabled = true,
		FolderName = "StudioHub",
		FileName = "Config"
	}
})


local function isTeam(plr)
	return LocalPlayer.Team and plr.Team and LocalPlayer.Team == plr.Team
end

local function createESP(model, nameText, teamCheck)
	local hrp = model:FindFirstChild("HumanoidRootPart")
	local hum = model:FindFirstChildOfClass("Humanoid")
	if not hrp or not hum or hrp:FindFirstChild("ESP") then return end

	local gui = Instance.new("BillboardGui", hrp)
	gui.Name = "ESP"
	gui.Size = UDim2.new(4,0,5,0)
	gui.AlwaysOnTop = true

	local box = Instance.new("Frame", gui)
	box.Size = UDim2.new(1,0,1,0)
	box.BackgroundTransparency = 1
	box.BorderSizePixel = 2

	local hpBack = Instance.new("Frame", gui)
	hpBack.Position = UDim2.new(-0.08,0,0,0)
	hpBack.Size = UDim2.new(0.05,0,1,0)
	hpBack.BackgroundColor3 = Color3.fromRGB(40,40,40)
	hpBack.BorderSizePixel = 0

	local hpBar = Instance.new("Frame", hpBack)
	hpBar.AnchorPoint = Vector2.new(0,1)
	hpBar.Position = UDim2.new(0,0,1,0)
	hpBar.BorderSizePixel = 0

	RunService.RenderStepped:Connect(function()
		if not ESP_ENABLED then gui.Enabled = false return else gui.Enabled = true end
		if not model.Parent then gui:Destroy() end

		local color = COLORS.Enemy
		if teamCheck and isTeam(model) then
			color = COLORS.Team
		end

		box.BorderColor3 = color
		hpBar.BackgroundColor3 = Color3.fromRGB(0,255,0)
		hpBar.Size = UDim2.new(1,0, hum.Health / hum.MaxHealth,0)
	end)
end


for _,p in pairs(Players:GetPlayers()) do
	if p ~= LocalPlayer then
		p.CharacterAdded:Connect(function(char)
			createESP(char, p.Name, true)
		end)
	end
end


workspace.ChildAdded:Connect(function(obj)
	if NPC_ESP and obj:IsA("Model") and obj:FindFirstChild("Humanoid") then
		createESP(obj, "NPC", false)
	end
end)


local ESPTab = Window:CreateTab("👁️ ESP")
local CarTab = Window:CreateTab("🚘 Car Mods")
local MiscTab = Window:CreateTab("⚙️ Misc")


ESPTab:CreateToggle({
	Name = "Enable ESP",
	CurrentValue = false,
	Callback = function(v) ESP_ENABLED = v end
})

ESPTab:CreateToggle({
	Name = "NPC ESP",
	CurrentValue = false,
	Callback = function(v) NPC_ESP = v end
})

ESPTab:CreateColorPicker({
	Name = "Enemy Color",
	Color = COLORS.Enemy,
	Callback = function(c) COLORS.Enemy = c end
})

ESPTab:CreateColorPicker({
	Name = "Team Color",
	Color = COLORS.Team,
	Callback = function(c) COLORS.Team = c end
})


ESPTab:CreateKeybind({
	Name = "Toggle ESP",
	CurrentKeybind = "F",
	Callback = function()
		ESP_ENABLED = not ESP_ENABLED
	end
})


CarTab:CreateButton({
	Name = "Car Fly",
	Callback = function()
		local seat = LocalPlayer.Character and LocalPlayer.Character.Humanoid.SeatPart
		if seat then
			seat.AssemblyLinearVelocity = Vector3.new(0,100,0)
		end
	end
})

CarTab:CreateButton({
	Name = "Drift",
	Callback = function()
		local seat = LocalPlayer.Character.Humanoid.SeatPart
		if seat then
			seat.AssemblyAngularVelocity = Vector3.new(0,20,0)
		end
	end
})

CarTab:CreateSlider({
	Name = "Suspension Height",
	Range = {0,10},
	Increment = 0.5,
	CurrentValue = 2,
	Callback = function(v)
		local seat = LocalPlayer.Character.Humanoid.SeatPart
		if seat then
			seat.CFrame *= CFrame.new(0,v,0)
		end
	end
})


MiscTab:CreateButton({
	Name = "Destroy UI",
	Callback = function()
		Rayfield:Destroy()
	end
})


local AIMBOT = {
	Enabled = false,
	FOV = 150,
	TeamCheck = true
}

local function getClosestTarget()
	local closest, dist = nil, AIMBOT.FOV
	for _,p in pairs(game.Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
			if AIMBOT.TeamCheck and LocalPlayer.Team == p.Team then continue end

			local head = p.Character.Head
			local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local mag = (Vector2.new(pos.X,pos.Y) -
					Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
				if mag < dist then
					dist = mag
					closest = head
				end
			end
		end
	end
	return closest
end

RunService.RenderStepped:Connect(function()
	if AIMBOT.Enabled then
		local target = getClosestTarget()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end
end)

local AimbotTab = Window:CreateTab("Aimbot")

AimbotTab:CreateToggle({
	Name = "Enable Aimbot",
	Callback = function(v) AIMBOT.Enabled = v end
})

AimbotTab:CreateToggle({
	Name = "Team Check",
	CurrentValue = true,
	Callback = function(v) AIMBOT.TeamCheck = v end
})

AimbotTab:CreateSlider({
	Name = "FOV",
	Range = {50,400},
	Increment = 10,
	CurrentValue = 150,
	Callback = function(v) AIMBOT.FOV = v end
})

AimbotTab:CreateKeybind({
	Name = "Toggle Aimbot",
	CurrentKeybind = "Q",
	Callback = function()
		AIMBOT.Enabled = not AIMBOT.Enabled
	end
})

local function createSkeleton(character)
	local bones = {
		{"Head","UpperTorso"},
		{"UpperTorso","LowerTorso"},
		{"UpperTorso","LeftUpperArm"},
		{"LeftUpperArm","LeftLowerArm"},
		{"UpperTorso","RightUpperArm"},
		{"RightUpperArm","RightLowerArm"},
		{"LowerTorso","LeftUpperLeg"},
		{"LeftUpperLeg","LeftLowerLeg"},
		{"LowerTorso","RightUpperLeg"},
		{"RightUpperLeg","RightLowerLeg"}
	}

	for _,pair in pairs(bones) do
		local a = character:FindFirstChild(pair[1])
		local b = character:FindFirstChild(pair[2])
		if a and b then
			local beam = Instance.new("Beam")
			beam.Attachment0 = Instance.new("Attachment", a)
			beam.Attachment1 = Instance.new("Attachment", b)
			beam.Width0 = 0.1
			beam.Width1 = 0.1
			beam.Color = ColorSequence.new(Color3.new(1,1,1))
			beam.Parent = a
		end
	end
end

local PRESETS = {
	FPS = {
		ESP = true,
		Aimbot = true,
		FOV = 120
	},
	Driving = {
		ESP = false,
		Aimbot = false,
		FOV = 70
	}
}

local PresetTab = Window:CreateTab("🧩 Presets")

PresetTab:CreateButton({
	Name = "FPS Preset",
	Callback = function()
		ESP_ENABLED = PRESETS.FPS.ESP
		AIMBOT.Enabled = PRESETS.FPS.Aimbot
		AIMBOT.FOV = PRESETS.FPS.FOV
	end
})

PresetTab:CreateButton({
	Name = "Driving Preset",
	Callback = function()
		ESP_ENABLED = PRESETS.Driving.ESP
		AIMBOT.Enabled = PRESETS.Driving.Aimbot
		AIMBOT.FOV = PRESETS.Driving.FOV
	end
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local function createTextESP(player)
	if player == LocalPlayer then return end

	local function onCharacter(char)
		local head = char:WaitForChild("Head", 5)
		local humanoid = char:WaitForChild("Humanoid", 5)
		if not head or not humanoid then return end
		if head:FindFirstChild("TextESP") then return end

		
		local gui = Instance.new("BillboardGui")
		gui.Name = "TextESP"
		gui.Adornee = head
		gui.Size = UDim2.new(0, 200, 0, 90)
		gui.StudsOffset = Vector3.new(0, 2.8, 0)
		gui.AlwaysOnTop = true
		gui.Parent = head

		
		local layout = Instance.new("UIListLayout", gui)
		layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
		layout.VerticalAlignment = Enum.VerticalAlignment.Center

		
		local nameLabel = Instance.new("TextLabel")
		nameLabel.Size = UDim2.new(1, 0, 0, 30)
		nameLabel.BackgroundTransparency = 1
		nameLabel.Text = player.Name
		nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
		nameLabel.TextStrokeTransparency = 0.3
		nameLabel.Font = Enum.Font.SourceSansBold
		nameLabel.TextScaled = true
		nameLabel.Parent = gui

		
		local teamLabel = Instance.new("TextLabel")
		teamLabel.Size = UDim2.new(1, 0, 0, 25)
		teamLabel.BackgroundTransparency = 1
		teamLabel.Text = player.Team and player.Team.Name or "Unknown"
		teamLabel.TextColor3 = Color3.fromRGB(90, 160, 255)
		teamLabel.TextStrokeTransparency = 0.3
		teamLabel.Font = Enum.Font.SourceSans
		teamLabel.TextScaled = true
		teamLabel.Parent = gui

		
		local hpLabel = Instance.new("TextLabel")
		hpLabel.Size = UDim2.new(1, 0, 0, 25)
		hpLabel.BackgroundTransparency = 1
		hpLabel.TextColor3 = Color3.fromRGB(80, 255, 80)
		hpLabel.TextStrokeTransparency = 0.3
		hpLabel.Font = Enum.Font.SourceSansBold
		hpLabel.TextScaled = true
		hpLabel.Parent = gui

		
		game:GetService("RunService").RenderStepped:Connect(function()
			if not char.Parent then
				gui:Destroy()
				return
			end
			hpLabel.Text = math.floor(humanoid.Health) .. " HP"
		end)
	end

	if player.Character then
		onCharacter(player.Character)
	end
	player.CharacterAdded:Connect(onCharacter)
end


for _, plr in pairs(Players:GetPlayers()) do
	createTextESP(plr)
end

Players.PlayerAdded:Connect(createTextESP)


local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer


local FLY_ENABLED = false
local FLY_SPEED = 80
local HEIGHT = 5


local input = {
	W = false,
	A = false,
	S = false,
	D = false,
	Up = false,
	Down = false
}


UIS.InputBegan:Connect(function(i,gp)
	if gp then return end
	if i.KeyCode == Enum.KeyCode.F then
		FLY_ENABLED = not FLY_ENABLED
	end
	if i.KeyCode == Enum.KeyCode.W then input.W = true end
	if i.KeyCode == Enum.KeyCode.S then input.S = true end
	if i.KeyCode == Enum.KeyCode.A then input.A = true end
	if i.KeyCode == Enum.KeyCode.D then input.D = true end
	if i.KeyCode == Enum.KeyCode.Space then input.Up = true end
	if i.KeyCode == Enum.KeyCode.LeftControl then input.Down = true end
end)

UIS.InputEnded:Connect(function(i)
	if i.KeyCode == Enum.KeyCode.W then input.W = false end
	if i.KeyCode == Enum.KeyCode.S then input.S = false end
	if i.KeyCode == Enum.KeyCode.A then input.A = false end
	if i.KeyCode == Enum.KeyCode.D then input.D = false end
	if i.KeyCode == Enum.KeyCode.Space then input.Up = false end
	if i.KeyCode == Enum.KeyCode.LeftControl then input.Down = false end
end)


RunService.RenderStepped:Connect(function(dt)
	if not FLY_ENABLED then return end

	local char = player.Character
	if not char or not char:FindFirstChild("Humanoid") then return end

	local seat = char.Humanoid.SeatPart
	if not seat or not seat:IsA("VehicleSeat") then return end

	local car = seat.AssemblyRootPart
	if not car then return end

	
	local moveDir = Vector3.zero
	if input.W then moveDir += car.CFrame.LookVector end
	if input.S then moveDir -= car.CFrame.LookVector end
	if input.A then moveDir -= car.CFrame.RightVector end
	if input.D then moveDir += car.CFrame.RightVector end

	
	if input.Up then HEIGHT += 0.5 end
	if input.Down then HEIGHT -= 0.5 end

	
	local targetPos = car.Position + moveDir * FLY_SPEED * dt
	targetPos = Vector3.new(
		targetPos.X,
		car.Position.Y + (HEIGHT - (car.Position.Y - seat.Position.Y)),
		targetPos.Z
	)

	
	local targetCF = CFrame.new(targetPos, targetPos + car.CFrame.LookVector)

	car.AssemblyLinearVelocity = Vector3.zero
	car.AssemblyAngularVelocity = Vector3.zero
	car.CFrame = car.CFrame:Lerp(targetCF, 0.15)
end)

local FlyKey = Enum.KeyCode.V
        local SpeedKey = Enum.KeyCode.LeftControl
        
        local SpeedKeyMultiplier = 3
        local FlightSpeed = 150
        local FlightAcceleration = 4
        local TurnSpeed = 16
        
        -- made by MoonHub
        
        -- enjoy :3
        
        local UserInputService = game:GetService("UserInputService")
        local StarterGui = game:GetService("StarterGui")
        local RunService = game:GetService("RunService")
        local Players = game:GetService("Players")
        local User = Players.LocalPlayer
        local Camera = workspace.CurrentCamera
        local UserCharacter = nil
        local UserRootPart = nil
        local Connection = nil
        
        
        workspace.Changed:Connect(function()
            Camera = workspace.CurrentCamera
        end)
        
        local setCharacter = function(c)
            UserCharacter = c
            UserRootPart = c:WaitForChild("HumanoidRootPart")
        end
        
        User.CharacterAdded:Connect(setCharacter)
        if User.Character then
            setCharacter(User.Character)
        end
        
        local CurrentVelocity = Vector3.new(0,0,0)
        local Flight = function(delta)
            local BaseVelocity = Vector3.new(0,0,0)
            if not UserInputService:GetFocusedTextBox() then
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    BaseVelocity = BaseVelocity + (Camera.CFrame.LookVector * FlightSpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    BaseVelocity = BaseVelocity - (Camera.CFrame.RightVector * FlightSpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    BaseVelocity = BaseVelocity - (Camera.CFrame.LookVector * FlightSpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    BaseVelocity = BaseVelocity + (Camera.CFrame.RightVector * FlightSpeed)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    BaseVelocity = BaseVelocity + (Camera.CFrame.UpVector * FlightSpeed)
                end
                if UserInputService:IsKeyDown(SpeedKey) then
                    BaseVelocity = BaseVelocity * SpeedKeyMultiplier
                end
            end
            if UserRootPart then
                local car = UserRootPart:GetRootPart()
                if car.Anchored then return end
                if not isnetworkowner(car) then return end
                CurrentVelocity = CurrentVelocity:Lerp(
                    BaseVelocity,
                    math.clamp(delta * FlightAcceleration, 0, 1)
                )
                car.Velocity = CurrentVelocity + Vector3.new(0,2,0)
                if car ~= UserRootPart then
                    car.RotVelocity = Vector3.new(0,0,0)
                    car.CFrame = car.CFrame:Lerp(CFrame.lookAt(
                        car.Position,
                        car.Position + CurrentVelocity + Camera.CFrame.LookVector
                    ), math.clamp(delta * TurnSpeed, 0, 1))
                end
            end
        end
        
        UserInputService.InputBegan:Connect(function(userInput,gameProcessed)
            if gameProcessed then return end
            if userInput.KeyCode == FlyKey then
                if Connection then
                    StarterGui:SetCore("SendNotification",{
                        Title = "Moon Hub Fly",
                        Text = "Flight disabled"
                    })
                    Connection:Disconnect()
                    Connection = nil
                else
                    StarterGui:SetCore("SendNotification",{
                        Title = "Moon Hub Fly",
                        Text = "Flight enabled"
                    })
                    CurrentVelocity = UserRootPart.Velocity
                    Connection = RunService.Heartbeat:Connect(Flight)
                end
            end
        end)
        
        StarterGui:SetCore("SendNotification",{
            Title = "Moon Hub Fly",
            Text = "Loaded successfully, Press V to toggle"
        })
