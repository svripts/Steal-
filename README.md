repeat task.wait() until game:IsLoaded()

-- ============================================================
-- INTRO SCREEN
-- ============================================================

local plr = game.Players.LocalPlayer
local gui = plr:WaitForChild("PlayerGui")
local ts = game:GetService("TweenService")

local introSg = Instance.new("ScreenGui")
introSg.IgnoreGuiInset = true
introSg.ResetOnSpawn = false
introSg.Parent = gui

local black = Instance.new("Frame")
black.Size = UDim2.new(1,0,1,0)
black.BackgroundColor3 = Color3.fromRGB(0,0,0)
black.BackgroundTransparency = 1
black.Parent = introSg

local txt = Instance.new("TextLabel")
txt.Size = UDim2.new(1,0,1,0)
txt.BackgroundTransparency = 1
txt.TextColor3 = Color3.fromRGB(259,259,259)
txt.TextTransparency = 1
txt.Font = Enum.Font.GothamBold
txt.TextScaled = true
txt.Parent = White

local tin = TweenInfo.new(0.9, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local tout = TweenInfo.new(0.9, Enum.EasingStyle.Quad, Enum.EasingDirection.In)

local function show(text)
    txt.Text = text
    ts:Create(txt, tin, {TextTransparency = 0}):Play()
    task.wait(2.2)
    ts:Create(txt, tout, {TextTransparency = 1}):Play()
    task.wait(0.9)
end

ts:Create(black, tin, {BackgroundTransparency = 0}):Play()
task.wait(0.8)

show("TIKI TIKI TIKI CS YARA YARA YARA YA VS MIGUEL MIGUEL")
show("MIGUEL WON | YOU GOT NIGGERIZED")
show("THIS SOURE IS MADE BY BANG BRO'S OFFICIAL")

ts:Create(black, tout, {BackgroundTransparency = 1}):Play()
task.wait(1)

introSg:Destroy()

-- ============================================================
-- MAIN SCRIPT
-- ============================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local Player = Players.LocalPlayer

local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local guiScale = isMobile and 0.4 or 1

-- Set GUI size based on platform
local guiWidth = isMobile and 198 or 340
local guiHeight = isMobile and 240 or 410

local sg = Instance.new("ScreenGui")
sg.Name = "FEMBOYS_VS_INDIANS_VS_GAYS_UI"
sg.ResetOnSpawn = false
sg.Parent = Player.PlayerGui

-- ============================================================
-- CONFIG SYSTEM
-- ============================================================

local FolderName = "EvoDuels"
local FileName = FolderName .. "/config.json"

-- Default Config table
local Config = {
    -- Feature States
    ["Lagger Mode"] = false,
    ["Auto Bat"] = false,
    ["Unwalk Mode"] = false,
    ["Bat Counter"] = false,
    ["Medusa Counter"] = false,
    ["Anti-Ragdoll"] = false,
    ["Stretch Rez"] = false,
    ["Infinite Jump"] = false,
    ["Infinite Jump Mode"] = "Hold",
    ["Auto Left"] = false,
    ["Auto Right"] = false,
    ["Auto Steal"] = false,
    ["Anti-Lag"] = false,
    
    -- Speed Values
    ["Speed Boost Value"] = 53,
    ["Carry Mode Value"] = 29,
    ["Lagger Boost Value"] = 10.1,
    ["Lagger Steal Value"] = 8,
    
    -- Auto Steal Values
    ["Steal Radius"] = 60,
    ["Steal Duration"] = 1.4,
    
    -- TP Height
    ["TP Height"] = -7,
    
    -- Keybinds
    ["Keybind Auto Left"] = "Z",
    ["Keybind Auto Right"] = "C",
    ["Keybind Auto Bat"] = "X",
    ["Keybind Drop"] = "Q",
    ["Keybind TP Down"] = "G",
    ["Keybind Lagger Mode"] = "V",
    
    -- Button Positions
    ["Button1 X"] = 1,
    ["Button1 Y"] = 0.5,
    ["Button1 OffsetX"] = -140,
    ["Button1 OffsetY"] = -170,
    ["Button2 X"] = 1,
    ["Button2 Y"] = 0.5,
    ["Button2 OffsetX"] = -68,
    ["Button2 OffsetY"] = -170,
    ["Button3 X"] = 1,
    ["Button3 Y"] = 0.5,
    ["Button3 OffsetX"] = -140,
    ["Button3 OffsetY"] = -102,
    ["Button4 X"] = 1,
    ["Button4 Y"] = 0.5,
    ["Button4 OffsetX"] = -68,
    ["Button4 OffsetY"] = -102,
    ["Button5 X"] = 1,
    ["Button5 Y"] = 0.5,
    ["Button5 OffsetX"] = -140,
    ["Button5 OffsetY"] = -34,
    ["Button6 X"] = 1,
    ["Button6 Y"] = 0.5,
    ["Button6 OffsetX"] = -68,
    ["Button6 OffsetY"] = -34,
    
    -- EVO Button Position
    ["EvoButton X"] = 0,
    ["EvoButton Y"] = 0,
    ["EvoButton OffsetX"] = 10,
    ["EvoButton OffsetY"] = 10,
    
    -- Main GUI Position
    ["MainGUI X"] = 0,
    ["MainGUI Y"] = 0.5,
    ["MainGUI OffsetX"] = 10,
    ["MainGUI OffsetY"] = -200,
    ["MainGUI Visible"] = true,
}

-- Create folder if doesn't exist
if not isfolder(FolderName) then
    makefolder(FolderName)
end

-- Create default config file if doesn't exist
if not isfile(FileName) then
    writefile(FileName, HttpService:JSONEncode(Config))
end

-- LOAD CONFIG FUNCTION
local function LoadConfig()
    local success, data = pcall(readfile, FileName)
    
    if success then
        local decoded = HttpService:JSONDecode(data)
        for key, value in pairs(decoded) do
            Config[key] = value
        end
    end
end

-- SAVE CONFIG FUNCTION
function SaveConfig()
    writefile(FileName, HttpService:JSONEncode(Config))
end

-- Load saved config
LoadConfig()

-- ============================================================
-- ANTI-LAG FUNCTIONS
-- ============================================================

local antiLagEnabled = Config["Anti-Lag"]
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local defLightBrightness, defLightClock, defLightAmbient

local function applyAntiLagDerender(obj)
    pcall(function()
        if obj:IsA("Accessory") or obj:IsA("Hat") then 
            obj:Destroy()
        elseif obj:IsA("BasePart") then 
            obj.Material = Enum.Material.Plastic
            obj.Reflectance = 0
            obj.CastShadow = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then 
            obj.Transparency = 1
        elseif obj:IsA("ParticleEmitter") or 
               obj:IsA("Trail") or 
               obj:IsA("Beam") or 
               obj:IsA("Fire") or 
               obj:IsA("Smoke") or 
               obj:IsA("Sparkles") then 
            obj.Enabled = false
        elseif obj:IsA("AnimationController") or obj:IsA("Animator") then
            for _, t in ipairs(obj:GetPlayingAnimationTracks()) do 
                pcall(function() t:Stop(0) end) 
            end
        end
    end)
end

local function enableAntiLag()
    removeAccessoriesEnabled = true
    antiLagEnabled = true
    Config["Anti-Lag"] = true
    SaveConfig()
    
    defLightBrightness = defLightBrightness or Lighting.Brightness
    defLightClock = defLightClock or Lighting.ClockTime
    defLightAmbient = defLightAmbient or Lighting.OutdoorAmbient
    
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 1e10
    Lighting.Brightness = 1
    Lighting.EnvironmentDiffuseScale = 0
    Lighting.EnvironmentSpecularScale = 0
    
    for _, e in pairs(Lighting:GetChildren()) do
        pcall(function()
            if e:IsA("BlurEffect") or 
               e:IsA("SunRaysEffect") or 
               e:IsA("ColorCorrectionEffect") or 
               e:IsA("BloomEffect") or 
               e:IsA("DepthOfFieldEffect") then
                e.Enabled = false
            end
        end)
    end
    
    for _, obj in ipairs(workspace:GetDescendants()) do 
        applyAntiLagDerender(obj) 
    end
    
    if antiLagDescConn then antiLagDescConn:Disconnect() end
    
    antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
        if removeAccessoriesEnabled then 
            applyAntiLagDerender(obj) 
        end
    end)
