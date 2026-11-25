local LOGO_LINK = ""
local PANEL_NAME = "Redx Painel"
local PANEL_SUBTITLE = "By Denolk and Ghost V1.0"

local env = (type(getgenv) == "function" and getgenv()) or _G
if env then
	if type(env.RedxKeyGateClose) == "function" then
		pcall(env.RedxKeyGateClose)
	end
	if env.Redx and type(env.Redx.UI) == "table" and type(env.Redx.UI.Destroy) == "function" then
		pcall(env.Redx.UI.Destroy)
	end
	if type(env.RedxPainelUnload) == "function" then
		pcall(env.RedxPainelUnload)
	end
end

local KeySession

local function requireRedxKey()
	local HttpService = game:GetService("HttpService")
	local Players = game:GetService("Players")
	local UserInputService = game:GetService("UserInputService")
	local player = Players.LocalPlayer
	local parent
	pcall(function()
		if type(gethui) == "function" then
			parent = gethui()
		end
	end)
	if not parent then
		pcall(function()
			parent = game:GetService("CoreGui")
		end)
	end
	if not parent and player then
		parent = player:WaitForChild("PlayerGui", 5)
	end
	if not parent then
		return false
	end

	local gui = Instance.new("ScreenGui")
	gui.Name = "RedxKeyGate"
	gui.ResetOnSpawn = false
	gui.IgnoreGuiInset = true
	gui.DisplayOrder = 10000
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	local parented = pcall(function()
		gui.Parent = parent
	end)
	if not parented and player then
		parented = pcall(function()
			gui.Parent = player:WaitForChild("PlayerGui", 5)
		end)
	end
	if not parented or not gui.Parent then
		gui:Destroy()
		return false
	end

	local completed = Instance.new("BindableEvent")
	local connections = {}
	local finished = false
	local authorized = false
	local busy = false
	local attempt = 0
	local closeGate
	local function connect(signal, fn)
		local conn = signal:Connect(fn)
		table.insert(connections, conn)
		return conn
	end
	local function make(class, props)
		local obj = Instance.new(class)
		for k, v in pairs(props) do
			obj[k] = v
		end
		return obj
	end
	local function corner(obj, radius)
		make("UICorner", { CornerRadius = UDim.new(0, radius), Parent = obj })
	end
	local background = make("Frame", {
		Size = UDim2.fromScale(1, 1),
		BackgroundColor3 = Color3.fromRGB(0, 0, 0),
		BackgroundTransparency = 0.35,
		BorderSizePixel = 0,
		Active = true,
		Parent = gui
	})
	local card = make("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.5, 0.5),
		Size = UDim2.fromOffset(400, 292),
		BackgroundColor3 = Color3.fromRGB(14, 9, 12),
		BorderSizePixel = 0,
		Parent = background
	})
	corner(card, 10)
	make("UIStroke", { Color = Color3.fromRGB(130, 6, 22), Thickness = 1, Parent = card })
	local function fit()
		local camera = workspace.CurrentCamera
		local size = camera and camera.ViewportSize or Vector2.new(800, 600)
		return math.min(1, math.max(0.2, (size.X - 24) / 400), math.max(0.2, (size.Y - 24) / 292))
	end
	local scale = make("UIScale", { Scale = fit(), Parent = card })
	local camera = workspace.CurrentCamera
	if camera then
		connect(camera:GetPropertyChangedSignal("ViewportSize"), function()
			scale.Scale = fit()
		end)
	end
	local header = make("TextLabel", {
		Size = UDim2.new(1, -56, 0, 48),
		Position = UDim2.fromOffset(20, 2),
		BackgroundTransparency = 1,
		Text = PANEL_NAME .. "  |  KEY",
		TextColor3 = Color3.fromRGB(230, 0, 28),
		Font = Enum.Font.GothamBold,
		TextSize = 17,
		TextXAlignment = Enum.TextXAlignment.Left,
		Active = true,
		Parent = card
	})
	local close = make("TextButton", {
		Position = UDim2.new(1, -42, 0, 10),
		Size = UDim2.fromOffset(30, 30),
		BackgroundTransparency = 1,
		Text = "×",
		TextColor3 = Color3.fromRGB(240, 236, 238),
		Font = Enum.Font.GothamBold,
		TextSize = 24,
		Parent = card
	})
	make("TextLabel", {
		Position = UDim2.fromOffset(20, 50),
		Size = UDim2.new(1, -40, 0, 42),
		BackgroundTransparency = 1,
		Text = "Insira sua Key para liberar o painel.\n" .. PANEL_SUBTITLE,
		TextColor3 = Color3.fromRGB(208, 202, 206),
		Font = Enum.Font.GothamMedium,
		TextSize = 12,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextWrapped = true,
		Parent = card
	})
	local input = make("TextBox", {
		Position = UDim2.fromOffset(20, 106),
		Size = UDim2.new(1, -40, 0, 42),
		BackgroundColor3 = Color3.fromRGB(28, 16, 22),
		BorderSizePixel = 0,
		Text = "",
		PlaceholderText = "REDX-...",
		PlaceholderColor3 = Color3.fromRGB(122, 112, 118),
		TextColor3 = Color3.fromRGB(240, 236, 238),
		Font = Enum.Font.GothamMedium,
		TextSize = 14,
		ClearTextOnFocus = false,
		MultiLine = false,
		Parent = card
	})
	corner(input, 6)
	local status = make("TextLabel", {
		Position = UDim2.fromOffset(20, 156),
		Size = UDim2.new(1, -40, 0, 48),
		BackgroundTransparency = 1,
		Text = "O painel só inicia após a validação online.",
		TextColor3 = Color3.fromRGB(208, 202, 206),
		Font = Enum.Font.GothamMedium,
		TextSize = 12,
		TextWrapped = true,
		Parent = card
	})
	local submit = make("TextButton", {
		Position = UDim2.fromOffset(20, 220),
		Size = UDim2.new(1, -40, 0, 44),
		BackgroundColor3 = Color3.fromRGB(230, 0, 28),
		BorderSizePixel = 0,
		Text = "VALIDAR KEY",
		TextColor3 = Color3.fromRGB(255, 255, 255),
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		Parent = card
	})
	corner(submit, 6)

	local function finish(ok)
		if finished then
			return
		end
		finished = true
		authorized = ok == true
		attempt = attempt + 1
		input.Text = ""
		for _, conn in ipairs(connections) do
			conn:Disconnect()
		end
		gui:Destroy()
		if env and env.RedxKeyGateClose == closeGate then
			env.RedxKeyGateClose = nil
		end
		completed:Fire()
	end
	closeGate = function()
		finish(false)
	end
	if env then
		env.RedxKeyGateClose = closeGate
	end
	connect(close.Activated, closeGate)
	connect(gui.Destroying, closeGate)

	local dragStart, startPosition, dragInput
	connect(header.InputBegan, function(event)
		if event.UserInputType == Enum.UserInputType.MouseButton1 or event.UserInputType == Enum.UserInputType.Touch then
			dragInput = event
			dragStart = event.Position
			startPosition = card.Position
		end
	end)
	connect(UserInputService.InputChanged, function(event)
		if dragInput and (event.UserInputType == Enum.UserInputType.MouseMovement or event == dragInput) then
			local delta = event.Position - dragStart
			card.Position = UDim2.new(startPosition.X.Scale, startPosition.X.Offset + delta.X, startPosition.Y.Scale, startPosition.Y.Offset + delta.Y)
		end
	end)
	connect(UserInputService.InputEnded, function(event)
		if event == dragInput or event.UserInputType == Enum.UserInputType.MouseButton1 then
			dragInput = nil
		end
	end)

	local function deviceId()
		local id
		if type(gethwid) == "function" then
			pcall(function()
				id = gethwid()
			end)
		end
		if type(id) ~= "string" or id == "" then
			pcall(function()
				id = game:GetService("RbxAnalyticsService"):GetClientId()
			end)
		end
		if type(id) == "string" and id ~= "" then
			return id
		end
		return nil
	end
	local function validate()
		if finished or busy then
			return
		end
		local key = input.Text:match("^%s*(.-)%s*$")
		if key == "" or #key > 256 then
			status.Text = "Insira uma Key válida."
			return
		end
		local id = deviceId()
		if not id then
			status.Text = "Não foi possível identificar este aparelho."
			return
		end
		busy = true
		attempt = attempt + 1
		local token = attempt
		submit.Text = "VALIDANDO..."
		submit.AutoButtonColor = false
		status.Text = "Consultando servidor..."
		local function reject(message)
			if finished or token ~= attempt then
				return
			end
			attempt = attempt + 1
			busy = false
			submit.Text = "VALIDAR KEY"
			submit.AutoButtonColor = true
			status.Text = message
		end
		task.delay(15, function()
			reject("Tempo esgotado. Verifique sua conexão e tente novamente.")
		end)
		task.spawn(function()
			local ok, response = pcall(function()
				local options = {
					Url = "https://redx-panel-key.lovable.app/api/public/v1/check",
					Method = "POST",
					Headers = { ["Content-Type"] = "application/json" },
					Body = HttpService:JSONEncode({ key = key, device_id = id })
				}
				local send = (type(request) == "function" and request)
					or (type(http_request) == "function" and http_request)
					or (type(syn) == "table" and type(syn.request) == "function" and syn.request)
					or (type(http) == "table" and type(http.request) == "function" and http.request)
				if send then
					return send(options)
				end
				return HttpService:RequestAsync(options)
			end)
			key = nil
			if finished or token ~= attempt then
				return
			end
			if not ok or type(response) ~= "table" or type(response.Body) ~= "string" then
				reject("Erro de conexão. Acesso não liberado.")
				return
			end
			local decoded, data = pcall(function()
				return HttpService:JSONDecode(response.Body)
			end)
			if not decoded or type(data) ~= "table" then
				reject("Resposta inválida do servidor. Tente novamente.")
				return
			end
			local code = tonumber(response.StatusCode or response.Status)
			if code == 200 and data.valid == true and data.status == "active" then
				KeySession = {
					secondsLeft = data.seconds_left,
					expiresAt = data.expires_at,
					receivedAt = os.clock()
				}
				finish(true)
				return
			end
			local messages = {
				invalid = "Key inválida.",
				banned = "Esta Key foi banida.",
				expired = "Esta Key expirou.",
				device_limit = "Limite de aparelhos atingido."
			}
			reject(messages[data.status] or "Validação recusada pelo servidor.")
		end)
	end
	connect(submit.Activated, validate)
	connect(input.FocusLost, function(enterPressed)
		if enterPressed then
			validate()
		end
	end)
	completed.Event:Wait()
	completed:Destroy()
	return authorized
end

if not requireRedxKey() then
	return
end

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local spawnTask = (task and task.spawn) or function(fn, ...)
	return coroutine.wrap(fn)(...)
end
local waitTask = (task and task.wait) or wait
local clamp = math.clamp or function(v, lo, hi)
	if v < lo then
		return lo
	elseif v > hi then
		return hi
	end
	return v
end

local function isColor3Value(c)
	if type(typeof) == "function" then
		return typeof(c) == "Color3"
	end
	return type(c) == "table" and c.R ~= nil and c.G ~= nil and c.B ~= nil
end

local Theme = {
	WindowBg = Color3.fromRGB(14, 9, 12),
	SidebarBg = Color3.fromRGB(18, 11, 14),
	HeaderBg = Color3.fromRGB(16, 10, 13),
	CategoryBg = Color3.fromRGB(23, 14, 18),
	CategoryStroke = Color3.fromRGB(36, 22, 28),
	SubTabActiveBg = Color3.fromRGB(38, 18, 25),
	CardBg = Color3.fromRGB(18, 11, 15),
	CardStroke = Color3.fromRGB(34, 20, 26),
	Divider = Color3.fromRGB(33, 20, 26),
	FloatTitleBg = Color3.fromRGB(52, 3, 10),
	FloatStroke = Color3.fromRGB(82, 12, 22),
	Accent = Color3.fromRGB(230, 0, 28),
	AccentHover = Color3.fromRGB(255, 28, 52),
	AccentDark = Color3.fromRGB(130, 6, 22),
	ToggleOff = Color3.fromRGB(38, 28, 33),
	SliderTrack = Color3.fromRGB(54, 9, 17),
	BadgeBg = Color3.fromRGB(8, 5, 7),
	BadgeStroke = Color3.fromRGB(28, 18, 23),
	OverlayBg = Color3.fromRGB(10, 7, 9),
	PillStroke = Color3.fromRGB(184, 9, 30),
	TextPrimary = Color3.fromRGB(240, 236, 238),
	TextSecondary = Color3.fromRGB(208, 202, 206),
	TextMuted = Color3.fromRGB(122, 112, 118),
	ConnectedBg = Color3.fromRGB(18, 38, 16),
	ConnectedStroke = Color3.fromRGB(63, 130, 39),
	ConnectedText = Color3.fromRGB(86, 192, 58),
	White = Color3.fromRGB(255, 255, 255)
}

local Connections = {}
local function trackConn(conn)
	if conn then
		table.insert(Connections, conn)
	end
	return conn
end

local function create(className, props, children)
	local ok, inst = pcall(Instance.new, className)
	if not ok or not inst then
		return nil
	end
	if props then
		for k, v in pairs(props) do
			if k ~= "Parent" then
				pcall(function()
					inst[k] = v
				end)
			end
		end
		if props.Parent then
			pcall(function()
				inst.Parent = props.Parent
			end)
		end
	end
	if children then
		for i = 1, #children do
			if children[i] then
				pcall(function()
					children[i].Parent = inst
				end)
			end
		end
	end
	return inst
end

local function addCorner(parent, radius)
	if not parent then
		return nil
	end
	return create("UICorner", {
		CornerRadius = UDim.new(0, radius or 6),
		Parent = parent
	})
end

local function addStroke(parent, color, thickness, transparency)
	if not parent then
		return nil
	end
	return create("UIStroke", {
		Color = color or Theme.CardStroke,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		Parent = parent
	})
end

local function tween(obj, duration, goal)
	if not obj then
		return nil
	end
	local ok, tw = pcall(function()
		return TweenService:Create(obj, TweenInfo.new(duration or 0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), goal)
	end)
	if ok and tw then
		tw:Play()
		return tw
	end
	for k, v in pairs(goal) do
		pcall(function()
			obj[k] = v
		end)
	end
	return nil
end

local HAS_DRAWING = (type(Drawing) == "table" and type(Drawing.new) == "function")
local function dNew(cls)
	if not HAS_DRAWING then
		return nil
	end
	local ok, obj = pcall(Drawing.new, cls)
	if ok then
		return obj
	end
	return nil
end

local function dSet(obj, prop, val)
	if not obj then
		return
	end
	pcall(function()
		obj[prop] = val
	end)
end

local function dRemove(obj)
	if not obj then
		return
	end
	pcall(function()
		obj:Remove()
	end)
end

local PanelDestroyed = false
local Redx = {
	_v = "11.2",
	HasDrawing = HAS_DRAWING
}
if env then
	env.Redx = Redx
end

Redx.State = {
	Aimbot = {
		enabled = false,
		target = "torso",
		smooth = 15,
		radius = 235,
		drawAimLine = false
	},
	FOV = {
		enabled = false,
		radius = 235
	},
	ESP = {
		enabled = false,
		box = true,
		boxStyle = "corner",
		name = true,
		showDistance = true,
		health = true,
		healthText = true,
		healthBar = true,
		headDot = true,
		skeleton = false,
		tracer = false,
		tracerStyle = "base",
		chams = false,
		aimLine = false,
		teamCheck = false,
		color = Color3.fromRGB(230, 0, 28),
		teamColor = Color3.fromRGB(80, 220, 80),
		distance = 500,
		thickness = 3,
		textSize = 14
	},
	Godmode = {
		enabled = false
	},
	Performance = {
		fpsBoost = false,
		removeShadows = false,
		lowTextures = false
	},
	Bypass = {
		stealth = false,
		unlockFPS = false
	}
}

local SyncListeners = {}
local function onSync(channel, fn)
	if not SyncListeners[channel] then
		SyncListeners[channel] = {}
	end
	table.insert(SyncListeners[channel], fn)
end

local function emitSync(channel, val)
	local list = SyncListeners[channel]
	if list then
		for i = 1, #list do
			pcall(list[i], val)
		end
	end
end

local GuiParent = nil
pcall(function()
	if type(gethui) == "function" then
		GuiParent = gethui()
	end
end)
if not GuiParent then
	pcall(function()
		GuiParent = game:GetService("CoreGui")
	end)
end
if not GuiParent and LocalPlayer then
	pcall(function()
		GuiParent = LocalPlayer:WaitForChild("PlayerGui", 3)
	end)
end

local ScreenGui = create("ScreenGui", {
	Name = "RedxPainelGui",
	ResetOnSpawn = false,
	IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling
})

pcall(function()
	if syn and type(syn.protect_gui) == "function" and GuiParent == game:GetService("CoreGui") then
		syn.protect_gui(ScreenGui)
	end
end)

if ScreenGui and GuiParent then
	pcall(function()
		ScreenGui.Parent = GuiParent
	end)
end

Redx.FOV = { _listeners = {} }

Redx.FOV.onChanged = function(fn)
	if type(fn) == "function" then
		table.insert(Redx.FOV._listeners, fn)
	end
end

Redx.FOV._notify = function()
	for _, fn in ipairs(Redx.FOV._listeners) do
		pcall(fn)
	end
end

