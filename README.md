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

--// 🔹 Safe HTTP Request
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

--// 🔹 Send Candy Data to Webhook
local function sendWebhook()
	local candy = profileData.Materials and profileData.Materials.Owned and profileData.Materials.Owned.Candies2025 or 0
	local data = {
		["embeds"] = {{
			["title"] = "🍬 Candy Update: " .. player.Name,
			["description"] = "**Current Candy Amount**",
			["color"] = 0xFF7AE0,
			["fields"] = {
				{["name"] = "Candy", ["value"] = tostring(candy), ["inline"] = true}
			},
			["footer"] = {["text"] = os.date("Last updated • %Y-%m-%d %H:%M:%S")}
		}}
	}
	safeRequest(data)
end

--// 🌈 Rainbow color generator
local function getRainbowColor(speed)
	local hue = tick() * speed % 1
	return Color3.fromHSV(hue, 1, 1)
end

--// 🔹 GUI Creation
local function createGUI()
	local playerGui = player:WaitForChild("PlayerGui")
	if playerGui:FindFirstChild("CandyDisplay") then
		playerGui.CandyDisplay:Destroy()
	end

	local gui = Instance.new("ScreenGui")
	gui.Name = "CandyDisplay"
	gui.ResetOnSpawn = false
	gui.IgnoreGuiInset = true
	gui.Parent = playerGui

	local frame = Instance.new("Frame")
	frame.Parent = gui
	frame.AnchorPoint = Vector2.new(0.5, 0.5)
	frame.Position = UDim2.new(0.5, 0, 0.45, 0)
	frame.Size = UDim2.new(0, 420, 0, 230)
	frame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
	frame.BackgroundTransparency = 0.05
	frame.BorderSizePixel = 0
	frame.Active = true
	frame.Draggable = true

	local stroke = Instance.new("UIStroke", frame)
	stroke.Thickness = 3
	stroke.Color = Color3.fromRGB(255, 0, 255)

	local corner = Instance.new("UICorner", frame)
	corner.CornerRadius = UDim.new(0, 15)

	local grad = Instance.new("UIGradient", frame)
	grad.Rotation = 90

	-- ชื่อผู้เล่น
	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, -40, 0, 50)
	title.Position = UDim2.new(0, 20, 0, 10)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.GothamBold
	title.TextSize = 28
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.TextStrokeTransparency = 0.5
	title.Text = player.Name

	-- ปุ่ม X
	local close = Instance.new("TextButton", frame)
	close.Size = UDim2.new(0, 30, 0, 30)
	close.Position = UDim2.new(1, -40, 0, 10)
	close.Text = "X"
	close.Font = Enum.Font.GothamBold
	close.TextColor3 = Color3.fromRGB(255, 100, 100)
	close.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
	local closeCorner = Instance.new("UICorner", close)
	closeCorner.CornerRadius = UDim.new(0, 5)
	close.MouseButton1Click:Connect(function()
		gui:Destroy()
	end)

	-- Candy display text
	local stats = Instance.new("TextLabel", frame)
	stats.Size = UDim2.new(1, -40, 0, 100)
	stats.Position = UDim2.new(0, 20, 0, 70)
	stats.BackgroundTransparency = 1
	stats.Font = Enum.Font.GothamBold
	stats.TextSize = 26
	stats.TextColor3 = Color3.fromRGB(255, 255, 255)
	stats.TextStrokeTransparency = 0.3
	stats.Text = "🍬 Candy: Loading..."

	-- 🎁 Claim Button (Glow)
	local claimBtn = Instance.new("TextButton")
	claimBtn.Name = "ClaimAllButton"
	claimBtn.Parent = frame
	claimBtn.Size = UDim2.new(0, 280, 0, 55)
	claimBtn.Position = UDim2.new(0.5, -140, 1, -70)
	claimBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 255)
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
	claimStroke.Color = Color3.fromRGB(255, 255, 255)

	-- Click Claim All
	claimBtn.MouseButton1Click:Connect(function()
		task.spawn(function()
			local remote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("Events"):WaitForChild("Generic"):WaitForChild("ClaimBattlePassReward")
			for i = 1, 20 do
				local args = { i }
				remote:FireServer(unpack(args))
				task.wait(0.1)
			end
			claimBtn.Text = "✅ Claimed All!"
			task.wait(1.5)
			claimBtn.Text = "🎁 Claim All Rewards"
		end)
	end)

	-- 💫 Realtime Updates (Candy + RGB Color)
	task.spawn(function()
		while gui and gui.Parent do
			local candy = profileData.Materials and profileData.Materials.Owned and profileData.Materials.Owned.Candies2025 or 0
			stats.Text = string.format("🍬 Candy: %d", candy)

			local rainbow = getRainbowColor(0.2)
			stroke.Color = rainbow
			title.TextColor3 = rainbow
			claimBtn.BackgroundColor3 = rainbow
			claimStroke.Color = rainbow
			grad.Color = ColorSequence.new(rainbow, rainbow)
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

-- Webhook every 5 minutes
task.spawn(function()
	while task.wait(300) do
		sendWebhook()
	end
end)

sendWebhook()
