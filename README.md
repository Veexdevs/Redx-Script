local LOGO_LINK = ""
local PANEL_NAME = "Redx Painel"
local PANEL_SUBTITLE = "By Denolk and Ghost V1.0"

local env = (type(getgenv) == "function" and getgenv()) or _G
if env and type(env.RedxPainelUnload) == "function" then
	pcall(env.RedxPainelUnload)
end

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local spawnTask = (task and task.spawn) or function(fn, ...)
	return coroutine.wrap(fn)(...)
end
local waitTask = (task and task.wait) or wait
local clampNum = math.clamp or function(v, lo, hi)
	if v < lo then
		return lo
	elseif v > hi then
		return hi
	end
	return v
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
	local inst = Instance.new(className)
	if props then
		for k, v in pairs(props) do
			if k ~= "Parent" then
				inst[k] = v
			end
		end
		if props.Parent then
			inst.Parent = props.Parent
		end
	end
	if children then
		for i = 1, #children do
			children[i].Parent = inst
		end
	end
	return inst
end

local function addCorner(parent, radius)
	return create("UICorner", {
		CornerRadius = UDim.new(0, radius or 6),
		Parent = parent
	})
end

local function addStroke(parent, color, thickness, transparency)
	return create("UIStroke", {
		Color = color or Theme.CardStroke,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		Parent = parent
	})
end