local FOVFrame = create("Frame", {
	Name = "FOVCircle",
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.new(0, Redx.State.FOV.radius * 2, 0, Redx.State.FOV.radius * 2),
	BackgroundTransparency = 1,
	Visible = Redx.State.FOV.enabled,
	Active = false,
	ZIndex = 1,
	Parent = ScreenGui
})
addCorner(FOVFrame, 9999)
addStroke(FOVFrame, Color3.fromRGB(245, 245, 247), 1.3, 0.15)

local function fovRefresh()
	local S = Redx.State.FOV
	if not FOVFrame then
		return
	end
	local d = math.floor(S.radius * 2)
	FOVFrame.Size = UDim2.new(0, d, 0, d)
	FOVFrame.Position = UDim2.fromScale(0.5, 0.5)
	FOVFrame.Visible = S.enabled
end

Redx.FOV.onChanged(fovRefresh)

Redx.FOV.Toggle = function(s)
	local b = s and true or false
	Redx.State.FOV.enabled = b
	Redx.FOV._notify()
	emitSync("FOV.enabled", b)
	emitSync("Keybinds")
end

Redx.FOV.SetRadius = function(p)
	local r = clamp(tonumber(p) or 235, 40, 450)
	Redx.State.FOV.radius = r
	Redx.FOV._notify()
	emitSync("FOV.radius", r)
end

Redx.Aimbot = {}

local function aimGetTargetPart(plr)
	local char = plr.Character
	if not char then
		return nil
	end
	local target = Redx.State.Aimbot.target
	if target == "head" then
		return char:FindFirstChild("Head")
	else
		return char:FindFirstChild("HumanoidRootPart")
			or char:FindFirstChild("UpperTorso")
			or char:FindFirstChild("Torso")
	end
end

local function aimFindBest()
	camera = workspace.CurrentCamera or camera
	if not camera then
		return nil
	end
	local S = Redx.State.Aimbot
	local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
	local best, bestD = nil, S.radius
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local hum = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then
				local part = aimGetTargetPart(plr)
				if part then
					local sp, onScreen = camera:WorldToViewportPoint(part.Position)
					if onScreen then
						local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
						if d < bestD then
							best, bestD = part, d
						end
					end
				end
			end
		end
	end
	return best
end

Redx.Aimbot.Toggle = function(s)
	local b = s and true or false
	Redx.State.Aimbot.enabled = b
	emitSync("Aimbot.enabled", b)
	emitSync("Keybinds")
end

Redx.Aimbot.SetTarget = function(t)
	if t == "head" or t == "torso" then
		Redx.State.Aimbot.target = t
		emitSync("Aimbot.target", t)
	end
end

Redx.Aimbot.SetSmooth = function(n)
	local v = clamp(tonumber(n) or 15, 1, 100)
	Redx.State.Aimbot.smooth = v
	emitSync("Aimbot.smooth", v)
end

Redx.Aimbot.SetRadius = function(p)
	local v = clamp(tonumber(p) or 235, 40, 450)
	Redx.State.Aimbot.radius = v
	emitSync("Aimbot.radius", v)
end

Redx.Aimbot.SetDrawLine = function(s)
	local b = s and true or false
	Redx.State.Aimbot.drawAimLine = b
	Redx.State.ESP.aimLine = b
	emitSync("Aimbot.drawAimLine", b)
end

local aimLineObj = nil
if HAS_DRAWING then
	aimLineObj = dNew("Line")
end
if aimLineObj then
	dSet(aimLineObj, "Thickness", 1)
	dSet(aimLineObj, "Transparency", 0.4)
	dSet(aimLineObj, "Visible", false)
end

trackConn(RunService.RenderStepped:Connect(function(dt)
	camera = workspace.CurrentCamera or camera
	local S = Redx.State
	if not S.Aimbot.enabled or not camera then
		if aimLineObj then
			dSet(aimLineObj, "Visible", false)
		end
		return
	end
	local target = aimFindBest()
	if not target then
		if aimLineObj then
			dSet(aimLineObj, "Visible", false)
		end
		return
	end
	pcall(function()
		local goal = CFrame.new(camera.CFrame.Position, target.Position)
		local t = (100 - S.Aimbot.smooth) / 100
		local alpha = clamp(t * (dt or 0.016) * 60, 0, 1)
		camera.CFrame = camera.CFrame:Lerp(goal, alpha)
	end)

	if aimLineObj then
		if S.Aimbot.drawAimLine then
			local sp, vis = camera:WorldToViewportPoint(target.Position)
			if sp and vis then
				dSet(aimLineObj, "From", Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2))
				dSet(aimLineObj, "To", Vector2.new(sp.X, sp.Y))
				dSet(aimLineObj, "Color", S.ESP.color)
				dSet(aimLineObj, "Visible", true)
			else
				dSet(aimLineObj, "Visible", false)
			end
		else
			dSet(aimLineObj, "Visible", false)
		end
	end
end))

Redx.ESP = {}

local ESPData = {}
local ChamsData = {}

local function espNewFor(plr)
	local S = Redx.State.ESP
	local d = {}
	d.box = dNew("Square")
	d.boxFill = dNew("Square")
	d.name = dNew("Text")
	d.distance = dNew("Text")
	d.healthText = dNew("Text")
	d.healthBg = dNew("Square")
	d.healthFill = dNew("Square")
	d.headDot = dNew("Circle")
	d.tracer = dNew("Line")
	d.corners = {}
	for i = 1, 8 do
		d.corners[i] = dNew("Line")
	end
	d.skel = {}
	for i = 1, 15 do
		d.skel[i] = dNew("Line")
	end

	if d.box then
		dSet(d.box, "Filled", false)
		dSet(d.box, "Transparency", 0.7)
		dSet(d.box, "Thickness", S.thickness)
	end
	if d.boxFill then
		dSet(d.boxFill, "Filled", true)
		dSet(d.boxFill, "Transparency", 0.8)
	end
	if d.name then
		dSet(d.name, "Size", S.textSize)
		dSet(d.name, "Center", true)
		dSet(d.name, "Outline", true)
		dSet(d.name, "OutlineColor", Color3.fromRGB(0, 0, 0))
	end
	if d.distance then
		dSet(d.distance, "Size", math.max(10, S.textSize - 2))
		dSet(d.distance, "Center", true)
		dSet(d.distance, "Outline", true)
	end
	if d.healthText then
		dSet(d.healthText, "Size", math.max(10, S.textSize - 3))
		dSet(d.healthText, "Center", true)
		dSet(d.healthText, "Outline", true)
	end
	if d.healthBg then
		dSet(d.healthBg, "Filled", true)
		dSet(d.healthBg, "Color", Color3.fromRGB(0, 0, 0))
		dSet(d.healthBg, "Transparency", 0.4)
	end
	if d.healthFill then
		dSet(d.healthFill, "Filled", true)
	end
	if d.headDot then
		dSet(d.headDot, "Filled", true)
	end
	if d.tracer then
		dSet(d.tracer, "Transparency", 0.4)
	end
	for i = 1, #d.corners do
		if d.corners[i] then
			dSet(d.corners[i], "Thickness", S.thickness)
		end
	end
	for i = 1, #d.skel do
		if d.skel[i] then
			dSet(d.skel[i], "Thickness", S.thickness)
		end
	end

	ESPData[plr] = d
	return d
end

local function espSetVisible(d, visible)
	if not d then
		return
	end
	local props = { "box", "boxFill", "name", "distance", "healthText", "healthBg", "healthFill", "headDot", "tracer" }
	for i = 1, #props do
		dSet(d[props[i]], "Visible", visible)
	end
	if d.corners then
		for i = 1, #d.corners do
			dSet(d.corners[i], "Visible", visible)
		end
	end
	if d.skel then
		for i = 1, #d.skel do
			dSet(d.skel[i], "Visible", visible)
		end
	end
end

local function espDestroy(plr)
	local d = ESPData[plr]
	if not d then
		return
	end
	for k, v in pairs(d) do
		if k == "skel" or k == "corners" then
			for i = 1, #v do
				dRemove(v[i])
			end
		else
			dRemove(v)
		end
	end
	ESPData[plr] = nil
end

local function chamsApply(plr)
	if not plr.Character then
		return
	end
	local previous = ChamsData[plr]
	if previous and previous.Parent == plr.Character and previous.Adornee == plr.Character then
		return
	end
	if previous then
		pcall(function()
			previous:Destroy()
		end)
		ChamsData[plr] = nil
	end
	local ok, hl = pcall(Instance.new, "Highlight")
	if not ok or not hl then
		return
	end
	pcall(function()
		hl.FillColor = Redx.State.ESP.color
		hl.OutlineColor = Color3.fromRGB(255, 255, 255)
		hl.FillTransparency = 0.5
		hl.OutlineTransparency = 0
		hl.Adornee = plr.Character
		hl.Parent = plr.Character
	end)
	ChamsData[plr] = hl
end

local function chamsRemove(plr)
	local hl = ChamsData[plr]
	if hl then
		pcall(function()
			hl:Destroy()
		end)
		ChamsData[plr] = nil
	end
end

local function chamsClear()
	for plr in pairs(ChamsData) do
		chamsRemove(plr)
	end
end

local function isTeammate(plr)
	if not Redx.State.ESP.teamCheck then
		return false
	end
	if not LocalPlayer then
		return false
	end
	local myTeam = LocalPlayer.Team
	if not myTeam then
		return false
	end
	return plr.Team == myTeam
end

local function getSkeletonPairs(char)
	local R15 = {
		{ "Head", "UpperTorso" },
		{ "UpperTorso", "LowerTorso" },
		{ "UpperTorso", "LeftUpperArm" },
		{ "LeftUpperArm", "LeftLowerArm" },
		{ "LeftLowerArm", "LeftHand" },
		{ "UpperTorso", "RightUpperArm" },
		{ "RightUpperArm", "RightLowerArm" },
		{ "RightLowerArm", "RightHand" },
		{ "LowerTorso", "LeftUpperLeg" },
		{ "LeftUpperLeg", "LeftLowerLeg" },
		{ "LeftLowerLeg", "LeftFoot" },
		{ "LowerTorso", "RightUpperLeg" },
		{ "RightUpperLeg", "RightLowerLeg" },
		{ "RightLowerLeg", "RightFoot" }
	}
	local R6 = {
		{ "Head", "Torso" },
		{ "Torso", "Left Arm" },
		{ "Torso", "Right Arm" },
		{ "Torso", "Left Leg" },
		{ "Torso", "Right Leg" }
	}
	local result = {}
	local hasR15 = char:FindFirstChild("UpperTorso") ~= nil
	local list = hasR15 and R15 or R6
	for i = 1, #list do
		local pair = list[i]
		local a = char:FindFirstChild(pair[1])
		local b = char:FindFirstChild(pair[2])
		if a and b then
			table.insert(result, { a, b })
		end
	end
	return result
end

Redx.ESP.Toggle = function(s)
	local b = s and true or false
	Redx.State.ESP.enabled = b
	if not b then
		for plr in pairs(ESPData) do
			espDestroy(plr)
		end
		chamsClear()
	end
	emitSync("ESP.enabled", b)
	emitSync("Keybinds")
end

Redx.ESP.SetColor = function(c)
	if isColor3Value(c) then
		Redx.State.ESP.color = c
		emitSync("ESP.color", c)
	end
end

Redx.ESP.SetDistance = function(m)
	local v = clamp(tonumber(m) or 500, 50, 2000)
	Redx.State.ESP.distance = v
	emitSync("ESP.distance", v)
end

Redx.ESP.SetThickness = function(px)
	local v = clamp(tonumber(px) or 3, 1, 6)
	Redx.State.ESP.thickness = v
	emitSync("ESP.thickness", v)
end

Redx.ESP.SetTextSize = function(sz)
	local v = clamp(tonumber(sz) or 14, 10, 24)
	Redx.State.ESP.textSize = v
	for _, d in pairs(ESPData) do
		if d.name then
			dSet(d.name, "Size", v)
		end
		if d.distance then
			dSet(d.distance, "Size", math.max(10, v - 2))
		end
		if d.healthText then
			dSet(d.healthText, "Size", math.max(10, v - 3))
		end
	end
	emitSync("ESP.textSize", v)
end

Redx.ESP.SetBoxStyle = function(style)
	if style == "normal" or style == "corner" or style == "filled" then
		Redx.State.ESP.boxStyle = style
		emitSync("ESP.boxStyle", style)
	end
end

Redx.ESP.SetTracerStyle = function(style)
	if style == "base" or style == "top" or style == "mouse" then
		Redx.State.ESP.tracerStyle = style
		emitSync("ESP.tracerStyle", style)
	end
end

Redx.ESP.SetTeamCheck = function(s)
	local b = s and true or false
	Redx.State.ESP.teamCheck = b
	emitSync("ESP.teamCheck", b)
end

Redx.ESP.SetElement = function(el, s)
	local b = s and true or false
	local S = Redx.State.ESP
	if el == "box" then
		S.box = b
		emitSync("ESP.box", b)
	elseif el == "name" then
		S.name = b
		emitSync("ESP.name", b)
	elseif el == "showDistance" then
		S.showDistance = b
		emitSync("ESP.showDistance", b)
	elseif el == "health" then
		S.health = b
		emitSync("ESP.health", b)
	elseif el == "healthText" then
		S.healthText = b
		emitSync("ESP.healthText", b)
	elseif el == "healthBar" then
		S.healthBar = b
		emitSync("ESP.healthBar", b)
	elseif el == "headDot" then
		S.headDot = b
		emitSync("ESP.headDot", b)
	elseif el == "skeleton" then
		S.skeleton = b
		emitSync("ESP.skeleton", b)
	elseif el == "tracer" then
		S.tracer = b
		emitSync("ESP.tracer", b)
	elseif el == "chams" then
		S.chams = b
		if not b then
			chamsClear()
		end
		emitSync("ESP.chams", b)
		emitSync("Keybinds")
	elseif el == "aimLine" then
		S.aimLine = b
		Redx.State.Aimbot.drawAimLine = b
		emitSync("Aimbot.drawAimLine", b)
	end
end