end

local function disableAntiLag()
    removeAccessoriesEnabled = false
    antiLagEnabled = false
    Config["Anti-Lag"] = false
    SaveConfig()
    
    if antiLagDescConn then 
        antiLagDescConn:Disconnect()
        antiLagDescConn = nil 
    end
    
    pcall(function()
        if defLightBrightness then Lighting.Brightness = defLightBrightness end
        if defLightClock then Lighting.ClockTime = defLightClock end
        if defLightAmbient then Lighting.OutdoorAmbient = defLightAmbient end
        Lighting.ExposureCompensation = 0
    end)
end

local function toggleAntiLag()
    if antiLagEnabled then
        disableAntiLag()
    else
        enableAntiLag()
    end
end

-- ============================================================
-- KEYBIND HELPER FUNCTIONS
-- ============================================================

local keybindMap = {
    ["Z"] = Enum.KeyCode.Z,
    ["C"] = Enum.KeyCode.C,
    ["X"] = Enum.KeyCode.X,
    ["Q"] = Enum.KeyCode.Q,
    ["G"] = Enum.KeyCode.G,
    ["V"] = Enum.KeyCode.V,
    ["U"] = Enum.KeyCode.U,
}

-- ============================================================
-- SPEED FUNCTIONS
-- ============================================================