local function tween(obj, duration, goal)
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
	elseif kind == "weapon" then
		local barrel = create("Frame", {
			Position = UDim2.fromScale(0.10, 0.24),
			Size = UDim2.fromScale(0.80, 0.26),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(barrel, 2)
		local grip = create("Frame", {
			Position = UDim2.fromScale(0.16, 0.44),
			Size = UDim2.fromScale(0.26, 0.42),
			Rotation = 14,
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(grip, 2)
		create("Frame", {
			Position = UDim2.fromScale(0.40, 0.48),
			Size = UDim2.fromScale(0.18, 0.14),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
	elseif kind == "world" then
		local globe = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.84, 0.84),
			BackgroundTransparency = 1,
			Parent = box
		})
		addCorner(globe, 99)
		addStroke(globe, col, 1.3)
		local meridian = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.42, 0.84),
			BackgroundTransparency = 1,
			Parent = box
		})
		addCorner(meridian, 99)
		addStroke(meridian, col, 1.1)
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.82, 0.09),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
	elseif kind == "jetpack" then
		local body = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.5, 0.5),
			Size = UDim2.fromScale(0.88, 0.22),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(body, 3)
		local wing = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.52, 0.5),
			Size = UDim2.fromScale(0.24, 0.88),
			Rotation = 16,
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(wing, 2)
		local tail = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.16, 0.5),
			Size = UDim2.fromScale(0.16, 0.52),
			Rotation = 14,
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(tail, 2)
	elseif kind == "panic" then
		local canopy = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0),
			Position = UDim2.fromScale(0.5, 0.06),
			Size = UDim2.fromScale(0.84, 0.42),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(canopy, 99)
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.34, 0.58),
			Size = UDim2.new(0, 1.2, 0.34, 0),
			Rotation = -18,
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		create("Frame", {
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.fromScale(0.66, 0.58),
			Size = UDim2.new(0, 1.2, 0.34, 0),
			Rotation = 18,
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		local crate = create("Frame", {
			AnchorPoint = Vector2.new(0.5, 1),
			Position = UDim2.fromScale(0.5, 0.94),
			Size = UDim2.fromScale(0.32, 0.24),
			BackgroundColor3 = col,
			BorderSizePixel = 0,
			Parent = box
		})
		addCorner(crate, 2)
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

local Store = {
	Toggles = {
		KitRapido = true,
		SaborTelaParada = false,
		BackJump = false,
		GhostHack = false,
		AutoSuicide = false,
		AutoRevive = true,
		FirstPerson = false,
		NoRecoil = true,
		NoScope = false,
		NoPreFireDelay = true,
		FastSwitch = false,
		UpsideDown = false,
		KillCover = false,
		TeleportWall = false,
		CameraFOV = true,
		FlyHack = true,
		SpawnJet = false,
		TeleportWayPoint = false,
		TeleKill = true,
		PanicAirPlaneToggle = true,
		PanicAction = false,
		AutoSpawnM249 = true,
		AutoSpawnUMP = true,
		AutoSpawnAwake = true,
		AutoSpawnJetPack = true,
		AimbotEnabled = false,
		SilentAim = false,
		AutoFire = false,
		AimIgnoreKnocked = true,
		AimVisibleCheck = true,
		AimDrawLine = false,
		SpawnAWM = false,
		SpawnMedKit = false,
		SpawnGlooWall = false,
		SpawnVestLv4 = false,
		FPSBoost = false,
		RemoveShadows = false,
		LowTextures = false,
		CleanMemory = false,
		PingStabilizer = false,
		ESPEnabled = false,
		ESPBox = false,
		ESPSkeleton = false,
		ESPLines = false,
		ESPName = false,
		ESPHealth = false,
		ChamsGlow = false,
		CollapseAllBars = false
	},
	Sliders = {
		Angle = 180,
		FOV = 235,
		Height = -45,
		Speed = 378,
		Ignore = 300,
		AimSmooth = 15,
		AimDistance = 300,
		AutoLootDist = 15,
		FPSCap = 120,
		RenderScale = 100,
		ESPDistance = 250,
		MainScale = 100,
		FloatScale = 100
	},
	Keybinds = {
		GhostHack = nil,
		AutoSuicide = nil,
		TeleportWayPoint = Enum.KeyCode.F3,
		PanicAction = Enum.KeyCode.F2,
		FlyHack = Enum.KeyCode.F5,
		TeleKill = Enum.KeyCode.F6,
		SpawnJet = Enum.KeyCode.F8,
		KillCover = Enum.KeyCode.F9,
		AimbotEnabled = nil,
		MenuToggle = Enum.KeyCode.RightShift
	},
	KeybindLabels = {
		TeleportWayPoint = "Teleport WayPoint",
		PanicAction = "Panic AirPlane",
		FlyHack = "Fly Hack",
		TeleKill = "TeleKill",
		SpawnJet = "Spawn Jet",
		KillCover = "Kill Cover",
		GhostHack = "Ghost Hack",
		AutoSuicide = "Auto Suicide",
		AimbotEnabled = "Aimbot"
	},
	KeybindOrder = {
		"TeleportWayPoint",
		"PanicAction",
		"FlyHack",
		"TeleKill",
		"SpawnJet",
		"KillCover",
		"GhostHack",
		"AutoSuicide",
		"AimbotEnabled"
	},
	Windows = {
		PanicWindow = true,
		JetPackWindow = true,
		PlayerWindow = false,
		WeaponWindow = false,
		WorldWindow = false,
		AimbotWindow = false,
		VisualsWindow = false,
		KeybindOverlay = true,
		TimerPill = true,
		SpectatorPill = true,
		FOVCircle = true
	},
	SidebarTabsEnabled = {
		Aimbot = true,
		Exploits = true,
		SpawnItens = true,
		Otimizacao = true,
		Visuals = true,
		Personalizacao = true,
		Configuracoes = true
	},
	ToggleListeners = {},
	SliderListeners = {},
	KeybindListeners = {},
	WindowListeners = {},
	SidebarTabListeners = {}
}

local function onToggleChanged(id, fn)
	if not Store.ToggleListeners[id] then
		Store.ToggleListeners[id] = {}
	end
	table.insert(Store.ToggleListeners[id], fn)
end

local function onSliderChanged(id, fn)
	if not Store.SliderListeners[id] then
		Store.SliderListeners[id] = {}
	end
	table.insert(Store.SliderListeners[id], fn)
end

local function onKeybindChanged(fn)
	table.insert(Store.KeybindListeners, fn)
end

local function onWindowChanged(id, fn)
	if not Store.WindowListeners[id] then
		Store.WindowListeners[id] = {}
	end
	table.insert(Store.WindowListeners[id], fn)
end

local function onSidebarTabChanged(id, fn)
	if not Store.SidebarTabListeners[id] then
		Store.SidebarTabListeners[id] = {}
	end
	table.insert(Store.SidebarTabListeners[id], fn)
end

local function notifyKeybinds()
	for i = 1, #Store.KeybindListeners do
		pcall(Store.KeybindListeners[i])
	end
end

local DefaultCameraFOV = 70
pcall(function()
	if workspace.CurrentCamera then
		DefaultCameraFOV = workspace.CurrentCamera.FieldOfView
	end
end)

local ESPHighlights = {}
local function updatePlayerESP(enabled)
	if not enabled then
		for plr, hl in pairs(ESPHighlights) do
			pcall(function()
				hl:Destroy()
			end)
			ESPHighlights[plr] = nil
		end
		return
	end
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character then
			if not ESPHighlights[plr] or not ESPHighlights[plr].Parent then
				pcall(function()
					local hl = Instance.new("Highlight")
					hl.FillColor = Theme.Accent
					hl.OutlineColor = Theme.White
					hl.FillTransparency = 0.65
					hl.OutlineTransparency = 0.1
					hl.Adornee = plr.Character
					hl.Parent = plr.Character
					ESPHighlights[plr] = hl
				end)
			end
		end
	end
end

local function setWindowActive(winId, state)
	local b = state and true or false
	Store.Windows[winId] = b
	local list = Store.WindowListeners[winId]
	if list then
		for i = 1, #list do
			pcall(list[i], b)
		end
	end
end

local function setSidebarTabEnabled(tabId, state)
	local b = state and true or false
	Store.SidebarTabsEnabled[tabId] = b
	local list = Store.SidebarTabListeners[tabId]
	if list then
		for i = 1, #list do
			pcall(list[i], b)
		end
	end
end

local function setToggle(id, state)
	local b = state and true or false
	Store.Toggles[id] = b
	local list = Store.ToggleListeners[id]
	if list then
		for i = 1, #list do
			pcall(list[i], b)
		end
	end
	if Store.KeybindLabels[id] then
		notifyKeybinds()
	end
	if id == "FirstPerson" and LocalPlayer then
		pcall(function()
			LocalPlayer.CameraMode = b and Enum.CameraMode.LockFirstPerson or Enum.CameraMode.Classic
		end)
	elseif id == "RemoveShadows" then
		pcall(function()
			Lighting.GlobalShadows = not b
		end)
	elseif id == "ESPEnabled" or id == "ChamsGlow" then
		updatePlayerESP(Store.Toggles.ESPEnabled or Store.Toggles.ChamsGlow)
	elseif id == "CameraFOV" then
		setWindowActive("FOVCircle", b)
		if not b then
			pcall(function()
				if workspace.CurrentCamera then
					workspace.CurrentCamera.FieldOfView = DefaultCameraFOV
				end
			end)
		end
	elseif id == "PanicAirPlaneToggle" then
		setWindowActive("PanicWindow", b)
	end
end

local function setSlider(id, value)
	Store.Sliders[id] = value
	local list = Store.SliderListeners[id]
	if list then
		for i = 1, #list do
			pcall(list[i], value)
		end
	end
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

local function setKeybind(id, keyCode)
	Store.Keybinds[id] = keyCode
	notifyKeybinds()
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

if GuiParent then
	ScreenGui.Parent = GuiParent
end

local ViewportSize = Vector2.new(1366, 768)
pcall(function()
	if workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize.X > 0 then
		ViewportSize = workspace.CurrentCamera.ViewportSize
	end
end)
local InitialGlobalScale = clampNum(math.min(ViewportSize.X / 1320, ViewportSize.Y / 720), 0.56, 1.0)

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

local FOVFrame = create("Frame", {
	Name = "FOVCircle",
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.new(0, Store.Sliders.FOV * 2, 0, Store.Sliders.FOV * 2),
	BackgroundTransparency = 1,
	Visible = Store.Windows.FOVCircle,
	ZIndex = 1,
	Parent = ScreenGui
})
addCorner(FOVFrame, 9999)
local FOVStroke = addStroke(FOVFrame, Color3.fromRGB(244, 244, 246), 1.2, 0.08)

onSliderChanged("FOV", function(val)
	local d = math.floor(val * 2)
	FOVFrame.Size = UDim2.new(0, d, 0, d)
	if Store.Toggles.CameraFOV then
		pcall(function()
			if workspace.CurrentCamera then
				workspace.CurrentCamera.FieldOfView = clampNum(50 + (val / 450) * 65, 50, 120)
			end
		end)
	end
end)

onWindowChanged("FOVCircle", function(vis)
	FOVFrame.Visible = vis
end)

local function createHudPill(name, posUDim, widthPx, iconKind, defaultText, winKey)
	local pill = create("Frame", {
		Name = name,
		AnchorPoint = Vector2.new(0.5, 0),
		Position = posUDim,
		Size = UDim2.new(0, widthPx, 0, 24),
		BackgroundColor3 = Theme.BadgeBg,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Visible = Store.Windows[winKey],
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

	registerCollapsible({
		setCollapsed = setCollapsedState
	})

	onWindowChanged(winKey, function(vis)
		pill.Visible = vis
	end)

	return pill, label, pillScale
end

local TimerPill, TimerLabel = createHudPill("TimerPill", UDim2.new(0.46, 0, 0.115, 0), 74, "clock", "00:00", "TimerPill")
local SpecPill, SpecLabel = createHudPill("SpectatorPill", UDim2.new(0.498, 0, 0.18, 0), 52, "player", "0", "SpectatorPill")

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
		local otherCount = math.max(0, #Players:GetPlayers() - 1)
		SpecLabel.Text = tostring(otherCount > 0 and 0 or 0)
	end
end)

local function buildToggleRow(parent, labelText, toggleId)
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

	local switchBg = create("Frame", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 32, 0, 16),
		BackgroundColor3 = Store.Toggles[toggleId] and Theme.Accent or Theme.ToggleOff,
		BorderSizePixel = 0,
		Parent = row
	})
	addCorner(switchBg, 999)

	local knob = create("Frame", {
		AnchorPoint = Vector2.new(0, 0.5),
		Position = Store.Toggles[toggleId] and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
		Size = UDim2.new(0, 12, 0, 12),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		Parent = switchBg
	})
	addCorner(knob, 999)

	local function refreshVisual(val)
		tween(switchBg, 0.14, {
			BackgroundColor3 = val and Theme.Accent or Theme.ToggleOff
		})
		tween(knob, 0.14, {
			Position = val and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
		})
	end

	btnRow.MouseButton1Click:Connect(function()
		setToggle(toggleId, not Store.Toggles[toggleId])
	end)

	onToggleChanged(toggleId, refreshVisual)
	return row
end

local function buildWindowToggleRow(parent, labelText, winKey)
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

	local switchBg = create("Frame", {
		AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, 0, 0.5, 0),
		Size = UDim2.new(0, 32, 0, 16),
		BackgroundColor3 = Store.Windows[winKey] and Theme.Accent or Theme.ToggleOff,
		BorderSizePixel = 0,
		Parent = row
	})
	addCorner(switchBg, 999)

	local knob = create("Frame", {
		AnchorPoint = Vector2.new(0, 0.5),
		Position = Store.Windows[winKey] and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
		Size = UDim2.new(0, 12, 0, 12),
		BackgroundColor3 = Theme.White,
		BorderSizePixel = 0,
		Parent = switchBg
	})
	addCorner(knob, 999)

	btnRow.MouseButton1Click:Connect(function()
		setWindowActive(winKey, not Store.Windows[winKey])
	end)

	onWindowChanged(winKey, function(val)
		tween(switchBg, 0.14, {
			BackgroundColor3 = val and Theme.Accent or Theme.ToggleOff
		})
		tween(knob, 0.14, {
			Position = val and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0)
		})
	end)

	return row