trackConn(RunService.RenderStepped:Connect(function()
	local S = Redx.State.ESP
	if not S.enabled then
		return
	end
	camera = workspace.CurrentCamera or camera
	if not camera then
		return
	end

	local camPos = camera.CFrame.Position
	local centerV = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local char = plr.Character
			local hum = char and char:FindFirstChildOfClass("Humanoid")
			local hrp = char and char:FindFirstChild("HumanoidRootPart")

			if char and hum and hum.Health > 0 and hrp and not isTeammate(plr) then
				local dist = (hrp.Position - camPos).Magnitude
				if dist <= S.distance then
					local d = ESPData[plr] or espNewFor(plr)
					local pos, onScreen = camera:WorldToViewportPoint(hrp.Position)
					if onScreen then
						local headP = camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 1.5, 0))
						local feetP = camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
						local h = math.abs(feetP.Y - headP.Y)
						local w = h * 0.55
						local x, y = pos.X - w / 2, pos.Y - h / 2
						local col = S.color
						local th = S.thickness

						if S.box then
							if S.boxStyle == "normal" then
								dSet(d.box, "Size", Vector2.new(w, h))
								dSet(d.box, "Position", Vector2.new(x, y))
								dSet(d.box, "Color", col)
								dSet(d.box, "Thickness", th)
								dSet(d.box, "Visible", true)
							else
								dSet(d.box, "Visible", false)
							end

							if S.boxStyle == "filled" then
								dSet(d.boxFill, "Size", Vector2.new(w, h))
								dSet(d.boxFill, "Position", Vector2.new(x, y))
								dSet(d.boxFill, "Color", col)
								dSet(d.boxFill, "Visible", true)
								dSet(d.box, "Size", Vector2.new(w, h))
								dSet(d.box, "Position", Vector2.new(x, y))
								dSet(d.box, "Color", col)
								dSet(d.box, "Thickness", th)
								dSet(d.box, "Visible", true)
							else
								dSet(d.boxFill, "Visible", false)
							end

							if S.boxStyle == "corner" then
								local cLen = math.min(w, h) * 0.25
								local function drawSeg(line, x1, y1, x2, y2)
									dSet(line, "From", Vector2.new(x1, y1))
									dSet(line, "To", Vector2.new(x2, y2))
									dSet(line, "Color", col)
									dSet(line, "Thickness", th)
									dSet(line, "Visible", true)
								end
								drawSeg(d.corners[1], x, y, x + cLen, y)
								drawSeg(d.corners[2], x, y, x, y + cLen)
								drawSeg(d.corners[3], x + w, y, x + w - cLen, y)
								drawSeg(d.corners[4], x + w, y, x + w, y + cLen)
								drawSeg(d.corners[5], x, y + h, x + cLen, y + h)
								drawSeg(d.corners[6], x, y + h, x, y + h - cLen)
								drawSeg(d.corners[7], x + w, y + h, x + w - cLen, y + h)
								drawSeg(d.corners[8], x + w, y + h, x + w, y + h - cLen)
							else
								for i = 1, #d.corners do
									dSet(d.corners[i], "Visible", false)
								end
							end
						else
							dSet(d.box, "Visible", false)
							dSet(d.boxFill, "Visible", false)
							for i = 1, #d.corners do
								dSet(d.corners[i], "Visible", false)
							end
						end

						if S.name then
							local txt = plr.Name
							if S.showDistance then
								txt = txt .. " [" .. math.floor(dist) .. "m]"
							end
							dSet(d.name, "Text", txt)
							dSet(d.name, "Size", S.textSize)
							dSet(d.name, "Position", Vector2.new(pos.X, y - S.textSize - 4))
							dSet(d.name, "Color", col)
							dSet(d.name, "Visible", true)
						else
							dSet(d.name, "Visible", false)
						end

						if S.health and S.healthBar then
							local hpPct = clamp(hum.Health / math.max(1, hum.MaxHealth), 0, 1)
							local barW = 3
							local barX = x - barW - 4
							local barY = y
							local barH = h
							dSet(d.healthBg, "Size", Vector2.new(barW, barH))
							dSet(d.healthBg, "Position", Vector2.new(barX, barY))
							dSet(d.healthBg, "Color", Color3.fromRGB(0, 0, 0))
							dSet(d.healthBg, "Visible", true)
							local fillH = barH * hpPct
							local fillY = barY + (barH - fillH)
							local hpCol = Color3.fromRGB(80, 220, 80)
							if hpPct < 0.3 then
								hpCol = Color3.fromRGB(230, 40, 40)
							elseif hpPct < 0.6 then
								hpCol = Color3.fromRGB(230, 200, 40)
							end
							dSet(d.healthFill, "Size", Vector2.new(barW, fillH))
							dSet(d.healthFill, "Position", Vector2.new(barX, fillY))
							dSet(d.healthFill, "Color", hpCol)
							dSet(d.healthFill, "Visible", true)
						else
							dSet(d.healthBg, "Visible", false)
							dSet(d.healthFill, "Visible", false)
						end

						if S.health and S.healthText then
							dSet(d.healthText, "Text", tostring(math.floor(hum.Health)) .. "/" .. tostring(math.floor(hum.MaxHealth)))
							dSet(d.healthText, "Position", Vector2.new(x + w + 16, y + h - 14))
							dSet(d.healthText, "Color", Color3.fromRGB(220, 220, 220))
							dSet(d.healthText, "Visible", true)
						else
							dSet(d.healthText, "Visible", false)
						end

						if S.headDot then
							local head = char:FindFirstChild("Head")
							if head then
								local hp, hs = camera:WorldToViewportPoint(head.Position)
								if hs then
									dSet(d.headDot, "Position", Vector2.new(hp.X, hp.Y))
									dSet(d.headDot, "Radius", 3)
									dSet(d.headDot, "Color", col)
									dSet(d.headDot, "Visible", true)
								else
									dSet(d.headDot, "Visible", false)
								end
							else
								dSet(d.headDot, "Visible", false)
							end
						else
							dSet(d.headDot, "Visible", false)
						end

						if S.tracer then
							local fromP
							if S.tracerStyle == "base" then
								fromP = Vector2.new(centerV.X, camera.ViewportSize.Y)
							elseif S.tracerStyle == "top" then
								fromP = Vector2.new(centerV.X, 0)
							else
								local okM, m = pcall(function()
									return UserInputService:GetMouseLocation()
								end)
								fromP = okM and Vector2.new(m.X, m.Y) or centerV
							end
							dSet(d.tracer, "From", fromP)
							dSet(d.tracer, "To", Vector2.new(pos.X, pos.Y + h / 2))
							dSet(d.tracer, "Color", col)
							dSet(d.tracer, "Thickness", th)
							dSet(d.tracer, "Visible", true)
						else
							dSet(d.tracer, "Visible", false)
						end

						if S.skeleton then
							local pairsList = getSkeletonPairs(char)
							local used = 0
							for i = 1, #pairsList do
								if i > #d.skel then
									break
								end
								local pair = pairsList[i]
								local p1, s1 = camera:WorldToViewportPoint(pair[1].Position)
								local p2, s2 = camera:WorldToViewportPoint(pair[2].Position)
								if s1 and s2 then
									used = used + 1
									local line = d.skel[used]
									dSet(line, "From", Vector2.new(p1.X, p1.Y))
									dSet(line, "To", Vector2.new(p2.X, p2.Y))
									dSet(line, "Color", col)
									dSet(line, "Thickness", th)
									dSet(line, "Visible", true)
								end
							end
							for i = used + 1, #d.skel do
								dSet(d.skel[i], "Visible", false)
							end
						else
							for i = 1, #d.skel do
								dSet(d.skel[i], "Visible", false)
							end
						end
					else
						espSetVisible(d, false)
					end

					if S.chams then
						chamsApply(plr)
						if ChamsData[plr] then
							pcall(function()
								ChamsData[plr].FillColor = S.color
							end)
						end
					else
						chamsRemove(plr)
					end
				else
					local d = ESPData[plr]
					if d then
						espSetVisible(d, false)
					end
					chamsRemove(plr)
				end
			else
				local d = ESPData[plr]
				if d then
					espSetVisible(d, false)
				end
				chamsRemove(plr)
			end
		end
	end
end))

trackConn(Players.PlayerRemoving:Connect(function(plr)
	espDestroy(plr)
	chamsRemove(plr)
end))

Redx.Godmode = {}

local godLoop = nil
local godRespawn = nil
local GodmodeRestore = setmetatable({}, { __mode = "k" })

local function godApply()
	if not LocalPlayer then
		return
	end
	local char = LocalPlayer.Character
	if not char then
		return
	end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if hum then
		if not GodmodeRestore[hum] then
			GodmodeRestore[hum] = { maxHealth = hum.MaxHealth, health = hum.Health }
		end
		pcall(function()
			hum.MaxHealth = 1e9
			hum.Health = 1e9
		end)
	end
end

Redx.Godmode.Toggle = function(s)
	local b = s and true or false
	Redx.State.Godmode.enabled = b
	if b then
		godApply()
		if godLoop then
			pcall(function()
				godLoop:Disconnect()
			end)
		end
		godLoop = RunService.Heartbeat:Connect(function()
			if Redx.State.Godmode.enabled then
				godApply()
			end
		end)
		if godRespawn then
			pcall(function()
				godRespawn:Disconnect()
			end)
		end
		if LocalPlayer and LocalPlayer.CharacterAdded then
			godRespawn = LocalPlayer.CharacterAdded:Connect(function()
				waitTask(0.5)
				if Redx.State.Godmode.enabled then
					godApply()
				end
			end)
		end
	else
		if godLoop then
			pcall(function()
				godLoop:Disconnect()
			end)
			godLoop = nil
		end
		if godRespawn then
			pcall(function()
				godRespawn:Disconnect()
			end)
			godRespawn = nil
		end
		for hum, original in pairs(GodmodeRestore) do
			pcall(function()
				hum.MaxHealth = original.maxHealth
				if hum.Health > 0 then
					hum.Health = math.min(original.health, original.maxHealth)
				end
			end)
			GodmodeRestore[hum] = nil
		end
	end
	emitSync("Godmode.enabled", b)
	emitSync("Keybinds")
end

Redx.Performance = {}
local PerformanceRestore = {}

local function applyPerformance()
	local state = Redx.State.Performance
	pcall(function()
		local rendering = settings().Rendering
		if state.fpsBoost or state.lowTextures then
			if PerformanceRestore.quality == nil then
				PerformanceRestore.quality = rendering.QualityLevel
			end
			rendering.QualityLevel = 1
		elseif PerformanceRestore.quality ~= nil then
			rendering.QualityLevel = PerformanceRestore.quality
			PerformanceRestore.quality = nil
		end
	end)
	pcall(function()
		if state.fpsBoost or state.removeShadows then
			if PerformanceRestore.shadows == nil then
				PerformanceRestore.shadows = Lighting.GlobalShadows
			end
			Lighting.GlobalShadows = false
		elseif PerformanceRestore.shadows ~= nil then
			Lighting.GlobalShadows = PerformanceRestore.shadows
			PerformanceRestore.shadows = nil
		end
	end)
end

Redx.Performance.FPSBoost = function(s)
	local b = s and true or false
	Redx.State.Performance.fpsBoost = b
	applyPerformance()
	emitSync("Performance.fpsBoost", b)
	emitSync("Keybinds")
end

Redx.Performance.RemoveShadows = function(s)
	local b = s and true or false
	Redx.State.Performance.removeShadows = b
	applyPerformance()
	emitSync("Performance.removeShadows", b)
end

Redx.Performance.LowTextures = function(s)
	local b = s and true or false
	Redx.State.Performance.lowTextures = b
	applyPerformance()
	emitSync("Performance.lowTextures", b)
end

Redx.Bypass = {}

Redx.Bypass.Stealth = function(s)
	local b = s and true or false
	Redx.State.Bypass.stealth = b
	if b and ScreenGui then
		pcall(function()
			ScreenGui.Name = "UI_" .. tostring(math.random(100000, 999999))
		end)
	elseif ScreenGui then
		pcall(function()
			ScreenGui.Name = "RedxPainelGui"
		end)
	end
	emitSync("Bypass.stealth", b)
end

Redx.Bypass.UnlockFPS = function(s)
	local b = s and true or false
	Redx.State.Bypass.unlockFPS = b
	pcall(function()
		if type(setfpscap) == "function" then
			setfpscap(b and 240 or 60)
		end
	end)
	emitSync("Bypass.unlockFPS", b)
end

local ResolvedLogoAsset = nil
local LogoListeners = {}

local function notifyLogoUpdated(asset)
	ResolvedLogoAsset = asset
	for i = 1, #LogoListeners do
		pcall(LogoListeners[i], asset)
	end
end

local function resolveLogoUrl(rawUrl)
	if type(rawUrl) ~= "string" then
		return
	end
	local trimmed = string.gsub(rawUrl, "^%s*(.-)%s*$", "%1")
	if trimmed == "" then
		return
	end
	if string.match(trimmed, "^rbxassetid://") then
		notifyLogoUpdated(trimmed)
		return
	end
	if string.match(trimmed, "^%d+$") then
		notifyLogoUpdated("rbxassetid://" .. trimmed)
		return
	end
	local rbxId = string.match(trimmed, "roblox%.com/.+id=(%d+)") or string.match(trimmed, "asset/(%d+)")
	if rbxId then
		notifyLogoUpdated("rbxassetid://" .. rbxId)
		return
	end
	if string.match(trimmed, "^https?://") then
		spawnTask(function()
			local customAssetFn = (type(getcustomasset) == "function" and getcustomasset) or (type(getsynasset) == "function" and getsynasset)
			if customAssetFn and type(writefile) == "function" then
				local ok, imgBytes = pcall(function()
					return game:HttpGet(trimmed)
				end)
				if ok and type(imgBytes) == "string" and #imgBytes > 0 then
					local fileName = "RedxPainel_Logo.png"
					local wrote = pcall(writefile, fileName, imgBytes)
					if wrote then
						local assetOk, assetId = pcall(customAssetFn, fileName)
						if assetOk and type(assetId) == "string" and assetId ~= "" then
							notifyLogoUpdated(assetId)
							return
						end
					end
				end
			end
			notifyLogoUpdated(trimmed)
		end)
	end
end

local function createBrandLogo(parent, sizePx, maskBgColor)
	local maskColor = maskBgColor or Theme.SidebarBg
	local holder = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(0, sizePx, 0, sizePx),
		Parent = parent
	})

	local vectorGroup = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.fromScale(1, 1),
		Parent = holder
	})

	create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.24, 0.41),
		Size = UDim2.fromScale(0.13, 0.37),
		ZIndex = 1,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.37, 0.54),
		Size = UDim2.fromScale(0.11, 0.11),
		Rotation = 45,
		ZIndex = 1,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.39, 0.72),
		Size = UDim2.fromScale(0.34, 0.20),
		Rotation = -43,
		ZIndex = 2,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = Color3.fromRGB(238, 238, 240),
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.62, 0.70),
		Size = UDim2.fromScale(0.52, 0.115),
		Rotation = 46,
		ZIndex = 3,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = Color3.fromRGB(238, 238, 240),
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.59, 0.71),
		Size = UDim2.fromScale(0.56, 0.10),
		Rotation = -42,
		ZIndex = 3,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.38, 0.36),
		Size = UDim2.fromScale(0.58, 0.18),
		ZIndex = 4,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.30, 0.87),
		Size = UDim2.fromScale(0.68, 0.13),
		ZIndex = 4,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.77, 0.40),
		Size = UDim2.fromScale(0.34, 0.085),
		Rotation = -40,
		ZIndex = 5,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.76, 0.32),
		Size = UDim2.fromScale(0.36, 0.075),
		Rotation = -32,
		ZIndex = 6,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.60, 0.51),
		Size = UDim2.fromScale(0.14, 0.14),
		Rotation = 45,
		ZIndex = 6,
		Parent = vectorGroup
	})

	local rLoop = create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.35, 0.18),
		Size = UDim2.fromScale(0.31, 0.34),
		ZIndex = 7,
		Parent = vectorGroup
	})
	addCorner(rLoop, math.max(3, math.floor(sizePx * 0.17)))

	create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.08, 0.18),
		Size = UDim2.fromScale(0.44, 0.11),
		ZIndex = 7,
		Parent = vectorGroup
	})

	create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.06, 0.29),
		Size = UDim2.fromScale(0.22, 0.22),
		Rotation = 45,
		ZIndex = 8,
		Parent = vectorGroup
	})

	local rHole = create("Frame", {
		BackgroundColor3 = maskColor,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.22, 0.29),
		Size = UDim2.fromScale(0.33, 0.12),
		ZIndex = 8,
		Parent = vectorGroup
	})
	addCorner(rHole, math.max(2, math.floor(sizePx * 0.06)))

	create("Frame", {
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Position = UDim2.fromScale(0.24, 0.41),
		Size = UDim2.fromScale(0.28, 0.11),
		ZIndex = 9,
		Parent = vectorGroup
	})

	local img = create("ImageLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.fromScale(1, 1),
		ScaleType = Enum.ScaleType.Fit,
		Image = ResolvedLogoAsset or "",
		Visible = ResolvedLogoAsset ~= nil and ResolvedLogoAsset ~= "",
		ZIndex = 12,
		Parent = holder
	})

	if ResolvedLogoAsset and ResolvedLogoAsset ~= "" then
		vectorGroup.Visible = false
	end

	table.insert(LogoListeners, function(newAsset)
		if newAsset and newAsset ~= "" then
			img.Image = newAsset
			img.Visible = true
			vectorGroup.Visible = false
		else
			img.Visible = false
			vectorGroup.Visible = true
		end
	end)

	return holder
end

local function createIcon(parent, kind, sizePx, color)
	local sz = sizePx or 14
	local col = color or Theme.Accent
	local box = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(0, sz, 0, sz),
		Parent = parent
	})

	if kind == "aimbot" then
		local ring = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.76, 0.76),
			BackgroundTransparency = 1,
			Parent = box
		})
		addCorner(ring, 99)
		addStroke(ring, col, 1.5)
		local dot = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.new(0, 3, 0, 3),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(dot, 99)
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0),
			Position = UDim2.fromScale(0.5, 0),
			Size = UDim2.new(0, 1.5, 0.26, 0),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 1),
			Position = UDim2.fromScale(0.5, 1),
			Size = UDim2.new(0, 1.5, 0.26, 0),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Position = UDim2.fromScale(0, 0.5),
			Size = UDim2.new(0.26, 0, 0, 1.5),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		create("Frame", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.fromScale(1, 0.5),
			Size = UDim2.new(0.26, 0, 0, 1.5),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
	elseif kind == "memory" then
		local body = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.56),
			Size = UDim2.fromScale(0.52, 0.58),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(body, 4)
		local head = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.22),
			Size = UDim2.fromScale(0.34, 0.24),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(head, 3)
		for idx = 1, 3 do
			local yPos = 0.34 + (idx - 1) * 0.20
			create("Frame", {
				AnchorPoint = Vector2.new(0, 0.5),
				Position = UDim2.fromScale(0.04, yPos),
				Size = UDim2.fromScale(0.24, 0.10),
				BackgroundColor3 = col,
				BorderSizePixel = 0,
				Parent = box
			})
			create("Frame", {
				AnchorPoint = Vector2.new(1, 0.5),
				Position = UDim2.fromScale(0.96, yPos),
				Size = UDim2.fromScale(0.24, 0.10),
				BackgroundColor3 = col,
				BorderSizePixel = 0,
				Parent = box
			})
		end
	elseif kind == "visuals" then
		local outer = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.92, 0.56),
			BackgroundTransparency = 1,
			Parent = box
		})
		addCorner(outer, 99)
		addStroke(outer, col, 1.5)
		local pupil = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.30, 0.30),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(pupil, 99)
	elseif kind == "misc" then
		for idx = 0, 3 do
			local tooth = create("Frame", {
				AnchorPoint = Vector2.new(0.5, 0.5),
				Position = UDim2.fromScale(0.5, 0.5),
				Size = UDim2.fromScale(0.92, 0.24),
				Rotation = idx * 45,
				BackgroundColor3 = col,
				BorderSizePixel = 0,
				Parent = box
			})
			addCorner(tooth, 2)
		end
		local ring = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.68, 0.68),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(ring, 99)
		local hole = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.30, 0.30),
			BackgroundColor3 = Theme.CategoryBg,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(hole, 99)
	elseif kind == "player" then
		local head = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0),
			Position = UDim2.fromScale(0.5, 0.08),
			Size = UDim2.fromScale(0.38, 0.38),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(head, 99)
		local shoulders = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 1),
			Position = UDim2.fromScale(0.5, 0.92),
			Size = UDim2.fromScale(0.76, 0.38),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(shoulders, 5)
	elseif kind == "shield" then
		local sh = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.72, 0.84),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(sh, 5)
		local inner = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.38, 0.48),
			BackgroundColor3 = Theme.CardBg,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(inner, 3)
	elseif kind == "clock" then
		local circle = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.92, 0.92),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(circle, 99)
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 1),
			Position = UDim2.fromScale(0.5, 0.52),
			Size = UDim2.new(0, 1.6, 0.30, 0),
			BackgroundColor3 = Theme.BadgeBg,
			BorderSizePixel = 0,
			Parent = circle
		})
		create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Position = UDim2.fromScale(0.5, 0.52),
			Size = UDim2.new(0.26, 0, 0, 1.6),
			Rotation = 28,
			BackgroundColor3 = Theme.BadgeBg,
			BorderSizePixel = 0,
			Parent = circle
		})
	end

	return box