local humanoid = nil
local humanoidRootPart = nil

local function getCharacter()
    local character = Player.Character or Player.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
end
getCharacter()

Player.CharacterAdded:Connect(function(character)
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
end)

local normalSpeedValue = Config["Speed Boost Value"]
local carrySpeedValue = Config["Carry Mode Value"]

local function updateSpeedValues()
    normalSpeedValue = Config["Speed Boost Value"]
    carrySpeedValue = Config["Carry Mode Value"]
end

-- ============================================================
-- LAGGER MODE FUNCTIONS
-- ============================================================

local laggerBoostValue = Config["Lagger Boost Value"]
local laggerStealValue = Config["Lagger Steal Value"]
local isLaggerModeActive = Config["Lagger Mode"]

local function setLaggerBoost(value)
    laggerBoostValue = math.clamp(value, 1, 200)
    Config["Lagger Boost Value"] = laggerBoostValue
    SaveConfig()
end

local function setLaggerSteal(value)
    laggerStealValue = math.clamp(value, 1, 200)
    Config["Lagger Steal Value"] = laggerStealValue
    SaveConfig()
end

local function handleSpeed()
    if not humanoid or not humanoidRootPart then return end
    
    local moveDirection = humanoid.MoveDirection
    if moveDirection.Magnitude == 0 then return end
    
    local currentSpeed
    
    if isLaggerModeActive then
        local isStealing = humanoid.WalkSpeed < 25
        currentSpeed = isStealing and laggerStealValue or laggerBoostValue
    else
        local isCarrying = humanoid.WalkSpeed < 25
        currentSpeed = isCarrying and carrySpeedValue or normalSpeedValue
    end
    
    humanoidRootPart.Velocity = Vector3.new(
        moveDirection.X * currentSpeed,
        humanoidRootPart.Velocity.Y,
        moveDirection.Z * currentSpeed
    )
end

local function toggleMode()
    isLaggerModeActive = not isLaggerModeActive
    Config["Lagger Mode"] = isLaggerModeActive
    SaveConfig()
    
    if button1 then
        if isLaggerModeActive then
            button1.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            button1.TextColor3 = Color3.fromRGB(0, 0, 0)
        else
            button1.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            button1.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
    end