end

local function buildKeybindRow(parent, labelText, keybindId)
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 24),
		Parent = parent
	})

	local labelBtn = create("TextButton", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(1, -56, 1, 0),
		Font = Enum.Font.GothamMedium,
		Text = labelText,
		TextColor3 = Store.Toggles[keybindId] and Theme.Accent or Theme.TextSecondary,
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
		Text = formatKeyCode(Store.Keybinds[keybindId]),
		TextColor3 = Theme.TextPrimary,
		TextSize = 10,
		AutoButtonColor = false,
		Parent = row
	})
	addCorner(keyBtn, 4)
	local stroke = addStroke(keyBtn, Theme.BadgeStroke, 1)

	labelBtn.MouseButton1Click:Connect(function()
		if Store.Toggles[keybindId] ~= nil then
			setToggle(keybindId, not Store.Toggles[keybindId])
		end
	end)

	if Store.Toggles[keybindId] ~= nil then
		onToggleChanged(keybindId, function(active)
			labelBtn.TextColor3 = active and Theme.Accent or Theme.TextSecondary
		end)
	end

	keyBtn.MouseButton1Click:Connect(function()
		if BindingTargetId == keybindId then
			BindingTargetId = nil
			notifyKeybinds()
			return
		end
		local prev = BindingTargetId
		BindingTargetId = keybindId
		if prev ~= nil then
			notifyKeybinds()
		end
		keyBtn.Text = "..."
		keyBtn.TextColor3 = Theme.Accent
		stroke.Color = Theme.Accent
	end)

	onKeybindChanged(function()
		if BindingTargetId ~= keybindId then
			keyBtn.Text = formatKeyCode(Store.Keybinds[keybindId])
			keyBtn.TextColor3 = Theme.TextPrimary
			stroke.Color = Theme.BadgeStroke
		end
	end)

	return row