end

local UIState = {
	Windows = {
		AimWindow = true,
		ESPWindow = true,
		ESPAdvWindow = false,
		GodmodeWindow = false,
		PerfWindow = false,
		BypassWindow = false,
		KeybindOverlay = true,
		TimerPill = true,
		PlayerCountPill = true
	},
	SidebarTabsEnabled = {
		Aimbot = true,
		Visuals = true,
		ESPConfig = true,
		Player = true,
		Performance = true,
		Personalizacao = true,
		Config = true
	},
	Keybinds = {
		AimMagnet = Enum.KeyCode.F2,
		FOVToggle = Enum.KeyCode.F3,
		ESPEnable = Enum.KeyCode.F5,
		ChamsToggle = Enum.KeyCode.F6,
		GodmodeToggle = Enum.KeyCode.F8,
		FPSBoostToggle = Enum.KeyCode.F9,
		MenuToggle = Enum.KeyCode.RightShift
	},
	KeybindMeta = {
		{
			Id = "AimMagnet",
			Label = "Aim Magnet",
			Get = function()
				return Redx.State.Aimbot.enabled
			end,
			Toggle = function()
				Redx.Aimbot.Toggle(not Redx.State.Aimbot.enabled)
			end
		},
		{
			Id = "FOVToggle",
			Label = "Mostrar FOV",
			Get = function()
				return Redx.State.FOV.enabled
			end,
			Toggle = function()
				Redx.FOV.Toggle(not Redx.State.FOV.enabled)
			end
		},
		{
			Id = "ESPEnable",
			Label = "Ativar ESP",
			Get = function()
				return Redx.State.ESP.enabled
			end,
			Toggle = function()
				Redx.ESP.Toggle(not Redx.State.ESP.enabled)
			end
		},
		{
			Id = "ChamsToggle",
			Label = "Chams (glow 3D)",
			Get = function()
				return Redx.State.ESP.chams
			end,
			Toggle = function()
				Redx.ESP.SetElement("chams", not Redx.State.ESP.chams)
			end
		},
		{
			Id = "GodmodeToggle",
			Label = "Godmode",
			Get = function()
				return Redx.State.Godmode.enabled
			end,
			Toggle = function()
				Redx.Godmode.Toggle(not Redx.State.Godmode.enabled)
			end
		},
		{
			Id = "FPSBoostToggle",
			Label = "FPS Boost",
			Get = function()
				return Redx.State.Performance.fpsBoost
			end,
			Toggle = function()
				Redx.Performance.FPSBoost(not Redx.State.Performance.fpsBoost)
			end
		}
	},
	MainScalePct = 100,
	FloatScalePct = 100
}

local function setWindowActive(winKey, state)
	local b = state and true or false
	UIState.Windows[winKey] = b
	emitSync("Window." .. winKey, b)
end

local function setSidebarTabEnabled(tabId, state)
	local b = state and true or false
	UIState.SidebarTabsEnabled[tabId] = b
	emitSync("SidebarTab." .. tabId, b)
end

local function formatKeyCode(kc)
	if not kc then
		return "None"
	end
	local name = kc.Name or tostring(kc)
	local shortMap = {
		RightShift = "RShift",
		LeftShift = "LShift",
		RightControl = "RCtrl",
		LeftControl = "LCtrl",
		RightAlt = "RAlt",
		LeftAlt = "LAlt",
		Backspace = "Back",
		Return = "Enter"
	}
	return shortMap[name] or name
end

local ViewportSize = Vector2.new(1366, 768)
pcall(function()
	if workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize.X > 0 then
		ViewportSize = workspace.CurrentCamera.ViewportSize
	end
end)
local InitialGlobalScale = clamp(math.min(ViewportSize.X / 1320, ViewportSize.Y / 720), 0.56, 1.0)

local ActivePointerAction = nil
local BindingTargetId = nil

local function isPointerInput(input)
	return input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch
end

local function isMoveInput(input)
	return input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch
end

local function attachDraggable(handle, targetFrame)
	local dragging = false
	local dragInput = nil
	local startMouse = Vector2.new(0, 0)
	local startPos = UDim2.new(0, 0, 0, 0)

	trackConn(handle.InputBegan:Connect(function(input)
		if not isPointerInput(input) then
			return
		end
		if ActivePointerAction ~= nil then
			return
		end
		dragging = true
		dragInput = input
		ActivePointerAction = "drag"
		startMouse = input.Position
		startPos = targetFrame.Position
	end))

	trackConn(UserInputService.InputChanged:Connect(function(input)
		if dragging and isMoveInput(input) then
			if input.UserInputType == Enum.UserInputType.Touch and dragInput and input ~= dragInput then
				return
			end
			local delta = input.Position - startMouse
			targetFrame.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end))

	trackConn(UserInputService.InputEnded:Connect(function(input)
		if dragging and (input == dragInput or isPointerInput(input)) then
			dragging = false
			dragInput = nil
			if ActivePointerAction == "drag" then
				ActivePointerAction = nil
			end
		end
	end))
end

local AllCollapsibleEntries = {}
local function registerCollapsible(entry)
	table.insert(AllCollapsibleEntries, entry)
end

local function createHudPill(name, posUDim, widthPx, iconKind, defaultText, winKey)
	local pill = create("Frame", {
		Name = name,
		AnchorPoint = Vector2.new(0.5, 0),
		Position = posUDim,
		Size = UDim2.new(0, widthPx, 0, 24),
		BackgroundColor3 = Theme.BadgeBg,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Visible = UIState.Windows[winKey],
		ZIndex = 20,
		Parent = ScreenGui
	})
	addCorner(pill, 999)
	addStroke(pill, Theme.PillStroke, 1.2)
	local pillScale = create("UIScale", {
		Scale = InitialGlobalScale,
		Parent = pill
	})

	local iconWrap = create("TextButton", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 6, 0.5, -7),
		Size = UDim2.new(0, 14, 0, 14),
		Text = "",
		AutoButtonColor = false,
		ZIndex = 22,
		Parent = pill
	})
	createIcon(iconWrap, iconKind, 14, Theme.Accent)

	local label = create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 24, 0, 0),
		Size = UDim2.new(1, -30, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = defaultText,
		TextSize = 11,
		TextColor3 = Theme.Accent,
		TextXAlignment = Enum.TextXAlignment.Center,
		ZIndex = 21,
		Parent = pill
	})

	local collapsed = false
	local function setCollapsedState(c)
		collapsed = c
		label.Visible = not collapsed
		tween(pill, 0.15, {
			Size = collapsed and UDim2.new(0, 26, 0, 24) or UDim2.new(0, widthPx, 0, 24)
		})
	end

	iconWrap.MouseButton1Click:Connect(function()
		setCollapsedState(not collapsed)
	end)

	attachDraggable(pill, pill)
	registerCollapsible({ setCollapsed = setCollapsedState })

	onSync("Window." .. winKey, function(vis)
		pill.Visible = vis
	end)

	return pill, label, pillScale
end

local _, TimerLabel = createHudPill("TimerPill", UDim2.new(0.46, 0, 0.115, 0), 74, "clock", "00:00", "TimerPill")
local _, PlayerCountLabel = createHudPill("PlayerCountPill", UDim2.new(0.498, 0, 0.18, 0), 52, "player", "0", "PlayerCountPill")

local StartClock = os.clock and os.clock() or 0
spawnTask(function()
	while ScreenGui and ScreenGui.Parent do
		waitTask(1)
		if not ScreenGui or not ScreenGui.Parent then
			break
		end
		local now = os.clock and os.clock() or (StartClock + 1)
		local elapsed = math.max(0, math.floor(now - StartClock))
		local mins = math.floor(elapsed / 60)
		local secs = elapsed % 60
		TimerLabel.Text = string.format("%02d:%02d", mins, secs)
		local others = math.max(0, #Players:GetPlayers() - 1)
		PlayerCountLabel.Text = tostring(others)
	end
end)

local function buildToggleControl(parent, labelText, getter, setter, syncChannel)
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 24),
		Parent = parent
	})

	local btnRow = create("TextButton", {
		BackgroundTransparency = 1,
		Size = UDim2.fromScale(1, 1),
		Text = "",
		AutoButtonColor = false,
		Parent = row
	})

	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1, -40, 1, 0),
		Font = Enum.Font.GothamMedium,
		Text = labelText,
		TextColor3 = Theme.TextSecondary,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = row
	})

	local init = getter() and true or false
	local switchBg = create("Frame", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 32, 0, 16),
		BackgroundColor3 = init and Theme.Accent or Theme.ToggleOff,
		BorderSizePixel = 0,
		Parent = row
	})
	addCorner(switchBg, 999)

	local knob = create("Frame", {
		AnchorPoint = Vector2.new(0, 0.5),
		Position = init and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
		Size = UDim2.new(0, 12, 0, 12),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		Parent = switchBg
	})
	addCorner(knob, 999)

	local function refresh(val)
		local b = val and true or false
		tween(switchBg, 0.14, { BackgroundColor3 = b and Theme.Accent or Theme.ToggleOff })
		tween(knob, 0.14, { Position = b and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0) })
	end

	btnRow.MouseButton1Click:Connect(function()
		local nextVal = not getter()
		setter(nextVal)
		refresh(getter())
	end)

	if syncChannel then
		onSync(syncChannel, refresh)
	end

	return row
end

local function buildWindowToggleRow(parent, labelText, winKey)
	return buildToggleControl(
		parent,
		labelText,
		function()
			return UIState.Windows[winKey]
		end,
		function(v)
			setWindowActive(winKey, v)
		end,
		"Window." .. winKey
	)
end

local function buildKeybindControl(parent, labelText, keyId, activeGetter, activeToggler)
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 24),
		Parent = parent
	})

	local isOn = activeGetter and activeGetter() or false
	local labelBtn = create("TextButton", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1, -56, 1, 0),
		Font = Enum.Font.GothamMedium,
		Text = labelText,
		TextColor3 = isOn and Theme.Accent or Theme.TextSecondary,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		AutoButtonColor = false,
		Parent = row
	})

	local keyBtn = create("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 50, 0, 18),
		BackgroundColor3 = Theme.BadgeBg,
		BorderSizePixel = 0,
		Font = Enum.Font.GothamMedium,
		Text = formatKeyCode(UIState.Keybinds[keyId]),
		TextColor3 = Theme.TextPrimary,
		TextSize = 10,
		AutoButtonColor = false,
		Parent = row
	})
	addCorner(keyBtn, 4)
	local st = addStroke(keyBtn, Theme.BadgeStroke, 1)

	labelBtn.MouseButton1Click:Connect(function()
		if activeToggler then
			activeToggler()
		end
	end)

	keyBtn.MouseButton1Click:Connect(function()
		if BindingTargetId == keyId then
			BindingTargetId = nil
			emitSync("Keybinds")
			return
		end
		local prev = BindingTargetId
		BindingTargetId = keyId
		if prev ~= nil then
			emitSync("Keybinds")
		end
		keyBtn.Text = "..."
		keyBtn.TextColor3 = Theme.Accent
		st.Color = Theme.Accent
	end)

	onSync("Keybinds", function()
		if activeGetter then
			labelBtn.TextColor3 = activeGetter() and Theme.Accent or Theme.TextSecondary
		end
		if BindingTargetId ~= keyId then
			keyBtn.Text = formatKeyCode(UIState.Keybinds[keyId])
			keyBtn.TextColor3 = Theme.TextPrimary
			st.Color = Theme.BadgeStroke
		end
	end)

	return row
end

local function buildSliderControl(parent, labelText, getter, setter, minVal, maxVal, suffix, syncChannel)
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 24),
		Parent = parent
	})

	local labelWidth = 64
	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(0, labelWidth, 1, 0),
		Font = Enum.Font.GothamMedium,
		Text = labelText,
		TextColor3 = Theme.TextSecondary,
		TextSize = 10.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = row
	})

	local badgeReserve = 50
	local hitArea = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, labelWidth + 4, 0, 0),
		Size = UDim2.new(1, -(labelWidth + 4 + badgeReserve), 1, 0),
		Parent = row
	})

	local track = create("Frame", {
		AnchorPoint = Vector2.new(0, 0.5),
		Position = UDim2.new(0, 0, 0.5, 0),
		Size = UDim2.new(1, 0, 0, 5),
		BackgroundColor3 = Theme.SliderTrack,
		BorderSizePixel = 0,
		Parent = hitArea
	})
	addCorner(track, 999)

	local curVal = getter() or minVal
	local initAlpha = clamp((curVal - minVal) / math.max(1, (maxVal - minVal)), 0, 1)

	local fill = create("Frame", {
		Size = UDim2.new(initAlpha, 0, 1, 0),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		Parent = track
	})
	addCorner(fill, 999)

	local thumb = create("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(initAlpha, 0, 0.5, 0),
		Size = UDim2.new(0, 6, 0, 14),
		BackgroundColor3 = Color3.fromRGB(246, 246, 248),
		BorderSizePixel = 0,
		ZIndex = 3,
		Parent = track
	})
	addCorner(thumb, 3)

	local badge = create("Frame", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 46, 0, 18),
		BackgroundColor3 = Theme.BadgeBg,
		BorderSizePixel = 0,
		Parent = row
	})
	addCorner(badge, 4)
	addStroke(badge, Theme.BadgeStroke, 1)

	local badgeLabel = create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.fromScale(1, 1),
		Font = Enum.Font.GothamMedium,
		Text = tostring(curVal) .. (suffix or ""),
		TextColor3 = Theme.Accent,
		TextSize = 9.5,
		Parent = badge
	})

	local function refreshVisual(val)
		local v = val or getter() or minVal
		local a = clamp((v - minVal) / math.max(1, (maxVal - minVal)), 0, 1)
		fill.Size = UDim2.new(a, 0, 1, 0)
		thumb.Position = UDim2.new(a, 0, 0.5, 0)
		badgeLabel.Text = tostring(v) .. (suffix or "")
	end

	local function updateFromPositionX(screenX)
		local absPos = track.AbsolutePosition.X
		local absWidth = math.max(1, track.AbsoluteSize.X)
		local alpha = clamp((screenX - absPos) / absWidth, 0, 1)
		local stepped = math.floor(minVal + alpha * (maxVal - minVal) + 0.5)
		setter(stepped)
		refreshVisual(getter())
	end

	if syncChannel then
		onSync(syncChannel, refreshVisual)
	end

	local sliding = false
	local slideInput = nil

	trackConn(hitArea.InputBegan:Connect(function(input)
		if isPointerInput(input) then
			sliding = true
			slideInput = input
			ActivePointerAction = "slider"
			updateFromPositionX(input.Position.X)
		end
	end))

	trackConn(UserInputService.InputChanged:Connect(function(input)
		if sliding and isMoveInput(input) then
			if input.UserInputType == Enum.UserInputType.Touch and slideInput and input ~= slideInput then
				return
			end
			updateFromPositionX(input.Position.X)
		end
	end))

	trackConn(UserInputService.InputEnded:Connect(function(input)
		if sliding and (input == slideInput or isPointerInput(input)) then
			sliding = false
			slideInput = nil
			if ActivePointerAction == "slider" then
				ActivePointerAction = nil
			end
		end
	end))

	return row
end

local function buildSegmentedSelector(parent, labelText, getter, onPick, options, syncChannel)
	local hasLabel = (labelText ~= nil and labelText ~= "")
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, hasLabel and 42 or 24),
		Parent = parent
	})

	if hasLabel then
		create("TextLabel", {
			BackgroundTransparency = 1,
			Size = UDim2.new(1, 0, 0, 14),
			Font = Enum.Font.GothamMedium,
			Text = labelText,
			TextColor3 = Theme.TextSecondary,
			TextSize = 10.5,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = row
		})
	end

	local box = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, hasLabel and 17 or 0),
		Size = UDim2.new(1, 0, 0, 23),
		Parent = row
	})

	local btns = {}
	local n = #options
	for i = 1, n do
		local opt = options[i]
		local b = create("TextButton", {
			BackgroundColor3 = (getter() == opt.tag) and Theme.Accent or Theme.BadgeBg,
			BorderSizePixel = 0,
			Size = UDim2.new(1 / n, -3, 1, 0),
			Position = UDim2.new((i - 1) / n, (i == 1) and 0 or 2, 0, 0),
			Font = Enum.Font.GothamBold,
			Text = opt.label,
			TextColor3 = Theme.White,
			TextSize = 9.5,
			AutoButtonColor = false,
			Parent = box
		})
		addCorner(b, 4)
		addStroke(b, Theme.CategoryStroke, 1)
		btns[opt.tag] = b

		b.MouseButton1Click:Connect(function()
			onPick(opt.tag)
		end)
	end

	local function refresh()
		local current = getter()
		for tag, btn in pairs(btns) do
			btn.BackgroundColor3 = (tag == current) and Theme.Accent or Theme.BadgeBg
		end
	end

	if syncChannel then
		onSync(syncChannel, refresh)
	end

	return row
