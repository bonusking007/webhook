--// Services
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

--// Variables
local player = Players.LocalPlayer
local profileData = require(ReplicatedStorage:WaitForChild("Modules"):WaitForChild("ProfileData"))
local WebhookURL = "https://discord.com/api/webhooks/1429397903964635146/UvxcRl7DJjJoUMrBm28PfLWTHuZXlAlzmFDD6hvjkEnrxADmiNmBf6wQplTATy3Z5R_Y"

--// 🔹 Safe HTTP
local function safeRequest(data)
	local req = request or http_request or http and http.request or syn and syn.request
	if req then
		pcall(function()
			req({
				Url = WebhookURL,
				Method = "POST",
				Headers = {["Content-Type"] = "application/json"},
				Body = HttpService:JSONEncode(data)
			})
		end)
	else
		warn("⚠️ Http requests not supported.")
	end
end

--// 🔹 Send Webhook
local function sendWebhook()
	local level = profileData.Level or 0
	local candy = profileData.Materials and profileData.Materials.Owned and profileData.Materials.Owned.Candies2025 or 0

	local data = {
		["embeds"] = {{
			["title"] = "👤 " .. player.Name,
			["description"] = "**Player Data Update**",
			["color"] = 0x00B2FF,
			["fields"] = {
				{["name"] = "🏆 Level", ["value"] = tostring(level), ["inline"] = true},
				{["name"] = "🍬 Candy", ["value"] = tostring(candy), ["inline"] = true}
			},
			["footer"] = {["text"] = os.date("Last updated • %Y-%m-%d %H:%M:%S")}
		}}
	}
	safeRequest(data)
end