end

task.spawn(function()
    while true do
        task.wait()
        if humanoid and humanoidRootPart and humanoid.Parent then
            handleSpeed()
        end
    end
end)

-- ============================================================
-- TP DOWN FUNCTIONS (MANUAL)
-- ============================================================

local tpHeightValue = Config["TP Height"]

local function doTPDown()
    local char = Player.Character
    if not char then return end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    hrp.CFrame = CFrame.new(hrp.Position.X, tpHeightValue, hrp.Position.Z) 
        * CFrame.Angles(0, select(2, hrp.CFrame:ToEulerAnglesYXZ()), 0)
    
    hrp.AssemblyLinearVelocity = Vector3.zero
end

-- ============================================================
-- DROP BRAINROT FUNCTIONS
-- ============================================================

local DROP_ASCEND_DURATION = 0.2
local DROP_ASCEND_SPEED = 150
local dropBrainrotActive = false

local function runDropBrainrot()
    if dropBrainrotActive then return end
    local char = Player.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    dropBrainrotActive = true
    local t0 = tick()
    local dc
    
    dc = RunService.Heartbeat:Connect(function()
        local r = char and char:FindFirstChild("HumanoidRootPart")
        if not r then 
            dc:Disconnect()
            dropBrainrotActive = false
            return 
        end
        if tick() - t0 >= DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp = RaycastParams.new()
            rp.FilterDescendantsInstances = {char}
            rp.FilterType = Enum.RaycastFilterType.Exclude
            local rr = workspace:Raycast(r.Position, Vector3.new(0, -2000, 0), rp)
            if rr then
                local hum2 = char:FindFirstChildOfClass("Humanoid")
                local off = (hum2 and hum2.HipHeight or 2) + (r.Size.Y / 2)
                r.CFrame = CFrame.new(r.Position.X, rr.Position.Y + off, r.Position.Z)
                r.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            end
            dropBrainrotActive = false
            return
        end
        r.AssemblyLinearVelocity = Vector3.new(r.AssemblyLinearVelocity.X, DROP_ASCEND_SPEED, r.AssemblyLinearVelocity.Z)
    end)
end

local function toggleDropBrainrot()
    runDropBrainrot()
    if button3 then
        button3.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        button3.TextColor3 = Color3.fromRGB(0, 0, 0)
        task.wait(0.2)
        button3.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        button3.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
end

-- ============================================================
-- AUTO BAT FUNCTIONS
-- ============================================================

local autoBatEnabled = Config["Auto Bat"]
local autoSwingEnabled = true
local autoBatEquippedThisRun = false
local _autoBatTarget = nil
local _autoBatLastScan = 0
local AUTO_BAT_SPEED = 58
local AUTO_BAT_VERT_SPEED = 52
local AUTO_BAT_DIST = -2.8
local AUTO_BAT_HEIGHT = 4.75
local AUTO_BAT_V_OFF = 1
local AUTO_BAT_TURN_SPEED = 285
local AUTO_BAT_MAX_TURN_RATE = 28

local function getAutoBatTarget()
    local root = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    
    local now = tick()
    if now - _autoBatLastScan <= 0.1 and _autoBatTarget and _autoBatTarget.Parent then
        local hum = _autoBatTarget.Parent:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 then return _autoBatTarget end
    end
    
    _autoBatLastScan = now
    _autoBatTarget = nil
    
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then 
                    minDist = dist
                    closest = tRoot 
                end
            end
        end
    end
    _autoBatTarget = closest
    return _autoBatTarget
end

local function resetAutoBatMotion()
    local char = Player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hrp then 
        hrp.AssemblyLinearVelocity = hrp.AssemblyLinearVelocity * 0.3
        hrp.AssemblyAngularVelocity = Vector3.zero 
    end
    if hum then hum.AutoRotate = true end
end