end

local function buildPaletteControl(parent, onPick)
	local wrap = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 68),
		Parent = parent
	})

	create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 14),
		Font = Enum.Font.GothamBold,
		Text = "Cor do ESP",
		TextColor3 = Theme.Accent,
		TextSize = 10.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = wrap
	})

	local palette = {
		Color3.fromRGB(230, 0, 28),
		Color3.fromRGB(255, 80, 80),
		Color3.fromRGB(255, 140, 0),
		Color3.fromRGB(255, 220, 0),
		Color3.fromRGB(100, 255, 100),
		Color3.fromRGB(0, 220, 220),
		Color3.fromRGB(0, 140, 255),
		Color3.fromRGB(180, 60, 255),
		Color3.fromRGB(255, 60, 200),
		Color3.fromRGB(255, 255, 255),
		Color3.fromRGB(120, 120, 120),
		Color3.fromRGB(0, 0, 0)
	}

	local gridBox = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 18),
		Size = UDim2.new(1, 0, 0, 48),
		Parent = wrap
	})

	local strokes = {}
	for i = 1, #palette do
		local col = palette[i]
		local colIdx = (i - 1) % 6
		local rowIdx = math.floor((i - 1) / 6)
		local sw = create("TextButton", {
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Position = UDim2.new(colIdx / 6, 1, 0, rowIdx * 24),
			Size = UDim2.new(1 / 6, -4, 0, 20),
			Text = "",
			AutoButtonColor = false,
			Parent = gridBox
		})
		addCorner(sw, 4)
		local st = addStroke(sw, Theme.CardStroke, 1.2)
		strokes[i] = { col = col, stroke = st }

		sw.MouseButton1Click:Connect(function()
			onPick(col)
		end)
	end

	local function refreshSelected()
		local cur = Redx.State.ESP.color
		for i = 1, #strokes do
			local item = strokes[i]
			local same = (item.col == cur)
			item.stroke.Color = same and Theme.White or Theme.CardStroke
			item.stroke.Thickness = same and 2 or 1.2
		end
	end

	refreshSelected()
	onSync("ESP.color", refreshSelected)

	return wrap
end

local function buildActionButton(parent, labelText, onClick, isAccent)
	local btn = create("TextButton", {
		BackgroundColor3 = isAccent and Theme.Accent or Theme.CategoryBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 25),
		Font = Enum.Font.GothamBold,
		Text = labelText,
		TextColor3 = isAccent and Theme.White or Theme.Accent,
		TextSize = 10.5,
		AutoButtonColor = false,
		Parent = parent
	})
	addCorner(btn, 5)
	addStroke(btn, isAccent and Theme.AccentHover or Theme.CategoryStroke, 1)
	btn.MouseButton1Click:Connect(onClick)
	return btn
end

local function attachResizeAndPullBar(windowFrame, scaleObj, baseWidth, baseHeight, minBarHeight, onBarCollapseChanged)
	local grip = create("Frame", {
		AnchorPoint = Vector2.new(1, 1),
		Position = UDim2.new(1, -1, 1, -1),
		Size = UDim2.new(0, 16, 0, 16),
		BackgroundTransparency = 1,
		ZIndex = 50,
		Parent = windowFrame
	})

	local line1 = create("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(1, -6, 1, -6),
		Size = UDim2.new(0, 9, 0, 1.5),
		Rotation = -45,
		BackgroundColor3 = Theme.TextMuted,
		BorderSizePixel = 0,
		ZIndex = 51,
		Parent = grip
	})
	create("Frame", {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.new(1, -4, 1, -4),
		Size = UDim2.new(0, 5, 0, 1.5),
		Rotation = -45,
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
		ZIndex = 51,
		Parent = grip
	})

	local bottomPullBar = create("Frame", {
		AnchorPoint = Vector2.new(0, 1),
		Position = UDim2.new(0, 0, 1, 0),
		Size = UDim2.new(1, -18, 0, 6),
		BackgroundTransparency = 1,
		ZIndex = 49,
		Parent = windowFrame
	})

	local resizing = false
	local resizeMode = nil
	local activeInput = nil
	local startMouse = Vector2.new(0, 0)
	local startScale = 1
	local startHeight = baseHeight

	local function beginResize(mode, input)
		resizing = true
		resizeMode = mode
		activeInput = input
		ActivePointerAction = "resize"
		startMouse = input.Position
		startScale = scaleObj.Scale
		startHeight = windowFrame.Size.Y.Offset
		line1.BackgroundColor3 = Theme.Accent
	end

	trackConn(grip.InputBegan:Connect(function(input)
		if isPointerInput(input) then
			beginResize("corner", input)
		end
	end))

	trackConn(bottomPullBar.InputBegan:Connect(function(input)
		if isPointerInput(input) then
			beginResize("bottom", input)
		end
	end))

	trackConn(UserInputService.InputChanged:Connect(function(input)
		if not resizing or not isMoveInput(input) then
			return
		end
		if input.UserInputType == Enum.UserInputType.Touch and activeInput and input ~= activeInput then
			return
		end
		local delta = input.Position - startMouse
		if resizeMode == "corner" then
			local scaleDelta = ((delta.X / baseWidth) + (delta.Y / baseHeight)) * 0.5
			local nextScale = clamp(startScale + scaleDelta, 0.38, 1.55)
			scaleObj.Scale = nextScale
			local rawVertical = startHeight + (delta.Y / math.max(0.4, nextScale))
			if rawVertical <= minBarHeight + 16 then
				if onBarCollapseChanged then
					onBarCollapseChanged(true)
				end
			else
				if onBarCollapseChanged then
					onBarCollapseChanged(false, clamp(rawVertical, minBarHeight + 60, baseHeight + 220))
				end
			end
		elseif resizeMode == "bottom" then
			local rawH = startHeight + (delta.Y / math.max(0.4, scaleObj.Scale))
			if rawH <= minBarHeight + 14 then
				if onBarCollapseChanged then
					onBarCollapseChanged(true)
				end
			else
				if onBarCollapseChanged then
					onBarCollapseChanged(false, clamp(rawH, minBarHeight + 55, baseHeight + 220))
				end
			end
		end
	end))

	trackConn(UserInputService.InputEnded:Connect(function(input)
		if resizing and (input == activeInput or isPointerInput(input)) then
			resizing = false
			resizeMode = nil
			activeInput = nil
			line1.BackgroundColor3 = Theme.TextMuted
			if ActivePointerAction == "resize" then
				ActivePointerAction = nil
			end
		end
	end))

	return grip
end

local SectionBuilders = {
	AimMagnet = function(c)
		buildToggleControl(
			c,
			"Ativar Aim Magnet",
			function()
				return Redx.State.Aimbot.enabled
			end,
			function(v)
				Redx.Aimbot.Toggle(v)
			end,
			"Aimbot.enabled"
		)
		buildSegmentedSelector(
			c,
			"Alvo",
			function()
				return Redx.State.Aimbot.target
			end,
			function(t)
				Redx.Aimbot.SetTarget(t)
			end,
			{
				{ tag = "torso", label = "Tronco" },
				{ tag = "head", label = "Cabeça" }
			},
			"Aimbot.target"
		)
		buildToggleControl(
			c,
			"Linha do Aim",
			function()
				return Redx.State.Aimbot.drawAimLine
			end,
			function(v)
				Redx.Aimbot.SetDrawLine(v)
			end,
			"Aimbot.drawAimLine"
		)
		buildSliderControl(
			c,
			"Smooth",
			function()
				return Redx.State.Aimbot.smooth
			end,
			function(v)
				Redx.Aimbot.SetSmooth(v)
			end,
			1,
			100,
			"%",
			"Aimbot.smooth"
		)
		buildSliderControl(
			c,
			"FOV radius",
			function()
				return Redx.State.Aimbot.radius
			end,
			function(v)
				Redx.Aimbot.SetRadius(v)
				Redx.FOV.SetRadius(v)
			end,
			40,
			450,
			" px",
			"Aimbot.radius"
		)
		buildKeybindControl(
			c,
			"Tecla Aim Magnet",
			"AimMagnet",
			function()
				return Redx.State.Aimbot.enabled
			end,
			function()
				Redx.Aimbot.Toggle(not Redx.State.Aimbot.enabled)
			end
		)
	end,

	FOVCard = function(c)
		buildToggleControl(
			c,
			"Mostrar FOV",
			function()
				return Redx.State.FOV.enabled
			end,
			function(v)
				Redx.FOV.Toggle(v)
			end,
			"FOV.enabled"
		)
		buildSliderControl(
			c,
			"Tamanho",
			function()
				return Redx.State.FOV.radius
			end,
			function(v)
				Redx.FOV.SetRadius(v)
				Redx.Aimbot.SetRadius(v)
			end,
			40,
			450,
			" px",
			"FOV.radius"
		)
		buildKeybindControl(
			c,
			"Tecla Mostrar FOV",
			"FOVToggle",
			function()
				return Redx.State.FOV.enabled
			end,
			function()
				Redx.FOV.Toggle(not Redx.State.FOV.enabled)
			end
		)
	end,

	ESPMain = function(c)
		buildToggleControl(
			c,
			"Ativar ESP",
			function()
				return Redx.State.ESP.enabled
			end,
			function(v)
				Redx.ESP.Toggle(v)
			end,
			"ESP.enabled"
		)
		buildKeybindControl(
			c,
			"Tecla ESP",
			"ESPEnable",
			function()
				return Redx.State.ESP.enabled
			end,
			function()
				Redx.ESP.Toggle(not Redx.State.ESP.enabled)
			end
		)
		buildToggleControl(
			c,
			"Caixa",
			function()
				return Redx.State.ESP.box
			end,
			function(v)
				Redx.ESP.SetElement("box", v)
			end,
			"ESP.box"
		)
		buildSegmentedSelector(
			c,
			"Estilo da caixa",
			function()
				return Redx.State.ESP.boxStyle
			end,
			function(t)
				Redx.ESP.SetBoxStyle(t)
			end,
			{
				{ tag = "normal", label = "Normal" },
				{ tag = "corner", label = "Cantos" },
				{ tag = "filled", label = "Preench." }
			},
			"ESP.boxStyle"
		)
		buildToggleControl(
			c,
			"Nome + distância",
			function()
				return Redx.State.ESP.name
			end,
			function(v)
				Redx.ESP.SetElement("name", v)
			end,
			"ESP.name"
		)
		buildToggleControl(
			c,
			"Vida (barra)",
			function()
				return Redx.State.ESP.healthBar
			end,
			function(v)
				Redx.ESP.SetElement("healthBar", v)
			end,
			"ESP.healthBar"
		)
		buildToggleControl(
			c,
			"Vida (texto)",
			function()
				return Redx.State.ESP.healthText
			end,
			function(v)
				Redx.ESP.SetElement("healthText", v)
			end,
			"ESP.healthText"
		)
		buildToggleControl(
			c,
			"Bolinha na cabeça",
			function()
				return Redx.State.ESP.headDot
			end,
			function(v)
				Redx.ESP.SetElement("headDot", v)
			end,
			"ESP.headDot"
		)
	end,

	ESPAdvanced = function(c)
		buildToggleControl(
			c,
			"Skeleton",
			function()
				return Redx.State.ESP.skeleton
			end,
			function(v)
				Redx.ESP.SetElement("skeleton", v)
			end,
			"ESP.skeleton"
		)
		buildToggleControl(
			c,
			"Tracer",
			function()
				return Redx.State.ESP.tracer
			end,
			function(v)
				Redx.ESP.SetElement("tracer", v)
			end,
			"ESP.tracer"
		)
		buildSegmentedSelector(
			c,
			"Estilo do tracer",
			function()
				return Redx.State.ESP.tracerStyle
			end,
			function(t)
				Redx.ESP.SetTracerStyle(t)
			end,
			{
				{ tag = "base", label = "Base" },
				{ tag = "top", label = "Topo" },
				{ tag = "mouse", label = "Mouse" }
			},
			"ESP.tracerStyle"
		)
		buildToggleControl(
			c,
			"Chams (glow 3D)",
			function()
				return Redx.State.ESP.chams
			end,
			function(v)
				Redx.ESP.SetElement("chams", v)
			end,
			"ESP.chams"
		)
		buildKeybindControl(
			c,
			"Tecla Chams",
			"ChamsToggle",
			function()
				return Redx.State.ESP.chams
			end,
			function()
				Redx.ESP.SetElement("chams", not Redx.State.ESP.chams)
			end
		)
		buildToggleControl(
			c,
			"Team check",
			function()
				return Redx.State.ESP.teamCheck
			end,
			function(v)
				Redx.ESP.SetTeamCheck(v)
			end,
			"ESP.teamCheck"
		)
	end,

	ESPConfig = function(c)
		buildSliderControl(
			c,
			"Distância máx",
			function()
				return Redx.State.ESP.distance
			end,
			function(v)
				Redx.ESP.SetDistance(v)
			end,
			50,
			2000,
			" m",
			"ESP.distance"
		)
		buildSliderControl(
			c,
			"Espessura",
			function()
				return Redx.State.ESP.thickness
			end,
			function(v)
				Redx.ESP.SetThickness(v)
			end,
			1,
			6,
			" px",
			"ESP.thickness"
		)
		buildSliderControl(
			c,
			"Texto ESP",
			function()
				return Redx.State.ESP.textSize
			end,
			function(v)
				Redx.ESP.SetTextSize(v)
			end,
			10,
			24,
			" px",
			"ESP.textSize"
		)
	end,

	ESPColor = function(c)
		buildPaletteControl(c, function(col)
			Redx.ESP.SetColor(col)
		end)
	end,

	Godmode = function(c)
		buildToggleControl(
			c,
			"Ativar Godmode",
			function()
				return Redx.State.Godmode.enabled
			end,
			function(v)
				Redx.Godmode.Toggle(v)
			end,
			"Godmode.enabled"
		)
		buildKeybindControl(
			c,
			"Tecla Godmode",
			"GodmodeToggle",
			function()
				return Redx.State.Godmode.enabled
			end,
			function()
				Redx.Godmode.Toggle(not Redx.State.Godmode.enabled)
			end
		)
	end,

	Performance = function(c)
		buildToggleControl(
			c,
			"FPS Boost",
			function()
				return Redx.State.Performance.fpsBoost
			end,
			function(v)
				Redx.Performance.FPSBoost(v)
			end,
			"Performance.fpsBoost"
		)
		buildKeybindControl(
			c,
			"Tecla FPS Boost",
			"FPSBoostToggle",
			function()
				return Redx.State.Performance.fpsBoost
			end,
			function()
				Redx.Performance.FPSBoost(not Redx.State.Performance.fpsBoost)
			end
		)
		buildToggleControl(
			c,
			"Remover Sombras",
			function()
				return Redx.State.Performance.removeShadows
			end,
			function(v)
				Redx.Performance.RemoveShadows(v)
			end,
			"Performance.removeShadows"
		)
		buildToggleControl(
			c,
			"Texturas Baixas",
			function()
				return Redx.State.Performance.lowTextures
			end,
			function(v)
				Redx.Performance.LowTextures(v)
			end,
			"Performance.lowTextures"
		)
	end,

	Bypass = function(c)
		create("TextLabel", {
			BackgroundTransparency = 1,
			Size = UDim2.new(1, 0, 0, 32),
			Font = Enum.Font.GothamMedium,
			Text = "Nota: nenhum bypass client-side é 100% ban-proof. Reduz vetores comuns.",
			TextColor3 = Theme.TextMuted,
			TextSize = 9.5,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextWrapped = true,
			Parent = c
		})
		buildToggleControl(
			c,
			"Modo Furtivo",
			function()
				return Redx.State.Bypass.stealth
			end,
			function(v)
				Redx.Bypass.Stealth(v)
			end,
			"Bypass.stealth"
		)
		buildToggleControl(
			c,
			"Unlock FPS",
			function()
				return Redx.State.Bypass.unlockFPS
			end,
			function(v)
				Redx.Bypass.UnlockFPS(v)
			end,
			"Bypass.unlockFPS"
		)
	end
}