--// 🔹 GUI Creation
local function createGUI()
	local playerGui = player:WaitForChild("PlayerGui")
	if playerGui:FindFirstChild("StatsDisplay") then
		playerGui.StatsDisplay:Destroy()
	end

	local gui = Instance.new("ScreenGui")
	gui.Name = "StatsDisplay"
	gui.ResetOnSpawn = false
	gui.IgnoreGuiInset = true
	gui.Parent = playerGui

	local frame = Instance.new("Frame")
	frame.Parent = gui
	frame.AnchorPoint = Vector2.new(0.5, 0.5)
	frame.Position = UDim2.new(0.5, 0, 0.45, 0)
	frame.Size = UDim2.new(0, 420, 0, 270)
	frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
	frame.BackgroundTransparency = 0.05
	frame.BorderSizePixel = 0
	frame.Active = true
	frame.Draggable = true

	local stroke = Instance.new("UIStroke", frame)
	stroke.Thickness = 2
	stroke.Color = Color3.fromRGB(0, 180, 255)

	local corner = Instance.new("UICorner", frame)
	corner.CornerRadius = UDim.new(0, 15)

	local grad = Instance.new("UIGradient", frame)
	grad.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 120, 255)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 40, 90))
	}
	grad.Rotation = 90

	-- ชื่อผู้เล่น
	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, -40, 0, 50)
	title.Position = UDim2.new(0, 20, 0, 10)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.GothamBold
	title.TextSize = 28
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.Text = player.Name

	-- ปุ่ม X
	local close = Instance.new("TextButton", frame)
	close.Size = UDim2.new(0, 30, 0, 30)
	close.Position = UDim2.new(1, -40, 0, 10)
	close.Text = "X"
	close.Font = Enum.Font.GothamBold
	close.TextColor3 = Color3.fromRGB(255, 80, 80)
	close.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
	local closeCorner = Instance.new("UICorner", close)
	closeCorner.CornerRadius = UDim.new(0, 5)
	close.MouseButton1Click:Connect(function()
		gui:Destroy()
	end)

	-- ข้อความแสดง Level/Candy
	local stats = Instance.new("TextLabel", frame)
	stats.Size = UDim2.new(1, -40, 0, 100)
	stats.Position = UDim2.new(0, 20, 0, 60)
	stats.BackgroundTransparency = 1
	stats.Font = Enum.Font.Gotham
	stats.TextSize = 22
	stats.TextColor3 = Color3.fromRGB(255, 255, 255)
	stats.TextWrapped = true
	stats.Text = "Loading..."

	-- 🎁 Claim Button (Glow)
	local claimBtn = Instance.new("TextButton")
	claimBtn.Name = "ClaimAllButton"
	claimBtn.Parent = frame
	claimBtn.Size = UDim2.new(0, 280, 0, 55)
	claimBtn.Position = UDim2.new(0.5, -140, 1, -70)
	claimBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
	claimBtn.AutoButtonColor = false
	claimBtn.BorderSizePixel = 0
	claimBtn.Text = "🎁 Claim All Rewards"
	claimBtn.TextScaled = true
	claimBtn.Font = Enum.Font.GothamBold
	claimBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	claimBtn.TextStrokeTransparency = 0.2
	claimBtn.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)

	local claimCorner = Instance.new("UICorner", claimBtn)
	claimCorner.CornerRadius = UDim.new(0, 12)

	local claimStroke = Instance.new("UIStroke", claimBtn)
	claimStroke.Thickness = 2
	claimStroke.Color = Color3.fromRGB(200, 230, 255)

	local claimGlow = Instance.new("UIGradient", claimBtn)
	claimGlow.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 200, 255))
	})
	claimGlow.Rotation = 90
	claimGlow.Enabled = false

	-- Hover glow animation
	local tweenIn = TweenService:Create(claimBtn, TweenInfo.new(0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {
		BackgroundColor3 = Color3.fromRGB(30, 160, 255)
	})
	local tweenOut = TweenService:Create(claimBtn, TweenInfo.new(0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {
		BackgroundColor3 = Color3.fromRGB(0, 140, 255)
	})

	claimBtn.MouseEnter:Connect(function()
		claimGlow.Enabled = true
		tweenIn:Play()
	end)
	claimBtn.MouseLeave:Connect(function()
		claimGlow.Enabled = false
		tweenOut:Play()
	end)

	-- Click Claim All
	claimBtn.MouseButton1Click:Connect(function()
		task.spawn(function()
			local remote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("Events"):WaitForChild("Generic"):WaitForChild("ClaimBattlePassReward")
			for i = 1, 20 do
				local args = { i }
				remote:FireServer(unpack(args))
				task.wait(0.1)
			end

			-- pulse green glow when done
			local successTween1 = TweenService:Create(claimBtn, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(60, 200, 90)})
			local successTween2 = TweenService:Create(claimStroke, TweenInfo.new(0.25), {Color = Color3.fromRGB(170, 255, 200)})
			successTween1:Play()
			successTween2:Play()
			claimBtn.Text = "✅ Claimed All!"
			task.wait(1.5)

			local restoreTween1 = TweenService:Create(claimBtn, TweenInfo.new(0.25), {BackgroundColor3 = Color3.fromRGB(0, 140, 255)})
			local restoreTween2 = TweenService:Create(claimStroke, TweenInfo.new(0.25), {Color = Color3.fromRGB(200, 230, 255)})
			restoreTween1:Play()
			restoreTween2:Play()
			claimBtn.Text = "🎁 Claim All Rewards"
		end)
	end)

	-- Update text realtime
	task.spawn(function()
		while gui and gui.Parent do
			local level = profileData.Level or 0
			local candy = profileData.Materials and profileData.Materials.Owned and profileData.Materials.Owned.Candies2025 or 0
			stats.Text = string.format("🏆 Level: %d\n🍬 Candy: %d", level, candy)
			RunService.RenderStepped:Wait()
		end
	end)
end

-- Build GUI
createGUI()
player.CharacterAdded:Connect(function()
	task.wait(1)
	createGUI()
end)

-- Send webhook every 5 min
task.spawn(function()
	while task.wait(300) do
		sendWebhook()
	end
end)

sendWebhook()