end

local function buildSliderRow(parent, labelText, sliderId, minVal, maxVal, unitSuffix, showBadge)
	local row = create("Frame", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 24),
		Parent = parent
	})

	local labelWidth = 52
	create("TextLabel", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, 0, 0, 0),
		Size = UDim2.new(0, labelWidth, 1, 0),
		Font = Enum.Font.GothamMedium,
		Text = labelText,
		TextColor3 = Theme.TextSecondary,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = row
	})

	local rightReserve = showBadge and 52 or 2
	local hitArea = create("Frame", {
		BackgroundTransparency = 1,
		Position = UDim2.new(0, labelWidth + 4, 0, 0),
		Size = UDim2.new(1, -(labelWidth + 4 + rightReserve), 1, 0),
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

	local curVal = Store.Sliders[sliderId] or minVal
	local initAlpha = clampNum((curVal - minVal) / math.max(1, (maxVal - minVal)), 0, 1)

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

	local badgeLabel = nil
	if showBadge then
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
		badgeLabel = create("TextLabel", {
			BackgroundTransparency = 1,
			Size = UDim2.fromScale(1, 1),
			Font = Enum.Font.GothamMedium,
			Text = tostring(curVal) .. (unitSuffix or ""),
			TextColor3 = Theme.Accent,
			TextSize = 9.5,
			Parent = badge
		})
	end

	local function updateFromPositionX(screenX)
		local absPos = track.AbsolutePosition.X
		local absWidth = math.max(1, track.AbsoluteSize.X)
		local alpha = clampNum((screenX - absPos) / absWidth, 0, 1)
		local stepped = math.floor(minVal + alpha * (maxVal - minVal) + 0.5)
		setSlider(sliderId, stepped)
	end

	onSliderChanged(sliderId, function(v)
		local a = clampNum((v - minVal) / math.max(1, (maxVal - minVal)), 0, 1)
		fill.Size = UDim2.new(a, 0, 1, 0)
		thumb.Position = UDim2.new(a, 0, 0.5, 0)
		if badgeLabel then
			badgeLabel.Text = tostring(v) .. (unitSuffix or "")
		end
	end)

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
	local line2 = create("Frame", {
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
			local nextScale = clampNum(startScale + scaleDelta, 0.38, 1.55)
			scaleObj.Scale = nextScale
			local rawVertical = startHeight + (delta.Y / math.max(0.4, nextScale))
			if rawVertical <= minBarHeight + 16 then
				if onBarCollapseChanged then
					onBarCollapseChanged(true)
				end
			else
				if onBarCollapseChanged then
					onBarCollapseChanged(false, clampNum(rawVertical, minBarHeight + 60, baseHeight + 220))
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
					onBarCollapseChanged(false, clampNum(rawH, minBarHeight + 55, baseHeight + 220))
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
	Player = function(container)
		buildToggleRow(container, "Kit rapido", "KitRapido")
		buildToggleRow(container, "Sabor Tela Parada", "SaborTelaParada")
		buildToggleRow(container, "Back Jump", "BackJump")
		buildKeybindRow(container, "Ghost Hack", "GhostHack")
		buildKeybindRow(container, "Auto Suicide", "AutoSuicide")
		buildToggleRow(container, "Auto Revive", "AutoRevive")
		buildToggleRow(container, "First Person", "FirstPerson")
	end,
	Weapon = function(container)
		buildToggleRow(container, "No Recoil", "NoRecoil")
		buildToggleRow(container, "No Scope", "NoScope")
		buildToggleRow(container, "No Pre-Fire Delay", "NoPreFireDelay")
		buildToggleRow(container, "Fast Switch", "FastSwitch")
	end,
	World = function(container)
		buildToggleRow(container, "Upside Down", "UpsideDown")
		buildKeybindRow(container, "Kill Cover", "KillCover")
		buildSliderRow(container, "Angle", "Angle", 0, 180, "°", false)
		buildToggleRow(container, "Teleport Wall", "TeleportWall")
		buildToggleRow(container, "Camera FOV", "CameraFOV")
		buildSliderRow(container, "FOV", "FOV", 40, 450, "", false)
	end,
	JetPack = function(container)
		buildKeybindRow(container, "Fly Hack", "FlyHack")
		buildKeybindRow(container, "Spawn Jet", "SpawnJet")
		buildSliderRow(container, "Height", "Height", -200, 200, " m", true)
		buildSliderRow(container, "Speed", "Speed", 0, 500, " m/s", true)
		buildKeybindRow(container, "Teleport WayPoint", "TeleportWayPoint")
		buildKeybindRow(container, "TeleKill", "TeleKill")
		buildToggleRow(container, "Panic AirPlane", "PanicAirPlaneToggle")
	end,
	PanicAirPlane = function(container)
		buildToggleRow(container, "Auto Spawn M249", "AutoSpawnM249")
		buildToggleRow(container, "Auto Spawn UMP", "AutoSpawnUMP")
		buildToggleRow(container, "Auto Spawn Awake", "AutoSpawnAwake")
		buildToggleRow(container, "Auto Spawn JetPack", "AutoSpawnJetPack")
		buildSliderRow(container, "Ignore", "Ignore", 0, 2000, " ms", true)
		buildKeybindRow(container, "Panic AirPlane", "PanicAction")
	end,
	AimbotMain = function(container)
		buildToggleRow(container, "Enable Aimbot", "AimbotEnabled")
		buildToggleRow(container, "Silent Aim", "SilentAim")
		buildToggleRow(container, "Auto Fire", "AutoFire")
		buildKeybindRow(container, "Aimbot Key", "AimbotEnabled")
		buildSliderRow(container, "Smooth", "AimSmooth", 1, 100, "%", true)
		buildSliderRow(container, "FOV", "FOV", 40, 450, "", true)
	end,
	AimbotFilters = function(container)
		buildToggleRow(container, "Ignore Knocked", "AimIgnoreKnocked")
		buildToggleRow(container, "Visible Check", "AimVisibleCheck")
		buildToggleRow(container, "Draw Target Line", "AimDrawLine")
		buildSliderRow(container, "Distance", "AimDistance", 20, 600, " m", true)
		buildToggleRow(container, "Camera FOV", "CameraFOV")
	end,
	LootItems = function(container)
		buildToggleRow(container, "Spawn AWM", "SpawnAWM")
		buildToggleRow(container, "Spawn MedKit", "SpawnMedKit")
		buildToggleRow(container, "Spawn Gloo Wall", "SpawnGlooWall")
		buildToggleRow(container, "Spawn Vest Lv4", "SpawnVestLv4")
		buildSliderRow(container, "Auto Loot", "AutoLootDist", 1, 50, " m", true)
	end,
	Performance = function(container)
		buildToggleRow(container, "FPS Boost", "FPSBoost")
		buildSliderRow(container, "Max FPS", "FPSCap", 30, 240, "", true)
		buildToggleRow(container, "Remove Shadows", "RemoveShadows")
		buildToggleRow(container, "Low Textures", "LowTextures")
		buildToggleRow(container, "Clean Memory", "CleanMemory")
		buildToggleRow(container, "Ping Stabilizer", "PingStabilizer")
		buildSliderRow(container, "Render", "RenderScale", 50, 100, "%", true)
	end,
	VisualsESP = function(container)
		buildToggleRow(container, "Enable ESP", "ESPEnabled")
		buildToggleRow(container, "ESP Box", "ESPBox")
		buildToggleRow(container, "ESP Skeleton", "ESPSkeleton")
		buildToggleRow(container, "ESP Lines", "ESPLines")
		buildToggleRow(container, "ESP Name", "ESPName")
		buildToggleRow(container, "ESP Health", "ESPHealth")
		buildToggleRow(container, "Chams Glow", "ChamsGlow")
		buildSliderRow(container, "Distance", "ESPDistance", 50, 500, " m", true)
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
		Parent = header
	})

	if linkedWinKey then
		local pinBtn = create("TextButton", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, -26, 0.5, 0),
			Size = UDim2.new(0, 18, 0, 18),
			BackgroundColor3 = Store.Windows[linkedWinKey] and Theme.AccentDark or Theme.BadgeBg,
			BorderSizePixel = 0,
			Font = Enum.Font.GothamBold,
			Text = "⧉",
			TextColor3 = Store.Windows[linkedWinKey] and Theme.White or Theme.TextMuted,
			TextSize = 10,
			AutoButtonColor = false,
			Parent = header
		})
		addCorner(pinBtn, 4)

		pinBtn.MouseButton1Click:Connect(function()
			setWindowActive(linkedWinKey, not Store.Windows[linkedWinKey])
		end)

		onWindowChanged(linkedWinKey, function(active)
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

local function createFloatingWindow(winKey, titleText, iconKind, builderKey, defaultPos, rowCount)
	local fullHeight = 64 + (rowCount * 28)
	local win = create("Frame", {
		Name = winKey,
		Position = defaultPos,
		Size = UDim2.new(0, 206, 0, fullHeight),
		BackgroundColor3 = Theme.WindowBg,
		BorderSizePixel = 0,
		ClipsDescendants = true,
		Visible = Store.Windows[winKey],
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
		Size = UDim2.new(1, -48, 1, 0),
		Font = Enum.Font.GothamBold,
		Text = titleText,
		TextColor3 = Theme.White,
		TextSize = 11,
		TextXAlignment = Enum.TextXAlignment.Left,
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
			tween(win, 0.14, { Size = UDim2.new(0, 206, 0, 24) })
		else
			if customHeight then
				win.Size = UDim2.new(0, 206, 0, expandedHeight)
			else
				tween(win, 0.14, { Size = UDim2.new(0, 206, 0, expandedHeight) })
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
	attachResizeAndPullBar(win, winScale, 206, fullHeight, 24, applyCollapse)

	registerCollapsible({
		setCollapsed = function(c)
			applyCollapse(c)
		end
	})

	onWindowChanged(winKey, function(vis)
		win.Visible = vis
	end)

	return win
end

createFloatingWindow("PanicWindow", "Panic AirPlane", "panic", "PanicAirPlane", UDim2.new(0.068, 0, 0.375, 0), 6)
createFloatingWindow("JetPackWindow", "JetPack", "jetpack", "JetPack", UDim2.new(0.208, 0, 0.232, 0), 7)
createFloatingWindow("PlayerWindow", "Player", "player", "Player", UDim2.new(0.355, 0, 0.24, 0), 7)
createFloatingWindow("WeaponWindow", "Weapon", "weapon", "Weapon", UDim2.new(0.355, 0, 0.56, 0), 4)
createFloatingWindow("WorldWindow", "World", "world", "World", UDim2.new(0.208, 0, 0.58, 0), 6)
createFloatingWindow("AimbotWindow", "Aimbot", "aimbot", "AimbotMain", UDim2.new(0.068, 0, 0.12, 0), 6)
createFloatingWindow("VisualsWindow", "Visuals", "visuals", "VisualsESP", UDim2.new(0.208, 0, 0.08, 0), 8)

local KeybindBox = create("Frame", {
	Name = "KeybindOverlay",
	Position = UDim2.new(0.818, 0, 0.415, 0),
	Size = UDim2.new(0, 202, 0, 158),
	BackgroundColor3 = Theme.OverlayBg,
	BackgroundTransparency = 0.04,
	BorderSizePixel = 0,
	ClipsDescendants = true,
	Visible = Store.Windows.KeybindOverlay,
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

local KbRedLine = create("Frame", {
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
	for i = 1, #Store.KeybindOrder do
		local id = Store.KeybindOrder[i]
		local kc = Store.Keybinds[id]
		if kc then
			visibleRows = visibleRows + 1
			local isActive = Store.Toggles[id] and true or false
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
				Text = Store.KeybindLabels[id] or id,
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
				if Store.Toggles[id] ~= nil then
					setToggle(id, not Store.Toggles[id])
				end
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

onKeybindChanged(rebuildKeybindList)
onWindowChanged("KeybindOverlay", function(vis)
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
	Size = UDim2.new(1, -16, 1, -144),
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
	AnchorPoint = Vector2.new(0, 1),
	Position = UDim2.new(0, 8, 1, -8),
	Size = UDim2.new(1, -16, 0, 36),
	BackgroundColor3 = Theme.CategoryBg,
	BorderSizePixel = 0,
	ZIndex = 12,
	Parent = Sidebar
})
addCorner(UserCard, 6)
addStroke(UserCard, Theme.CategoryStroke, 1)

local AvatarWrap = create("Frame", {
	Position = UDim2.new(0, 5, 0.5, -13),
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
	Size = UDim2.new(1, -140, 1, 0),
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
	Size = UDim2.new(0, 224, 0, 308),
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

buildWindowToggleRow(DrawerScroll, "Aba Panic AirPlane", "PanicWindow")
buildWindowToggleRow(DrawerScroll, "Aba JetPack", "JetPackWindow")
buildWindowToggleRow(DrawerScroll, "Aba Player", "PlayerWindow")
buildWindowToggleRow(DrawerScroll, "Aba Weapon", "WeaponWindow")
buildWindowToggleRow(DrawerScroll, "Aba World", "WorldWindow")
buildWindowToggleRow(DrawerScroll, "Aba Aimbot", "AimbotWindow")
buildWindowToggleRow(DrawerScroll, "Aba Visuals", "VisualsWindow")
buildWindowToggleRow(DrawerScroll, "Keybind List", "KeybindOverlay")
buildWindowToggleRow(DrawerScroll, "Barra Timer (00:00)", "TimerPill")
buildWindowToggleRow(DrawerScroll, "Barra Contador (0)", "SpectatorPill")
buildWindowToggleRow(DrawerScroll, "Circulo FOV", "FOVCircle")

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
			Visible = Store.SidebarTabsEnabled[st.Id] ~= false,
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

		onSidebarTabChanged(st.Id, function(enabled)
			btn.Visible = enabled
			if not enabled and CurrentTabId == st.Id then
				for fallbackId, isOn in pairs(Store.SidebarTabsEnabled) do
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
createSidebarCategory(2, "Memory", "memory", {
	{ Id = "Exploits", Label = "Exploits" },
	{ Id = "SpawnItens", Label = "Spawn Itens" },
	{ Id = "Otimizacao", Label = "Otimização" }
})
createSidebarCategory(3, "Visuals", "visuals", {
	{ Id = "Visuals", Label = "Visuals" }
})
createSidebarCategory(4, "Miscellaneous", "misc", {
	{ Id = "Personalizacao", Label = "Personalização" },
	{ Id = "Configuracoes", Label = "Configurações" }
})

local PageAimbot = createTabPage("Aimbot")
createSectionCard(PageAimbot.Left, "Aimbot", "aimbot", "AimbotMain", "AimbotWindow")
createSectionCard(PageAimbot.Right, "Aim Filters", "world", "AimbotFilters", nil)

local PageExploits = createTabPage("Exploits")
createSectionCard(PageExploits.Left, "Player", "player", "Player", "PlayerWindow")
createSectionCard(PageExploits.Left, "Weapon", "weapon", "Weapon", "WeaponWindow")
createSectionCard(PageExploits.Right, "World", "world", "World", "WorldWindow")
createSectionCard(PageExploits.Right, "JetPack", "jetpack", "JetPack", "JetPackWindow")

local PageSpawn = createTabPage("SpawnItens")
createSectionCard(PageSpawn.Left, "Panic AirPlane", "panic", "PanicAirPlane", "PanicWindow")
createSectionCard(PageSpawn.Right, "Spawn Itens", "weapon", "LootItems", nil)

local PageOpt = createTabPage("Otimizacao")
createSectionCard(PageOpt.Left, "Otimização", "memory", "Performance", nil)
createSectionCard(PageOpt.Right, "World", "world", "World", "WorldWindow")

local PageVisuals = createTabPage("Visuals")
createSectionCard(PageVisuals.Left, "Visuals", "visuals", "VisualsESP", "VisualsWindow")
createSectionCard(PageVisuals.Right, "Overlays", "misc", function(body)
	buildWindowToggleRow(body, "Keybind List", "KeybindOverlay")
	buildWindowToggleRow(body, "Timer (00:00)", "TimerPill")
	buildWindowToggleRow(body, "Spectator (0)", "SpectatorPill")
	buildWindowToggleRow(body, "FOV Circle", "FOVCircle")
end, nil)

local PagePers = createTabPage("Personalizacao")
createSectionCard(PagePers.Left, "Abas Flutuantes", "misc", function(body)
	buildWindowToggleRow(body, "Panic AirPlane", "PanicWindow")
	buildWindowToggleRow(body, "JetPack", "JetPackWindow")
	buildWindowToggleRow(body, "Player", "PlayerWindow")
	buildWindowToggleRow(body, "Weapon", "WeaponWindow")
	buildWindowToggleRow(body, "World", "WorldWindow")
	buildWindowToggleRow(body, "Aimbot", "AimbotWindow")
	buildWindowToggleRow(body, "Visuals", "VisualsWindow")
	buildWindowToggleRow(body, "Keybind List", "KeybindOverlay")
	buildWindowToggleRow(body, "Barra Timer", "TimerPill")
	buildWindowToggleRow(body, "Barra Contador", "SpectatorPill")
end, nil)

createSectionCard(PagePers.Right, "Escala & Abas Menu", "visuals", function(body)
	buildSliderRow(body, "Menu", "MainScale", 45, 145, "%", true)
	buildSliderRow(body, "Flutuante", "FloatScale", 45, 145, "%", true)
	local sidebarItems = {
		{ Id = "Aimbot", Label = "Aba Aimbot" },
		{ Id = "Exploits", Label = "Aba Exploits" },
		{ Id = "SpawnItens", Label = "Aba Spawn Itens" },
		{ Id = "Otimizacao", Label = "Aba Otimização" },
		{ Id = "Visuals", Label = "Aba Visuals" },
		{ Id = "Configuracoes", Label = "Aba Configurações" }
	}
	for i = 1, #sidebarItems do
		local item = sidebarItems[i]
		local row = create("Frame", {
			BackgroundTransparency = 1,
			Size = UDim2.new(1, 0, 0, 24),
			Parent = body
		})
		local btn = create("TextButton", {
			BackgroundTransparency = 1,
			Size = UDim2.fromScale(1, 1),
			Text = "",
			AutoButtonColor = false,
			Parent = row
		})
		create("TextLabel", {
			BackgroundTransparency = 1,
			Size = UDim2.new(1, -40, 1, 0),
			Font = Enum.Font.GothamMedium,
			Text = item.Label,
			TextColor3 = Theme.TextSecondary,
			TextSize = 11,
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = row
		})
		local sw = create("Frame", {
			AnchorPoint = Vector2.new(1, 0.5),
			Position = UDim2.new(1, 0, 0.5, 0),
			Size = UDim2.new(0, 32, 0, 16),
			BackgroundColor3 = Store.SidebarTabsEnabled[item.Id] and Theme.Accent or Theme.ToggleOff,
			BorderSizePixel = 0,
			Parent = row
		})
		addCorner(sw, 999)
		local kn = create("Frame", {
			AnchorPoint = Vector2.new(0, 0.5),
			Position = Store.SidebarTabsEnabled[item.Id] and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0),
			Size = UDim2.new(0, 12, 0, 12),
			BackgroundColor3 = Theme.White,
			BorderSizePixel = 0,
			Parent = sw
		})
		addCorner(kn, 999)
		btn.MouseButton1Click:Connect(function()
			local nextVal = not Store.SidebarTabsEnabled[item.Id]
			setSidebarTabEnabled(item.Id, nextVal)
			tween(sw, 0.14, { BackgroundColor3 = nextVal and Theme.Accent or Theme.ToggleOff })
			tween(kn, 0.14, { Position = nextVal and UDim2.new(1, -14, 0.5, 0) or UDim2.new(0, 2, 0.5, 0) })
		end)
	end
end, nil)

onSliderChanged("MainScale", function(pct)
	MainScale.Scale = clampNum((pct / 100) * InitialGlobalScale, 0.38, 1.55)
end)

onSliderChanged("FloatScale", function(pct)
	local s = clampNum((pct / 100) * InitialGlobalScale, 0.38, 1.55)
	for i = 1, #FloatingScales do
		FloatingScales[i].Scale = s
	end
end)

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
		SubtitleHolder.Size = UDim2.new(1, -238, 1, 0)
		tween(MainWindow, 0.15, { Size = UDim2.new(0, MainBaseWidth, 0, 38) })
	else
		TopHeader.Position = UDim2.new(0, SidebarWidth + 1, 0, 0)
		TopHeader.Size = UDim2.new(1, -(SidebarWidth + 1), 0, 38)
		SubtitleHolder.Position = UDim2.new(0, 28, 0, 0)
		SubtitleHolder.Size = UDim2.new(1, -140, 1, 0)
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

local PageConfig = createTabPage("Configuracoes")
createSectionCard(PageConfig.Left, "Configurações", "misc", function(body)
	buildKeybindRow(body, "Tecla do Painel", "MenuToggle")

	local foldAllBtn = create("TextButton", {
		BackgroundColor3 = Theme.FloatTitleBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 26),
		Font = Enum.Font.GothamBold,
		Text = "Guardar / Expandir Todas Barras",
		TextColor3 = Theme.White,
		TextSize = 10.5,
		AutoButtonColor = false,
		Parent = body
	})
	addCorner(foldAllBtn, 5)
	foldAllBtn.MouseButton1Click:Connect(function()
		AllBarsFolded = not AllBarsFolded
		for i = 1, #AllCollapsibleEntries do
			pcall(AllCollapsibleEntries[i].setCollapsed, AllBarsFolded)
		end
		setMainCollapsedToBar(AllBarsFolded)
	end)

	local resetBtn = create("TextButton", {
		BackgroundColor3 = Theme.CategoryBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 26),
		Font = Enum.Font.GothamBold,
		Text = "Resetar Escala & Posições",
		TextColor3 = Theme.TextPrimary,
		TextSize = 10.5,
		AutoButtonColor = false,
		Parent = body
	})
	addCorner(resetBtn, 5)
	addStroke(resetBtn, Theme.CategoryStroke, 1)
	resetBtn.MouseButton1Click:Connect(function()
		setSlider("MainScale", 100)
		setSlider("FloatScale", 100)
		setMainCollapsedToBar(false, MainBaseHeight)
		MainWindow.Position = UDim2.new(0.606, 0, 0.062, 0)
	end)
end, nil)

createSectionCard(PageConfig.Right, PANEL_NAME, "player", function(body)
	local infoA = create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 20),
		Font = Enum.Font.GothamBold,
		Text = PANEL_NAME,
		TextColor3 = Theme.Accent,
		TextSize = 11.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = body
	})
	local infoB = create("TextLabel", {
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 18),
		Font = Enum.Font.GothamMedium,
		Text = PANEL_SUBTITLE,
		TextColor3 = Theme.TextSecondary,
		TextSize = 10.5,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = body
	})
	local unloadBtn = create("TextButton", {
		BackgroundColor3 = Theme.BadgeBg,
		BorderSizePixel = 0,
		Size = UDim2.new(1, 0, 0, 26),
		Font = Enum.Font.GothamBold,
		Text = "Fechar Painel",
		TextColor3 = Theme.Accent,
		TextSize = 10.5,
		AutoButtonColor = false,
		Parent = body
	})
	addCorner(unloadBtn, 5)
	addStroke(unloadBtn, Theme.FloatStroke, 1)
	unloadBtn.MouseButton1Click:Connect(function()
		if env and type(env.RedxPainelUnload) == "function" then
			pcall(env.RedxPainelUnload)
		end
	end)
	return infoA, infoB
end, nil)

selectSubTab("Exploits")

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
			setKeybind(target, nil)
		else
			setKeybind(target, kc)
		end
		return
	end

	if gameProcessed then
		return
	end

	if Store.Keybinds.MenuToggle and kc == Store.Keybinds.MenuToggle then
		if MiniRestorePill.Visible then
			MiniRestorePill.Visible = false
			MainWindow.Visible = true
		else
			MainWindow.Visible = not MainWindow.Visible
		end
		return
	end

	for id, boundKc in pairs(Store.Keybinds) do
		if id ~= "MenuToggle" and boundKc == kc and Store.Toggles[id] ~= nil then
			setToggle(id, not Store.Toggles[id])
		end
	end
end))