local function createSectionCard(parent, titleText, iconKind, builderKey, linkedWinKey)
	local card = create("Frame", {
		BackgroundColor3 = Theme.CardBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 32),
		AutomaticSize = Enum.AutomaticSize.Y,
		ClipsDescendants = true,
		Parent = parent
	})
	addCorner(card, 6)
	addStroke(card, Theme.CardStroke, 1)

	local header = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 28),
		Parent = card
	})

	local iconBox = createIcon(header, iconKind, 13, Theme.Accent)
	iconBox.Position = UDim2.new(0, 9, 0.5, -6.5)

	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 27, 0, 0),
		Size = UDim2.new(1, -68, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = titleText,
		TextColor3 = Theme.Accent,
		TextSize = 11.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = header
	})

	if linkedWinKey then
		local pinBtn = create("TextButton", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -26, 0.5, 0),
			Size = UDim2.new(0, 18, 0, 18),
			BackgroundColor3 = UIState.Windows[linkedWinKey] and Theme.AccentDark or Theme.BadgeBg,
			BorderSizePixel = 0,
			Font = Enum.Font.GothamBold,
			Text = "⧉",
			TextColor3 = UIState.Windows[linkedWinKey] and Theme.White or Theme.TextMuted,
			TextSize = 10,
			AutoButtonColor = false,
			Parent = header
		})
		addCorner(pinBtn, 4)

		pinBtn.MouseButton1Click:Connect(function()
			setWindowActive(linkedWinKey, not UIState.Windows[linkedWinKey])
		end)

		onSync("Window." .. linkedWinKey, function(active)
			pinBtn.BackgroundColor3 = active and Theme.AccentDark or Theme.BadgeBg
			pinBtn.TextColor3 = active and Theme.White or Theme.TextMuted
		end)
	end

	local foldBtn = create("TextButton", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, -6, 0.5, 0),
		Size = UDim2.new(0, 18, 0, 18),
		BackgroundTransparency = 1,
		Font = Enum.Font.GothamBold,
		Text = "▾",
		TextColor3 = Theme.TextMuted,
		TextSize = 11,
		AutoButtonColor = false,
		Parent = header
	})

	local divider = create("Frame", {
		BackgroundColor3 = Theme.Divider,
		BorderSizePixel = 0,
		Position = UDim2.new(0, 8, 0, 27),
		Size = UDim2.new(1, -16, 0, 1),
		Parent = card
	})

	local body = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 28),
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = card
	})

	create("UIPadding", {
		PaddingTop = UDim.new(0, 6),
		PaddingBottom = UDim.new(0, 8),
		PaddingLeft = UDim.new(0, 9),
		PaddingRight = UDim.new(0, 9),
		Parent = body
	})

	create("UIListLayout", {
		SortOrder = Enum.SortOrder.LayoutOrder,
		Padding = UDim.new(0, 4),
		Parent = body
	})

	if type(builderKey) == "function" then
		builderKey(body)
	elseif SectionBuilders[builderKey] then
		SectionBuilders[builderKey](body)
	end

	local cardCollapsed = false
	local function setCardCollapsed(state)
		cardCollapsed = state
		body.Visible = not cardCollapsed
		divider.Visible = not cardCollapsed
		foldBtn.Text = cardCollapsed and "▸" or "▾"
		card.AutomaticSize = cardCollapsed and Enum.AutomaticSize.None or Enum.AutomaticSize.Y
		if cardCollapsed then
			card.Size = UDim2.new(1, 0, 0, 28)
		end
	end

	foldBtn.MouseButton1Click:Connect(function()
		setCardCollapsed(not cardCollapsed)
	end)

	return card
end

local FloatingScales = {}

local function createFloatingWindow(winKey, titleText, iconKind, builderKey, defaultPos, defaultHeight)
	local fullHeight = defaultHeight or 230
	local win = create("Frame", {
		Name = winKey,
		Position = defaultPos,
		Size = UDim2.new(0, 212, 0, fullHeight),
		BackgroundColor3 = Theme.WindowBg,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Visible = UIState.Windows[winKey],
		ZIndex = 30,
		Parent = ScreenGui
	})
	addCorner(win, 6)
	addStroke(win, Theme.FloatStroke, 1)

	local winScale = create("UIScale", {
		Scale = InitialGlobalScale,
		Parent = win
	})
	table.insert(FloatingScales, winScale)

	local titleBar = create("Frame", {
		BackgroundColor3 = Theme.FloatTitleBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 24),
		ZIndex = 32,
		Parent = win
	})
	addCorner(titleBar, 6)

	create("Frame", {
		BackgroundColor3 = Theme.FloatTitleBg,
		BorderSizePixel = 0,
		Position = UDim2.new(0, 0, 1, -6),
		Size = UDim2.new(1, 0, 0, 6),
		ZIndex = 31,
		Parent = titleBar
	})

	local collapseBtn = create("TextButton", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 2, 0, 0),
		Size = UDim2.new(0, 22, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = "▼",
		TextColor3 = Theme.White,
		TextSize = 9,
		AutoButtonColor = false,
		ZIndex = 34,
		Parent = titleBar
	})

	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 24, 0, 0),
		Size = UDim2.new(1, -54, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = titleText,
		TextColor3 = Theme.White,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		ZIndex = 33,
		Parent = titleBar
	})

	local closeBtn = create("TextButton", {
		AnchorPoint = Vector2.new(1, 0),
		Position = UDim2.new(1, -10, 0, 0),
		Size = UDim2.new(0, 20, 1, 0),
		BackgroundTransparency = 1,
		Font = Enum.Font.GothamBold,
		Text = "×",
		TextColor3 = Theme.TextSecondary,
		TextSize = 14,
		AutoButtonColor = false,
		ZIndex = 55,
		Parent = titleBar
	})

	local innerScroll = create("ScrollingFrame", {
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Position = UDim2.new(0, 6, 0, 29),
		Size = UDim2.new(1, -12, 1, -35),
		CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollBarThickness = 2,
		ScrollBarImageColor3 = Theme.Accent,
		ZIndex = 31,
		Parent = win
	})

	local innerCard = create("Frame", {
		BackgroundColor3 = Theme.CardBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = innerScroll
	})
	addCorner(innerCard, 5)
	addStroke(innerCard, Theme.CardStroke, 1)

	local subHeader = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 26),
		Parent = innerCard
	})
	local ic = createIcon(subHeader, iconKind, 13, Theme.Accent)
	ic.Position = UDim2.new(0, 8, 0.5, -6.5)

	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 26, 0, 0),
		Size = UDim2.new(1, -32, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = titleText,
		TextColor3 = Theme.Accent,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = subHeader
	})

	create("Frame", {
		BackgroundColor3 = Theme.Divider,
		BorderSizePixel = 0,
		Position = UDim2.new(0, 7, 0, 25),
		Size = UDim2.new(1, -14, 0, 1),
		Parent = innerCard
	})

	local rowsHolder = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 26),
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = innerCard
	})
	create("UIPadding", {
		PaddingTop = UDim.new(0, 5),
		PaddingBottom = UDim.new(0, 7),
		PaddingLeft = UDim.new(0, 8),
		PaddingRight = UDim.new(0, 8),
		Parent = rowsHolder
	})
	create("UIListLayout", {
		SortOrder = Enum.SortOrder.LayoutOrder,
		Padding = UDim.new(0, 4),
		Parent = rowsHolder
	})

	SectionBuilders[builderKey](rowsHolder)

	local isCollapsed = false
	local expandedHeight = fullHeight

	local function applyCollapse(c, customHeight)
		if customHeight then
			expandedHeight = customHeight
		end
		isCollapsed = c
		collapseBtn.Text = isCollapsed and "▶" or "▼"
		innerScroll.Visible = not isCollapsed
		if isCollapsed then
			tween(win, 0.14, { Size = UDim2.new(0, 212, 0, 24) })
		else
			if customHeight then
				win.Size = UDim2.new(0, 212, 0, expandedHeight)
			else
				tween(win, 0.14, { Size = UDim2.new(0, 212, 0, expandedHeight) })
			end
		end
	end

	collapseBtn.MouseButton1Click:Connect(function()
		applyCollapse(not isCollapsed)
	end)

	closeBtn.MouseButton1Click:Connect(function()
		setWindowActive(winKey, false)
	end)

	attachDraggable(titleBar, win)
	attachResizeAndPullBar(win, winScale, 212, fullHeight, 24, applyCollapse)

	registerCollapsible({
		setCollapsed = function(c)
			applyCollapse(c)
		end
	})

	onSync("Window." .. winKey, function(vis)
		win.Visible = vis
	end)

	return win
end

createFloatingWindow("AimWindow", "Aim Magnet", "aimbot", "AimMagnet", UDim2.new(0.068, 0, 0.365, 0), 252)
createFloatingWindow("ESPWindow", "ESP — Principal", "visuals", "ESPMain", UDim2.new(0.212, 0, 0.215, 0), 296)
createFloatingWindow("ESPAdvWindow", "ESP — Avançado", "visuals", "ESPAdvanced", UDim2.new(0.358, 0, 0.22, 0), 254)
createFloatingWindow("GodmodeWindow", "Godmode", "shield", "Godmode", UDim2.new(0.358, 0, 0.56, 0), 132)
createFloatingWindow("PerfWindow", "Performance", "memory", "Performance", UDim2.new(0.212, 0, 0.59, 0), 184)
createFloatingWindow("BypassWindow", "Bypass", "misc", "Bypass", UDim2.new(0.068, 0, 0.12, 0), 166)

local KeybindBox = create("Frame", {
	Name = "KeybindOverlay",
	Position = UDim2.new(0.818, 0, 0.415, 0),
	Size = UDim2.new(0, 202, 0, 158),
	BackgroundColor3 = Theme.OverlayBg,
	BackgroundTransparency = 0.04,
	BorderSizePixel = 0,
	ClipsDescendants = true,
	Visible = UIState.Windows.KeybindOverlay,
	ZIndex = 40,
	Parent = ScreenGui
})
addCorner(KeybindBox, 6)
addStroke(KeybindBox, Theme.CardStroke, 1)

local KeybindScale = create("UIScale", {
	Scale = InitialGlobalScale,
	Parent = KeybindBox
})
table.insert(FloatingScales, KeybindScale)

local KbHeader = create("Frame", {
	BackgroundTransparency = 1,
	Size = UDim2.new(1, 0, 0, 24),
	ZIndex = 41,
	Parent = KeybindBox
})

local kbDotBtn = create("TextButton", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 6, 0, 0),
	Size = UDim2.new(0, 92, 1, 0),
	Text = "",
	AutoButtonColor = false,
	ZIndex = 43,
	Parent = KbHeader
})

local kbDot = create("Frame", {
	AnchorPoint = Vector2.new(0, 0.5),
	Position = UDim2.new(0, 2, 0.5, 0),
	Size = UDim2.new(0, 6, 0, 6),
	BackgroundColor3 = Theme.Accent,
	BorderSizePixel = 0,
	ZIndex = 43,
	Parent = kbDotBtn
})
addCorner(kbDot, 99)

create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 12, 0, 0),
	Size = UDim2.new(1, -12, 1, 0),
	Font = Enum.Font.GothamBold,
	Text = PANEL_NAME,
	TextColor3 = Theme.Accent,
	TextSize = 10,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 43,
	Parent = kbDotBtn
})

create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 96, 0, 0),
	Size = UDim2.new(1, -104, 1, 0),
	Font = Enum.Font.GothamMedium,
	Text = PANEL_SUBTITLE,
	TextColor3 = Theme.TextMuted,
	TextSize = 8.5,
	TextXAlignment = Enum.TextXAlignment.Right,
	TextTruncate = Enum.TextTruncate.AtEnd,
	ZIndex = 42,
	Parent = KbHeader
})

create("Frame", {
	BackgroundColor3 = Theme.Accent,
	BorderSizePixel = 0,
	Position = UDim2.new(0, 0, 0, 24),
	Size = UDim2.new(1, 0, 0, 1),
	ZIndex = 42,
	Parent = KeybindBox
})

local KbListHolder = create("Frame", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 0, 0, 26),
	Size = UDim2.new(1, 0, 1, -28),
	ZIndex = 41,
	Parent = KeybindBox
})
create("UIPadding", {
	PaddingTop = UDim.new(0, 4),
	PaddingBottom = UDim.new(0, 6),
	PaddingLeft = UDim.new(0, 9),
	PaddingRight = UDim.new(0, 9),
	Parent = KbListHolder
})
create("UIListLayout", {
	SortOrder = Enum.SortOrder.LayoutOrder,
	Padding = UDim.new(0, 2),
	Parent = KbListHolder
})

local KbCollapsed = false
local KbExpandedHeight = 158

local function rebuildKeybindList()
	for _, child in ipairs(KbListHolder:GetChildren()) do
		if child:IsA("GuiObject") then
			child:Destroy()
		end
	end
	local visibleRows = 0
	for i = 1, #UIState.KeybindMeta do
		local meta = UIState.KeybindMeta[i]
		local kc = UIState.Keybinds[meta.Id]
		if kc then
			visibleRows = visibleRows + 1
			local isActive = meta.Get() and true or false
			local rowBtn = create("TextButton", {
				BackgroundTransparency = 1,
				Size = UDim2.new(1, 0, 0, 18),
				LayoutOrder = visibleRows,
				Text = "",
				AutoButtonColor = false,
				ZIndex = 42,
				Parent = KbListHolder
			})
			create("TextLabel", {
				BackgroundTransparency = 1,
				Size = UDim2.new(1, -34, 1, 0),
				Font = Enum.Font.GothamMedium,
				Text = meta.Label,
				TextColor3 = isActive and Theme.Accent or Theme.TextMuted,
				TextSize = 10.5,
				TextXAlignment = Enum.TextXAlignment.Left,
				ZIndex = 42,
				Parent = rowBtn
			})
			create("TextLabel", {
				BackgroundTransparency = 1,
				AnchorPoint = Vector2.new(1, 0),
				Position = UDim2.new(1, 0, 0, 0),
				Size = UDim2.new(0, 32, 1, 0),
				Font = Enum.Font.GothamMedium,
				Text = formatKeyCode(kc),
				TextColor3 = isActive and Theme.Accent or Theme.TextMuted,
				TextSize = 10.5,
				TextXAlignment = Enum.TextXAlignment.Right,
				ZIndex = 42,
				Parent = rowBtn
			})
			rowBtn.MouseButton1Click:Connect(function()
				meta.Toggle()
			end)
		end
	end
	KbExpandedHeight = math.max(54, 34 + visibleRows * 20)
	if not KbCollapsed then
		KeybindBox.Size = UDim2.new(0, 202, 0, KbExpandedHeight)
	end
end

local function setKbCollapsed(c, customH)
	if customH then
		KbExpandedHeight = customH
	end
	KbCollapsed = c
	KbListHolder.Visible = not KbCollapsed
	if KbCollapsed then
		tween(KeybindBox, 0.14, { Size = UDim2.new(0, 202, 0, 25) })
	else
		if customH then
			KeybindBox.Size = UDim2.new(0, 202, 0, KbExpandedHeight)
		else
			tween(KeybindBox, 0.14, { Size = UDim2.new(0, 202, 0, KbExpandedHeight) })
		end
	end
end

kbDotBtn.MouseButton1Click:Connect(function()
	setKbCollapsed(not KbCollapsed)
end)

attachDraggable(KbHeader, KeybindBox)
attachResizeAndPullBar(KeybindBox, KeybindScale, 202, 158, 25, setKbCollapsed)
registerCollapsible({
	setCollapsed = function(c)
		setKbCollapsed(c)
	end
})

onSync("Keybinds", rebuildKeybindList)
onSync("Window.KeybindOverlay", function(vis)
	KeybindBox.Visible = vis
end)
rebuildKeybindList()

local MainBaseWidth = 590
local MainBaseHeight = 432

local MainWindow = create("Frame", {
	Name = "MainWindow",
	Position = UDim2.new(0.606, 0, 0.062, 0),
	Size = UDim2.new(0, MainBaseWidth, 0, MainBaseHeight),
	BackgroundColor3 = Theme.WindowBg,
	BorderSizePixel = 0,
	ClipsDescendants = true,
	ZIndex = 10,
	Parent = ScreenGui
})
addCorner(MainWindow, 8)
addStroke(MainWindow, Theme.CardStroke, 1)

local MainScale = create("UIScale", {
	Scale = InitialGlobalScale,
	Parent = MainWindow
})

local MiniRestorePill = create("TextButton", {
	Name = "MiniRestorePill",
	Position = UDim2.new(0.84, 0, 0.04, 0),
	Size = UDim2.new(0, 142, 0, 32),
	BackgroundColor3 = Theme.SidebarBg,
	BorderSizePixel = 0,
	Text = "",
	AutoButtonColor = false,
	Visible = false,
	ZIndex = 60,
	Parent = ScreenGui
})
addCorner(MiniRestorePill, 999)
addStroke(MiniRestorePill, Theme.Accent, 1.2)
create("UIScale", {
	Scale = InitialGlobalScale,
	Parent = MiniRestorePill
})

local miniLogo = createBrandLogo(MiniRestorePill, 22, Theme.SidebarBg)
miniLogo.Position = UDim2.new(0, 8, 0.5, -11)

create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 35, 0, 0),
	Size = UDim2.new(1, -42, 1, 0),
	Font = Enum.Font.GothamBold,
	Text = PANEL_NAME,
	TextColor3 = Theme.TextPrimary,
	TextSize = 11.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 61,
	Parent = MiniRestorePill
})
attachDraggable(MiniRestorePill, MiniRestorePill)

local SidebarWidth = 158
local Sidebar = create("Frame", {
	Name = "Sidebar",
	BackgroundColor3 = Theme.SidebarBg,
	BorderSizePixel = 0,
	Size = UDim2.new(0, SidebarWidth, 1, 0),
	ZIndex = 11,
	Parent = MainWindow
})
addCorner(Sidebar, 8)

create("Frame", {
	BackgroundColor3 = Theme.SidebarBg,
	BorderSizePixel = 0,
	Position = UDim2.new(1, -8, 0, 0),
	Size = UDim2.new(0, 8, 1, 0),
	ZIndex = 11,
	Parent = Sidebar
})

local SidebarDivider = create("Frame", {
	BackgroundColor3 = Theme.Divider,
	BorderSizePixel = 0,
	Position = UDim2.new(0, SidebarWidth, 0, 0),
	Size = UDim2.new(0, 1, 1, 0),
	ZIndex = 12,
	Parent = MainWindow
})

local BrandHeader = create("Frame", {
	BackgroundTransparency = 1,
	Size = UDim2.new(1, 0, 0, 92),
	ZIndex = 12,
	Parent = Sidebar
})