local function toggleAutoBat()
    autoBatEnabled = not autoBatEnabled
    Config["Auto Bat"] = autoBatEnabled
    SaveConfig()
    
    if autoBatEnabled then
        autoBatEquippedThisRun = false
        if button4 then
            button4.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            button4.TextColor3 = Color3.fromRGB(0, 0, 0)
        end
    else
        autoBatEquippedThisRun = false
        local char = Player.Character
        if char then
            local hum2 = char:FindFirstChildOfClass("Humanoid")
            if hum2 then hum2.AutoRotate = true end
        end
        resetAutoBatMotion()
        if button4 then
            button4.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            button4.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
    end
end

RunService.Heartbeat:Connect(function()
    if not autoBatEnabled then return end
    
    local char = Player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root or not hum then return end
    
    if not autoBatEquippedThisRun then
        autoBatEquippedThisRun = true
        if not char:FindFirstChildOfClass("Tool") then
            local bp = Player:FindFirstChildOfClass("Backpack") or Player:FindFirstChild("Backpack")
            local bpBat = bp and bp:FindFirstChild("Bat")
            if bpBat then pcall(function() hum:EquipTool(bpBat) end) end
        end
    end
    
    local target = getAutoBatTarget()
    
    if target then
        local targetVel = target.AssemblyLinearVelocity
        local aimTargetPos = target.Position + (targetVel * math.clamp(targetVel.Magnitude / 130, 0.05, 0.15)) + Vector3.new(0, AUTO_BAT_V_OFF, 0)
        
        hum.AutoRotate = false
        local look = aimTargetPos - root.Position
        local flatLook = Vector3.new(look.X, 0, look.Z)
        
        if look.Magnitude > 0.01 and flatLook.Magnitude > 0.01 then
            local targetYaw = math.deg(math.atan2(-flatLook.X, -flatLook.Z))
            local yawDelta = (targetYaw - root.Orientation.Y + 180) % 360 - 180
            local targetPitch = math.deg(math.atan2(look.Y, flatLook.Magnitude))
            local pitchDelta = (targetPitch - root.Orientation.X + 180) % 360 - 180
            local yawRate = math.clamp(math.rad(yawDelta) * AUTO_BAT_TURN_SPEED, -AUTO_BAT_MAX_TURN_RATE, AUTO_BAT_MAX_TURN_RATE)
            local pitchRate = math.clamp(math.rad(pitchDelta) * AUTO_BAT_TURN_SPEED, -AUTO_BAT_MAX_TURN_RATE, AUTO_BAT_MAX_TURN_RATE)
            local yawRad = math.rad(root.Orientation.Y)
            local rightAxis = Vector3.new(math.cos(yawRad), 0, -math.sin(yawRad))
            root.AssemblyAngularVelocity = Vector3.new(0, yawRate, 0) + (rightAxis * pitchRate)
        else
            root.AssemblyAngularVelocity = Vector3.zero
        end
        
        local dir = look.Magnitude > 0.01 and look.Unit or Vector3.zero
        local standPos = aimTargetPos - (dir * AUTO_BAT_DIST) + Vector3.new(0, AUTO_BAT_HEIGHT, 0)
        local moveDir = standPos - root.Position
        
        local hDir = Vector3.new(moveDir.X, 0, moveDir.Z)
        local hVel = hDir.Magnitude > 0.1 and hDir.Unit * AUTO_BAT_SPEED or Vector3.zero
        local vVel = math.abs(moveDir.Y) > 0.1 and Vector3.new(0, math.sign(moveDir.Y) * AUTO_BAT_VERT_SPEED, 0) or Vector3.new(0, -2, 0)
        
        root.AssemblyLinearVelocity = hVel + vVel
        
        if hDir.Magnitude > 0.5 then
            hum:Move(hDir.Unit, false)
        end
    else
        hum.AutoRotate = true
        root.AssemblyAngularVelocity = Vector3.zero
    end
    
    if autoSwingEnabled then
        local bat = char:FindFirstChild("Bat")
        if bat and bat:Is