trackConn(RunService.RenderStepped:Connect(function(dt)
	if Store.Toggles.FlyHack and LocalPlayer and LocalPlayer.Character then
		pcall(function()
			local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
			local cam = workspace.CurrentCamera
			if hrp and hum and cam then
				local moveDir = hum.MoveDirection
				local spd = clampNum(Store.Sliders.Speed * 0.22, 8, 160)
				local yOffset = clampNum(Store.Sliders.Height * 0.05, -25, 25)
				if moveDir.Magnitude > 0 then
					hrp.AssemblyLinearVelocity = Vector3.new(moveDir.X * spd, yOffset, moveDir.Z * spd)
				end
			end
		end)
	end
end))

local function unloadAll()
	for i = 1, #Connections do
		pcall(function()
			Connections[i]:Disconnect()
		end)
	end
	Connections = {}
	updatePlayerESP(false)
	pcall(function()
		if workspace.CurrentCamera then
			workspace.CurrentCamera.FieldOfView = DefaultCameraFOV
		end
	end)
	if ScreenGui then
		pcall(function()
			ScreenGui:Destroy()
		end)
	end
end

if env then
	env.RedxPainelUnload = unloadAll
	env.RedxPainelSetLogo = resolveLogoUrl
end

resolveLogoUrl(LOGO_LINK)