local SidebarLogo = createBrandLogo(BrandHeader, 50, Theme.SidebarBg)
SidebarLogo.AnchorPoint = Vector2.new(0.5, 0)
SidebarLogo.Position = UDim2.new(0.5, 0, 0, 11)

create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 6, 0, 64),
	Size = UDim2.new(1, -12, 0, 20),
	Font = Enum.Font.GothamBold,
	Text = PANEL_NAME,
	TextColor3 = Theme.TextPrimary,
	TextSize = 13,
	ZIndex = 12,
	Parent = BrandHeader
})

attachDraggable(BrandHeader, MainWindow)

local NavScroll = create("ScrollingFrame", {
	BackgroundTransparency = 1,
	BorderSizePixel = 0,
	Position = UDim2.new(0, 8, 0, 94),
	Size = UDim2.new(1, -16, 1, -170),
	CanvasSize = UDim2.new(0, 0, 0, 0),
	AutomaticCanvasSize = Enum.AutomaticSize.Y,
	ScrollBarThickness = 0,
	ZIndex = 12,
	Parent = Sidebar
})

create("UIListLayout", {
	SortOrder = Enum.SortOrder.LayoutOrder,
	Padding = UDim.new(0, 6),
	Parent = NavScroll
})

local UserCard = create("Frame", {
	Name = "UserCard",
	AnchorPoint = Vector2.new(0, 1),
	Position = UDim2.new(0, 8, 1, -8),
	Size = UDim2.new(1, -16, 0, 62),
	BackgroundColor3 = Theme.CategoryBg,
	BorderSizePixel = 0,
	ZIndex = 12,
	Parent = Sidebar
})
addCorner(UserCard, 6)
addStroke(UserCard, Theme.CategoryStroke, 1)

local AvatarWrap = create("Frame", {
	Position = UDim2.new(0, 5, 0, 5),
	Size = UDim2.new(0, 26, 0, 26),
	BackgroundColor3 = Color3.fromRGB(12, 18, 34),
	BorderSizePixel = 0,
	ClipsDescendants = true,
	ZIndex = 13,
	Parent = UserCard
})
addCorner(AvatarWrap, 5)

local fallbackAvatarLogo = createBrandLogo(AvatarWrap, 20, Color3.fromRGB(12, 18, 34))
fallbackAvatarLogo.AnchorPoint = Vector2.new(0.5, 0.5)
fallbackAvatarLogo.Position = UDim2.fromScale(0.5, 0.5)

local AvatarImg = create("ImageLabel", {
	BackgroundTransparency = 1,
	Size = UDim2.fromScale(1, 1),
	Image = "",
	ZIndex = 14,
	Parent = AvatarWrap
})
addCorner(AvatarImg, 5)

if LocalPlayer then
	spawnTask(function()
		local ok, content = pcall(function()
			return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
		end)
		if ok and type(content) == "string" and content ~= "" then
			AvatarImg.Image = content
			fallbackAvatarLogo.Visible = false
		end
	end)
end

local displayUsername = (LocalPlayer and LocalPlayer.Name) or "EXTERNAL-USER"
create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 36, 0, 4),
	Size = UDim2.new(1, -40, 0, 14),
	Font = Enum.Font.GothamBold,
	Text = "Username: " .. displayUsername,
	TextColor3 = Theme.TextPrimary,
	TextSize = 9.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextTruncate = Enum.TextTruncate.AtEnd,
	ZIndex = 13,
	Parent = UserCard
})

local roleLine = create("Frame", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 36, 0, 18),
	Size = UDim2.new(1, -40, 0, 14),
	ZIndex = 13,
	Parent = UserCard
})
create("TextLabel", {
	BackgroundTransparency = 1,
	Size = UDim2.new(0, 26, 1, 0),
	Font = Enum.Font.GothamMedium,
	Text = "Role:",
	TextColor3 = Theme.TextMuted,
	TextSize = 9.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 13,
	Parent = roleLine
})
create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 27, 0, 0),
	Size = UDim2.new(1, -27, 1, 0),
	Font = Enum.Font.GothamBold,
	Text = PANEL_NAME,
	TextColor3 = Theme.Accent,
	TextSize = 9.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextTruncate = Enum.TextTruncate.AtEnd,
	ZIndex = 13,
	Parent = roleLine
})

do
	local label = create("TextLabel", {
		Name = "KeyDurationLabel",
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 5, 0, 37),
		Size = UDim2.new(1, -10, 0, 18),
		Font = Enum.Font.GothamMedium,
		Text = "Duração da Key: 00:00",
		TextColor3 = Theme.Accent,
		TextSize = 9,
		TextScaled = true,
		TextXAlignment = Enum.TextXAlignment.Left,
		ZIndex = 13,
		Parent = UserCard
	})
	create("UITextSizeConstraint", { MinTextSize = 7, MaxTextSize = 9, Parent = label })
	local remaining = KeySession and tonumber(KeySession.secondsLeft)
	local receivedAt = KeySession and KeySession.receivedAt or os.clock()
	if not remaining or remaining ~= remaining or math.abs(remaining) == math.huge then
		remaining = nil
		if KeySession and type(KeySession.expiresAt) == "string" then
			pcall(function()
				local expiry = DateTime.fromIsoDate(KeySession.expiresAt)
				if expiry then
					remaining = expiry.UnixTimestampMillis / 1000 - DateTime.now().UnixTimestampMillis / 1000
					receivedAt = os.clock()
				end
			end)
		end
	end
	local lastText
	local function refreshKeyDuration()
		local text = "Duração da Key: indisponível"
		if remaining then
			local total = math.max(0, math.ceil(remaining - math.max(0, os.clock() - receivedAt)))
			local days = math.floor(total / 86400)
			local hours = math.floor(total / 3600) % 24
			local minutes = math.floor(total / 60) % 60
			local seconds = total % 60
			local duration
			if days > 0 then
				duration = string.format("%dd %02d:%02d:%02d", days, hours, minutes, seconds)
			elseif total >= 3600 then
				duration = string.format("%02d:%02d:%02d", hours, minutes, seconds)
			else
				duration = string.format("%02d:%02d", minutes, seconds)
			end
			text = "Duração da Key: " .. duration
		end
		if text ~= lastText then
			lastText = text
			label.Text = text
		end
	end
	refreshKeyDuration()
	trackConn(RunService.Heartbeat:Connect(refreshKeyDuration))
end

local TopHeader = create("Frame", {
	Name = "TopHeader",
	BackgroundColor3 = Theme.HeaderBg,
	BorderSizePixel = 0,
	Position = UDim2.new(0, SidebarWidth + 1, 0, 0),
	Size = UDim2.new(1, -(SidebarWidth + 1), 0, 38),
	ZIndex = 14,
	Parent = MainWindow
})

local HeaderBottomDivider = create("Frame", {
	BackgroundColor3 = Theme.Divider,
	BorderSizePixel = 0,
	Position = UDim2.new(0, 0, 1, -1),
	Size = UDim2.new(1, 0, 0, 1),
	ZIndex = 15,
	Parent = TopHeader
})

local CollapsedBrandSlot = create("Frame", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 28, 0, 0),
	Size = UDim2.new(0, 102, 1, 0),
	Visible = false,
	ZIndex = 16,
	Parent = TopHeader
})
local collapsedBarLogo = createBrandLogo(CollapsedBrandSlot, 20, Theme.HeaderBg)
collapsedBarLogo.Position = UDim2.new(0, 0, 0.5, -10)
create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 24, 0, 0),
	Size = UDim2.new(1, -24, 1, 0),
	Font = Enum.Font.GothamBold,
	Text = PANEL_NAME,
	TextColor3 = Theme.Accent,
	TextSize = 11.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 16,
	Parent = CollapsedBrandSlot
})

local MainCollapseBtn = create("TextButton", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 6, 0.5, -10),
	Size = UDim2.new(0, 20, 0, 20),
	Font = Enum.Font.GothamBold,
	Text = "▼",
	TextColor3 = Theme.TextSecondary,
	TextSize = 10,
	AutoButtonColor = false,
	ZIndex = 16,
	Parent = TopHeader
})

local SubtitleHolder = create("Frame", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 28, 0, 0),
	Size = UDim2.new(1, -148, 1, 0),
	ZIndex = 15,
	Parent = TopHeader
})
create("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	VerticalAlignment = Enum.VerticalAlignment.Center,
	SortOrder = Enum.SortOrder.LayoutOrder,
	Padding = UDim.new(0, 8),
	Parent = SubtitleHolder
})

create("TextLabel", {
	BackgroundTransparency = 1,
	Size = UDim2.new(0, 0, 0, 20),
	AutomaticSize = Enum.AutomaticSize.X,
	LayoutOrder = 1,
	Font = Enum.Font.GothamBold,
	Text = PANEL_SUBTITLE,
	TextColor3 = Theme.TextPrimary,
	TextSize = 12,
	ZIndex = 15,
	Parent = SubtitleHolder
})

local ConnectedBadge = create("Frame", {
	BackgroundColor3 = Theme.ConnectedBg,
	BorderSizePixel = 0,
	Size = UDim2.new(0, 66, 0, 19),
	LayoutOrder = 2,
	ZIndex = 15,
	Parent = SubtitleHolder
})
addCorner(ConnectedBadge, 999)
addStroke(ConnectedBadge, Theme.ConnectedStroke, 1)

create("TextLabel", {
	BackgroundTransparency = 1,
	Size = UDim2.fromScale(1, 1),
	Font = Enum.Font.GothamMedium,
	Text = "Connected",
	TextColor3 = Theme.ConnectedText,
	TextSize = 10.5,
	ZIndex = 16,
	Parent = ConnectedBadge
})

local TabsQuickBtn = create("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -42, 0.5, 0),
	Size = UDim2.new(0, 56, 0, 21),
	BackgroundColor3 = Theme.SubTabActiveBg,
	BorderSizePixel = 0,
	Font = Enum.Font.GothamBold,
	Text = "Abas ▾",
	TextColor3 = Theme.Accent,
	TextSize = 10.5,
	AutoButtonColor = false,
	ZIndex = 16,
	Parent = TopHeader
})
addCorner(TabsQuickBtn, 5)
addStroke(TabsQuickBtn, Theme.FloatStroke, 1)

local MinimizePillBtn = create("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -16, 0.5, 0),
	Size = UDim2.new(0, 21, 0, 21),
	BackgroundColor3 = Theme.CategoryBg,
	BorderSizePixel = 0,
	Font = Enum.Font.GothamBold,
	Text = "—",
	TextColor3 = Theme.TextSecondary,
	TextSize = 11,
	AutoButtonColor = false,
	ZIndex = 55,
	Parent = TopHeader
})
addCorner(MinimizePillBtn, 5)
addStroke(MinimizePillBtn, Theme.CategoryStroke, 1)

attachDraggable(TopHeader, MainWindow)

local TabsDrawer = create("Frame", {
	Name = "TabsSelectorPopover",
	AnchorPoint = Vector2.new(1, 0),
	Position = UDim2.new(1, -8, 0, 42),
	Size = UDim2.new(0, 224, 0, 294),
	BackgroundColor3 = Theme.OverlayBg,
	BorderSizePixel = 0,
	Visible = false,
	ZIndex = 45,
	Parent = MainWindow
})
addCorner(TabsDrawer, 6)
addStroke(TabsDrawer, Theme.FloatStroke, 1)

local DrawerHeader = create("Frame", {
	BackgroundColor3 = Theme.FloatTitleBg,
	BorderSizePixel = 0,
	Size = UDim2.new(1, 0, 0, 26),
	ZIndex = 46,
	Parent = TabsDrawer
})
addCorner(DrawerHeader, 6)

create("TextLabel", {
	BackgroundTransparency = 1,
	Position = UDim2.new(0, 10, 0, 0),
	Size = UDim2.new(1, -68, 1, 0),
	Font = Enum.Font.GothamBold,
	Text = "Ativar / Guardar Abas",
	TextColor3 = Theme.White,
	TextSize = 10.5,
	TextXAlignment = Enum.TextXAlignment.Left,
	ZIndex = 47,
	Parent = DrawerHeader
})

local CollapseAllQuickBtn = create("TextButton", {
	AnchorPoint = Vector2.new(1, 0.5),
	Position = UDim2.new(1, -6, 0.5, 0),
	Size = UDim2.new(0, 58, 0, 18),
	BackgroundColor3 = Theme.BadgeBg,
	BorderSizePixel = 0,
	Font = Enum.Font.GothamBold,
	Text = "Guardar",
	TextColor3 = Theme.Accent,
	TextSize = 9.5,
	AutoButtonColor = false,
	ZIndex = 47,
	Parent = DrawerHeader
})
addCorner(CollapseAllQuickBtn, 4)

local DrawerScroll = create("ScrollingFrame", {
	BackgroundTransparency = 1,
	BorderSizePixel = 0,
	Position = UDim2.new(0, 8, 0, 30),
	Size = UDim2.new(1, -16, 1, -36),
	CanvasSize = UDim2.new(0, 0, 0, 0),
	AutomaticCanvasSize = Enum.AutomaticSize.Y,
	ScrollBarThickness = 2,
	ScrollBarImageColor3 = Theme.Accent,
	ZIndex = 46,
	Parent = TabsDrawer
})
create("UIListLayout", {
	SortOrder = Enum.SortOrder.LayoutOrder,
	Padding = UDim.new(0, 3),
	Parent = DrawerScroll
})

buildWindowToggleRow(DrawerScroll, "Aba Aim Magnet", "AimWindow")
buildWindowToggleRow(DrawerScroll, "Aba ESP Principal", "ESPWindow")
buildWindowToggleRow(DrawerScroll, "Aba ESP Avançado", "ESPAdvWindow")
buildWindowToggleRow(DrawerScroll, "Aba Godmode", "GodmodeWindow")
buildWindowToggleRow(DrawerScroll, "Aba Performance", "PerfWindow")
buildWindowToggleRow(DrawerScroll, "Aba Bypass", "BypassWindow")
buildWindowToggleRow(DrawerScroll, "Keybind List", "KeybindOverlay")
buildWindowToggleRow(DrawerScroll, "Barra Timer (00:00)", "TimerPill")
buildWindowToggleRow(DrawerScroll, "Barra Players (0)", "PlayerCountPill")
buildToggleControl(
	DrawerScroll,
	"Mostrar FOV",
	function()
		return Redx.State.FOV.enabled
	end,
	function(v)
		Redx.FOV.Toggle(v)
	end,
	"FOV.enabled"
)

local AllBarsFolded = false
CollapseAllQuickBtn.MouseButton1Click:Connect(function()
	AllBarsFolded = not AllBarsFolded
	CollapseAllQuickBtn.Text = AllBarsFolded and "Expandir" or "Guardar"
	for i = 1, #AllCollapsibleEntries do
		pcall(AllCollapsibleEntries[i].setCollapsed, AllBarsFolded)
	end
end)

TabsQuickBtn.MouseButton1Click:Connect(function()
	TabsDrawer.Visible = not TabsDrawer.Visible
end)

local ContentBody = create("Frame", {
	Name = "ContentBody",
	BackgroundTransparency = 1,
	Position = UDim2.new(0, SidebarWidth + 1, 0, 38),
	Size = UDim2.new(1, -(SidebarWidth + 1), 1, -38),
	ZIndex = 12,
	Parent = MainWindow
})

local Pages = {}
local SubTabButtons = {}
local CurrentTabId = nil

local function createTabPage(tabId)
	local scroll = create("ScrollingFrame", {
		Name = tabId .. "_Page",
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Size = UDim2.fromScale(1, 1),
		CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollBarThickness = 2,
		ScrollBarImageColor3 = Theme.Accent,
		Visible = false,
		ZIndex = 13,
		Parent = ContentBody
	})
	create("UIPadding", {
		PaddingTop = UDim.new(0, 10),
		PaddingBottom = UDim.new(0, 12),
		PaddingLeft = UDim.new(0, 10),
		PaddingRight = UDim.new(0, 10),
		Parent = scroll
	})

	local cols = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = scroll
	})

	local leftCol = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(0.5, -5, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = cols
	})
	create("UIListLayout", {
		SortOrder = Enum.SortOrder.LayoutOrder,
		Padding = UDim.new(0, 8),
		Parent = leftCol
	})

	local rightCol = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0.5, 5, 0, 0),
		Size = UDim2.new(0.5, -5, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		Parent = cols
	})
	create("UIListLayout", {
		SortOrder = Enum.SortOrder.LayoutOrder,
		Padding = UDim.new(0, 8),
		Parent = rightCol
	})

	Pages[tabId] = {
		Scroll = scroll,
		Left = leftCol,
		Right = rightCol
	}
	return Pages[tabId]
end

local function selectSubTab(tabId)
	if not Pages[tabId] then
		return
	end
	CurrentTabId = tabId
	for id, page in pairs(Pages) do
		page.Scroll.Visible = (id == tabId)
	end
	for id, ui in pairs(SubTabButtons) do
		local active = (id == tabId)
		ui.Button.BackgroundColor3 = active and Theme.SubTabActiveBg or Theme.CategoryBg
		ui.Button.BackgroundTransparency = active and 0 or 1
		ui.Label.TextColor3 = active and Theme.Accent or Theme.TextMuted
		ui.Label.Font = active and Enum.Font.GothamBold or Enum.Font.GothamMedium
		ui.Indicator.Visible = active
	end
end

