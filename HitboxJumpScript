local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- Config
local toggleGuiKey = Enum.KeyCode.RightControl -- Key to open/close GUI

local hitboxEnabled = false
local hitboxVisible = false
local headBoostEnabled = false
local hitboxSize = Vector3.new(4,4,4)
local boostPower = 240
local boostCooldown = 3
local lastBoost = {}
local overlays = {}

local ballProximityEnabled = false
local ballProximityDistance = 20
local ball = workspace:FindFirstChild("Ball") -- assumes the ball is named "Ball"

-- GUI setup
local playerGui = player:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "TestHitboxGUI"

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 320)
frame.Position = UDim2.new(0, 20, 1, -350)
frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
frame.BorderSizePixel = 0
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Text = "Hitbox Test Panel"
title.Size = UDim2.new(1,0,0,24)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.Parent = frame

-- Buttons
local function createButton(text,posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,200,0,30)
    btn.Position = UDim2.new(0,25,0,posY)
    btn.Text = text
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Parent = frame
    return btn
end

local toggleBtn = createButton("Hitbox: OFF",35)
local visBtn = createButton("Visible: OFF",70)
local boostBtn = createButton("Head Boost: OFF",105)
local ballBtn = createButton("Ball Proximity: OFF",140)

-- Sliders
local function createSlider(posY,labelText)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0,200,0,20)
    label.Position = UDim2.new(0,25,0,posY)
    label.BackgroundTransparency = 1
    label.Text = labelText
    label.Font = Enum.Font.SourceSans
    label.TextSize = 14
    label.TextColor3 = Color3.fromRGB(255,255,255)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local frameS = Instance.new("Frame")
    frameS.Size = UDim2.new(0,200,0,10)
    frameS.Position = UDim2.new(0,25,0,posY+20)
    frameS.BackgroundColor3 = Color3.fromRGB(100,100,100)
    frameS.Parent = frame

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,10,0,10)
    btn.Position = UDim2.new(0,0,0,0)
    btn.BackgroundColor3 = Color3.fromRGB(200,200,200)
    btn.Text = ""
    btn.Parent = frameS

    return label, frameS, btn
end

local sliderLabel, sliderFrame, sliderBtn = createSlider(175,"Size: 4")
local boostLabel, boostSliderFrame, boostSliderBtn = createSlider(215,"Boost Power: 100")
local ballLabel, ballSliderFrame, ballSliderBtn = createSlider(260,"Ball Distance: 20")

-- Draggable GUI
local dragging, dragInput, mousePos, framePos = false
local function update(input)
    local delta = input.Position - mousePos
    frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X,
        framePos.Y.Scale, framePos.Y.Offset + delta.Y)
end
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then update(input) end
end)

-- Round head hitbox creation
local function createHitbox(char)
    if char == player.Character then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    local part = overlays[char]
    if not part then
        part = Instance.new("Part")
        part.Name = "DebugHeadHitbox"
        part.Shape = Enum.PartType.Ball
        part.Size = hitboxSize
        part.Transparency = hitboxVisible and 0.5 or 1
        part.Color = Color3.fromRGB(255,0,0)
        part.CanCollide = true
        part.Massless = true
        part.Anchored = false
        part.Parent = char

        local weld = Instance.new("Weld")
        weld.Part0 = head
        weld.Part1 = part
        weld.C0 = CFrame.new(0,0,0)
        weld.Parent = part

        -- Head boost with ball detection (instant boost)
        part.Touched:Connect(function(hit)
            if headBoostEnabled and hit.Parent == player.Character and ballProximityEnabled and ball then
                local distance = (ball.Position - part.Position).Magnitude
                if distance <= ballProximityDistance then
                    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        -- Instant boost upwards
                        hrp.Velocity = Vector3.new(hrp.Velocity.X, boostPower, hrp.Velocity.Z)
                    end
                end
            end
        end)

        overlays[char] = part
    end

    part.Size = hitboxSize
    part.Transparency = hitboxVisible and 0.5 or 1
end

local function removeHitbox(char)
    if overlays[char] then
        overlays[char]:Destroy()
        overlays[char] = nil
    end
end

local function updateHitboxes()
    for _, plr in pairs(Players:GetPlayers()) do
        local char = plr.Character
        if char then
            if hitboxEnabled then createHitbox(char)
            else removeHitbox(char) end
        end
    end
end

-- Update hitboxes every frame
RunService.RenderStepped:Connect(function()
    for _, part in pairs(overlays) do
        part.Size = hitboxSize
    end
end)

-- GUI Buttons
toggleBtn.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    toggleBtn.Text = hitboxEnabled and "Hitbox: ON" or "Hitbox: OFF"
    updateHitboxes()
end)

visBtn.MouseButton1Click:Connect(function()
    hitboxVisible = not hitboxVisible
    visBtn.Text = hitboxVisible and "Visible: ON" or "Visible: OFF"
    for _, part in pairs(overlays) do part.Transparency = hitboxVisible and 0.5 or 1 end
end)

boostBtn.MouseButton1Click:Connect(function()
    headBoostEnabled = not headBoostEnabled
    boostBtn.Text = headBoostEnabled and "Head Boost: ON" or "Head Boost: OFF"
end)

ballBtn.MouseButton1Click:Connect(function()
    ballProximityEnabled = not ballProximityEnabled
    ballBtn.Text = ballProximityEnabled and "Ball Proximity: ON" or "Ball Proximity: OFF"
end)

-- Slider logic
local sliding, boosting, ballSliding = false,false,false
sliderBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding=true end end)
sliderBtn.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then sliding=false end end)
boostSliderBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then boosting=true end end)
boostSliderBtn.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then boosting=false end end)
ballSliderBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then ballSliding=true end end)
ballSliderBtn.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then ballSliding=false end end)

UserInputService.InputChanged:Connect(function(input)
    if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then
        local relativeX = math.clamp(input.Position.X - sliderFrame.AbsolutePosition.X,0,sliderFrame.AbsoluteSize.X)
        sliderBtn.Position = UDim2.new(0,relativeX-5,0,0)
        hitboxSize = Vector3.new(relativeX/5+2,relativeX/5+2,relativeX/5+2)
        sliderLabel.Text = "Size: "..math.floor(hitboxSize.X)
    end
    if boosting and input.UserInputType == Enum.UserInputType.MouseMovement then
        local relativeX = math.clamp(input.Position.X - boostSliderFrame.AbsolutePosition.X,0,boostSliderFrame.AbsoluteSize.X)
        boostSliderBtn.Position = UDim2.new(0,relativeX-5,0,0)
        boostPower = math.floor(relativeX/2+50)
        boostLabel.Text = "Boost Power: "..boostPower
    end
    if ballSliding and input.UserInputType == Enum.UserInputType.MouseMovement then
        local relativeX = math.clamp(input.Position.X - ballSliderFrame.AbsolutePosition.X,0,ballSliderFrame.AbsoluteSize.X)
        ballSliderBtn.Position = UDim2.new(0,relativeX-5,0,0)
        ballProximityDistance = math.floor(relativeX/2+5)
        ballLabel.Text = "Ball Distance: "..ballProximityDistance
    end
end)

-- Keybind to open/close GUI
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == toggleGuiKey then
        frame.Visible = not frame.Visible
    end
end)

-- Apply to players on spawn
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        task.wait(0.1)
        if hitboxEnabled then createHitbox(char) end
    end)
end)
if player.Character then
    task.wait(0.1)
    if hitboxEnabled then createHitbox(player.Character) end
end