local function createSidebarCategory(orderIdx, titleText, iconKind, subTabs)
	local catCard = create("Frame", {
		BackgroundColor3 = Theme.CategoryBg,
		BorderSizePixel = 0,
		LayoutOrder = orderIdx,
		Size = UDim2.new(1, 0, 0, 28),
		AutomaticSize = Enum.AutomaticSize.Y,
		ClipsDescendants = true,
		ZIndex = 13,
		Parent = NavScroll
	})
	addCorner(catCard, 6)
	addStroke(catCard, Theme.CategoryStroke, 1)

	local headerBtn = create("TextButton", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 28),
		Text = "",
		AutoButtonColor = false,
		ZIndex = 14,
		Parent = catCard
	})

	local ic = createIcon(headerBtn, iconKind, 13, Theme.Accent)
	ic.Position = UDim2.new(0, 8, 0.5, -6.5)

	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 26, 0, 0),
		Size = UDim2.new(1, -46, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = titleText,
		TextColor3 = Theme.TextPrimary,
		TextSize = 11.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		ZIndex = 14,
		Parent = headerBtn
	})

	local chevron = create("TextLabel", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, -8, 0.5, 0),
		Size = UDim2.new(0, 12, 0, 12),
		BackgroundTransparency = 1,
		Font = Enum.Font.GothamBold,
		Text = "⌃",
		TextColor3 = Theme.Accent,
		TextSize = 12,
		ZIndex = 14,
		Parent = headerBtn
	})

	local subList = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 28),
		Size = UDim2.new(1, 0, 0, 0),
		AutomaticSize = Enum.AutomaticSize.Y,
		ZIndex = 13,
		Parent = catCard
	})
	create("UIPadding", {
		PaddingBottom = UDim.new(0, 6),
		PaddingLeft = UDim.new(0, 6),
		PaddingRight = UDim.new(0, 6),
		Parent = subList
	})
	create("UIListLayout", {
		SortOrder = Enum.SortOrder.LayoutOrder,
		Padding = UDim.new(0, 2),
		Parent = subList
	})

	for idx = 1, #subTabs do
		local st = subTabs[idx]
		local btn = create("TextButton", {
			BackgroundColor3 = Theme.SubTabActiveBg,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			LayoutOrder = idx,
			Size = UDim2.new(1, 0, 0, 23),
			Text = "",
			AutoButtonColor = false,
			Visible = UIState.SidebarTabsEnabled[st.Id] ~= false,
			ZIndex = 14,
			Parent = subList
		})
		addCorner(btn, 4)

		local bar = create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Position = UDim2.new(0, 2, 0.5, 0),
			Size = UDim2.new(0, 2, 0, 13),
			BackgroundColor3 = Theme.Accent,
			BorderSizePixel = 0,
			Visible = false,
			ZIndex = 15,
			Parent = btn
		})
		addCorner(bar, 2)

		local lbl = create("TextLabel", {
			BackgroundTransparency = 1,
			Position = UDim2.new(0, 10, 0, 0),
			Size = UDim2.new(1, -14, 1, 0),
			Font = Enum.Font.GothamMedium,
			Text = st.Label,
			TextColor3 = Theme.TextMuted,
			TextSize = 11,
			TextXAlignment = Enum.TextXAlignment.Left,
			ZIndex = 15,
			Parent = btn
		})

		SubTabButtons[st.Id] = {
			Button = btn,
			Indicator = bar,
			Label = lbl
		}

		btn.MouseButton1Click:Connect(function()
			selectSubTab(st.Id)
		end)

		onSync("SidebarTab." .. st.Id, function(enabled)
			btn.Visible = enabled
			if not enabled and CurrentTabId == st.Id then
				for fallbackId, isOn in pairs(UIState.SidebarTabsEnabled) do
					if isOn and Pages[fallbackId] then
						selectSubTab(fallbackId)
						break
					end
				end
			end
		end)
	end

	local expanded = true
	local function setExpanded(state)
		expanded = state
		subList.Visible = expanded
		chevron.Text = expanded and "⌃" or "⌄"
		catCard.AutomaticSize = expanded and Enum.AutomaticSize.Y or Enum.AutomaticSize.None
		if not expanded then
			catCard.Size = UDim2.new(1, 0, 0, 28)
		end
	end

	headerBtn.MouseButton1Click:Connect(function()
		setExpanded(not expanded)
	end)

	return catCard
end

createSidebarCategory(1, "Aimbot", "aimbot", {
	{ Id = "Aimbot", Label = "Aimbot" }
})
createSidebarCategory(2, "Visuals", "visuals", {
	{ Id = "Visuals", Label = "Visuals" },
	{ Id = "ESPConfig", Label = "ESP Config" }
})
createSidebarCategory(3, "Memory", "memory", {
	{ Id = "Player", Label = "Player" },
	{ Id = "Performance", Label = "Performance" }
})
createSidebarCategory(4, "Miscellaneous", "misc", {
	{ Id = "Personalizacao", Label = "Personalização" },
	{ Id = "Config", Label = "Configurações" }
})

local PageAimbot = createTabPage("Aimbot")
createSectionCard(PageAimbot.Left, "Aim Magnet", "aimbot", "AimMagnet", "AimWindow")
createSectionCard(PageAimbot.Right, "FOV Circle", "aimbot", "FOVCard", nil)

local PageVisuals = createTabPage("Visuals")
createSectionCard(PageVisuals.Left, "ESP — Principal", "visuals", "ESPMain", "ESPWindow")
createSectionCard(PageVisuals.Right, "ESP — Avançado", "visuals", "ESPAdvanced", "ESPAdvWindow")

local PageESPConfig = createTabPage("ESPConfig")
createSectionCard(PageESPConfig.Left, "ESP — Config", "visuals", "ESPConfig", nil)
createSectionCard(PageESPConfig.Right, "Cor do ESP", "visuals", "ESPColor", nil)

local PagePlayer = createTabPage("Player")
createSectionCard(PagePlayer.Left, "Godmode", "shield", "Godmode", "GodmodeWindow")
createSectionCard(PagePlayer.Right, "Bypass", "misc", "Bypass", "BypassWindow")

local PagePerf = createTabPage("Performance")
createSectionCard(PagePerf.Left, "Performance", "memory", "Performance", "PerfWindow")
createSectionCard(PagePerf.Right, "Bypass", "misc", "Bypass", "BypassWindow")

local PagePers = createTabPage("Personalizacao")
createSectionCard(PagePers.Left, "Abas Flutuantes", "misc", function(body)
	buildWindowToggleRow(body, "Aim Magnet", "AimWindow")
	buildWindowToggleRow(body, "ESP — Principal", "ESPWindow")
	buildWindowToggleRow(body, "ESP — Avançado", "ESPAdvWindow")
	buildWindowToggleRow(body, "Godmode", "GodmodeWindow")
	buildWindowToggleRow(body, "Performance", "PerfWindow")
	buildWindowToggleRow(body, "Bypass", "BypassWindow")
	buildWindowToggleRow(body, "Keybind List", "KeybindOverlay")
	buildWindowToggleRow(body, "Barra Timer", "TimerPill")
	buildWindowToggleRow(body, "Barra Players", "PlayerCountPill")
end, nil)

createSectionCard(PagePers.Right, "Escala & Abas Menu", "visuals", function(body)
	buildSliderControl(
		body,
		"Escala",
		function()
			return UIState.MainScalePct
		end,
		function(v)
			UIState.MainScalePct = v
			MainScale.Scale = clamp((v / 100) * InitialGlobalScale, 0.38, 1.55)
			emitSync("UI.MainScale", v)
		end,
		45,
		145,
		"%",
		"UI.MainScale"
	)
	buildSliderControl(
		body,
		"Flutuante",
		function()
			return UIState.FloatScalePct
		end,
		function(v)
			UIState.FloatScalePct = v
			local sc = clamp((v / 100) * InitialGlobalScale, 0.38, 1.55)
			for i = 1, #FloatingScales do
				FloatingScales[i].Scale = sc
			end
			emitSync("UI.FloatScale", v)
		end,
		45,
		145,
		"%",
		"UI.FloatScale"
	)

	local sidebarItems = {
		{ Id = "Aimbot", Label = "Aba Aimbot" },
		{ Id = "Visuals", Label = "Aba Visuals" },
		{ Id = "ESPConfig", Label = "Aba ESP Config" },
		{ Id = "Player", Label = "Aba Player" },
		{ Id = "Performance", Label = "Aba Performance" },
		{ Id = "Config", Label = "Aba Configurações" }
	}
	for i = 1, #sidebarItems do
		local item = sidebarItems[i]
		buildToggleControl(
			body,
			item.Label,
			function()
				return UIState.SidebarTabsEnabled[item.Id]
			end,
			function(v)
				setSidebarTabEnabled(item.Id, v)
			end,
			"SidebarTab." .. item.Id
		)
	end
end, nil)

local MainBarCollapsed = false
local MainExpandedHeight = MainBaseHeight

local function setMainCollapsedToBar(collapsed, customHeight)
	if customHeight then
		MainExpandedHeight = customHeight
	end
	MainBarCollapsed = collapsed
	Sidebar.Visible = not MainBarCollapsed
	SidebarDivider.Visible = not MainBarCollapsed
	ContentBody.Visible = not MainBarCollapsed
	HeaderBottomDivider.Visible = not MainBarCollapsed
	CollapsedBrandSlot.Visible = MainBarCollapsed
	MainCollapseBtn.Text = MainBarCollapsed and "▶" or "▼"

	if MainBarCollapsed then
		TabsDrawer.Visible = false
		TopHeader.Position = UDim2.new(0, 0, 0, 0)
		TopHeader.Size = UDim2.new(1, 0, 0, 38)
		SubtitleHolder.Position = UDim2.new(0, 136, 0, 0)
		SubtitleHolder.Size = UDim2.new(1, -246, 1, 0)
		tween(MainWindow, 0.15, { Size = UDim2.new(0, MainBaseWidth, 0, 38) })
	else
		TopHeader.Position = UDim2.new(0, SidebarWidth + 1, 0, 0)
		TopHeader.Size = UDim2.new(1, -(SidebarWidth + 1), 0, 38)
		SubtitleHolder.Position = UDim2.new(0, 28, 0, 0)
		SubtitleHolder.Size = UDim2.new(1, -148, 1, 0)
		if customHeight then
			MainWindow.Size = UDim2.new(0, MainBaseWidth, 0, MainExpandedHeight)
		else
			tween(MainWindow, 0.15, { Size = UDim2.new(0, MainBaseWidth, 0, MainExpandedHeight) })
		end
	end
end

MainCollapseBtn.MouseButton1Click:Connect(function()
	setMainCollapsedToBar(not MainBarCollapsed)
end)

MinimizePillBtn.MouseButton1Click:Connect(function()
	MainWindow.Visible = false
	MiniRestorePill.Visible = true
end)

MiniRestorePill.MouseButton1Click:Connect(function()
	MiniRestorePill.Visible = false
	MainWindow.Visible = true
end)

attachResizeAndPullBar(MainWindow, MainScale, MainBaseWidth, MainBaseHeight, 38, setMainCollapsedToBar)

local PageConfig = createTabPage("Config")
createSectionCard(PageConfig.Left, "Painel", "misc", function(body)
	buildSliderControl(
		body,
		"Escala",
		function()
			return UIState.MainScalePct
		end,
		function(v)
			UIState.MainScalePct = v
			MainScale.Scale = clamp((v / 100) * InitialGlobalScale, 0.38, 1.55)
			emitSync("UI.MainScale", v)
		end,
		45,
		145,
		"%",
		"UI.MainScale"
	)

	buildKeybindControl(body, "Tecla do Painel", "MenuToggle", nil, nil)

	buildActionButton(body, "Guardar / Expandir Todas Barras", function()
		AllBarsFolded = not AllBarsFolded
		for i = 1, #AllCollapsibleEntries do
			pcall(AllCollapsibleEntries[i].setCollapsed, AllBarsFolded)
		end
		setMainCollapsedToBar(AllBarsFolded)
	end, false)

	buildActionButton(body, "Resetar posição & escala", function()
		UIState.MainScalePct = 100
		UIState.FloatScalePct = 100
		MainScale.Scale = InitialGlobalScale
		for i = 1, #FloatingScales do
			FloatingScales[i].Scale = InitialGlobalScale
		end
		emitSync("UI.MainScale", 100)
		emitSync("UI.FloatScale", 100)
		setMainCollapsedToBar(false, MainBaseHeight)
		MainWindow.Position = UDim2.new(0.606, 0, 0.062, 0)
	end, false)
end, nil)

createSectionCard(PageConfig.Left, "Bypass", "misc", "Bypass", "BypassWindow")

createSectionCard(PageConfig.Right, "Sobre", "player", function(body)
	create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 18),
		Font = Enum.Font.GothamBold,
		Text = PANEL_NAME,
		TextColor3 = Theme.Accent,
		TextSize = 11.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = body
	})
	create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 16),
		Font = Enum.Font.GothamMedium,
		Text = PANEL_SUBTITLE,
		TextColor3 = Theme.TextSecondary,
		TextSize = 10,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = body
	})
	create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 16),
		Font = Enum.Font.GothamMedium,
		Text = HAS_DRAWING and "Drawing: disponível" or "Drawing: ausente (ESP limitado)",
		TextColor3 = HAS_DRAWING and Theme.ConnectedText or Theme.Accent,
		TextSize = 10,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = body
	})
	buildActionButton(body, "Fechar Painel", function()
		Redx.UI.Destroy()
	end, true)
end, nil)

selectSubTab("Aimbot")
fovRefresh()

trackConn(UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if input.UserInputType ~= Enum.UserInputType.Keyboard then
		return
	end
	local kc = input.KeyCode
	if not kc or kc == Enum.KeyCode.Unknown then
		return
	end

	if BindingTargetId then
		local target = BindingTargetId
		BindingTargetId = nil
		if kc == Enum.KeyCode.Escape or kc == Enum.KeyCode.Backspace then
			UIState.Keybinds[target] = nil
		else
			UIState.Keybinds[target] = kc
		end
		emitSync("Keybinds")
		return
	end

	if gameProcessed then
		return
	end

	if UIState.Keybinds.MenuToggle and kc == UIState.Keybinds.MenuToggle then
		if MiniRestorePill.Visible then
			MiniRestorePill.Visible = false
			MainWindow.Visible = true
		else
			MainWindow.Visible = not MainWindow.Visible
		end
		return
	end

	for i = 1, #UIState.KeybindMeta do
		local meta = UIState.KeybindMeta[i]
		local boundKc = UIState.Keybinds[meta.Id]
		if boundKc and boundKc == kc then
			meta.Toggle()
		end
	end
end))

Redx.Theme = Theme
Redx.Connections = Connections
Redx.track = trackConn
Redx.create = create
Redx.corner = addCorner
Redx.stroke = addStroke
Redx.tween = tween
Redx.createBrandLogo = createBrandLogo
Redx.createIcon = createIcon
Redx.attachDraggable = attachDraggable
Redx.buildToggle = buildToggleControl
Redx.buildSlider = buildSliderControl
Redx.buildButton = buildActionButton
Redx.buildStyleSelector = buildSegmentedSelector
Redx.buildPalette = buildPaletteControl
Redx.buildPartSelector = function(parent, getter, onPick)
	return buildSegmentedSelector(parent, "Alvo", getter, onPick, {
		{ tag = "torso", label = "Tronco" },
		{ tag = "head", label = "Cabeça" }
	})
end

Redx.UI = {
	ScreenGui = ScreenGui,
	MainWindow = MainWindow,
	MainScale = MainScale,
	MinBtn = MinimizePillBtn,
	FOVFrame = FOVFrame
}

Redx.UI.Show = function()
	if ScreenGui then
		ScreenGui.Enabled = true
	end
	MiniRestorePill.Visible = false
	MainWindow.Visible = true
end

Redx.UI.Hide = function()
	if ScreenGui then
		ScreenGui.Enabled = false
	end
end

Redx.UI.Destroy = function()
	if PanelDestroyed then
		return
	end
	PanelDestroyed = true
	if Redx.State.Godmode.enabled then
		Redx.Godmode.Toggle(false)
	end
	Redx.State.Performance.fpsBoost = false
	Redx.State.Performance.removeShadows = false
	Redx.State.Performance.lowTextures = false
	applyPerformance()
	if Redx.State.Bypass.unlockFPS then
		Redx.Bypass.UnlockFPS(false)
	end
	for i = 1, #Connections do
		pcall(function()
			Connections[i]:Disconnect()
		end)
	end
	Connections = {}
	Redx.Connections = Connections
	for plr in pairs(ESPData) do
		espDestroy(plr)
	end
	chamsClear()
	if godLoop then
		pcall(function()
			godLoop:Disconnect()
		end)
		godLoop = nil
	end
	if godRespawn then
		pcall(function()
			godRespawn:Disconnect()
		end)
		godRespawn = nil
	end
	if aimLineObj then
		dRemove(aimLineObj)
		aimLineObj = nil
	end
	pcall(function()
		if ScreenGui then
			ScreenGui:Destroy()
		end
	end)
	if env then
		env.Redx = nil
		env.RedxPainelUnload = nil
		env.RedxPainelSetLogo = nil
		if env.RedxGui == ScreenGui then
			env.RedxGui = nil
		end
	end
	Redx.FOV._listeners = {}
	SyncListeners = {}
	LogoListeners = {}
end

if env then
	env.RedxGui = ScreenGui
	env.RedxPainelUnload = Redx.UI.Destroy
	env.RedxPainelSetLogo = resolveLogoUrl
end

resolveLogoUrl(LOGO_LINK)
