# - --[[
    🔥 DOORS ULTIMATE HUB v8.2 🔥
    - Knob Death Loop: збір → смерть → респавн → знову збір
    - ESP для дверей, золота, шаф, ліжок
    - Покращений God Mode
    - Fly + Couch Spoof
    - Всі монстри літають
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- =============================================
-- НАЛАШТУВАННЯ (FLAGS)
-- =============================================
local Flags = {
    ESP = false,
    ESPDoors = false,
    ESPGold = false,
    ESPCupboards = false,
    ESPBeds = false,
    SeekHelper = false,
    Fullbright = false,
    Speed = false,
    Noclip = false,
    InfiniteJump = false,
    GodMode = false,
    Fly = false,
    CouchSpoof = false,
    AutoOpenDoors = false,
    KnobFarm = false,
    KnobDeathLoop = false,
    DeathFarm = false,
    HitboxEnabled = false,
    HitboxSize = 5,
    WalkSpeed = 16,
    JumpPower = 50,
    FlySpeed = 60,
    KnobFarmSpeed = 0.3,
    KnobDeathDelay = 3,
}

-- =============================================
-- KNOB DEATH LOOP (НОВИЙ)
-- =============================================
local KnobDeathLoopActive = false
local KnobDeathLoopRunning = false
local KnobDeathCount = 0

local function FindKnobs()
    local knobs = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and (v.Name:lower():find("knob") or v.Name:lower():find("button") or v.Name:lower():find("switch") or v.Name:lower():find("lever") or v.Name:lower():find("handle")) then
            table.insert(knobs, v)
        end
    end
    return knobs
end

local function CollectKnobs()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local knobs = FindKnobs()
    local collected = 0
    
    for _, knob in ipairs(knobs) do
        local distance = (hrp.Position - knob.Position).Magnitude
        if distance < 50 then
            -- Телепортуємося до кноба
            hrp.CFrame = knob.CFrame * CFrame.new(0, 2, 0)
            task.wait(Flags.KnobFarmSpeed)
            
            -- Клікаємо кноб
            for _, child in ipairs(knob:GetChildren()) do
                if child:IsA("ClickDetector") then
                    fireclickdetector(child)
                    collected = collected + 1
                end
            end
            
            local parent = knob.Parent
            if parent then
                for _, child in ipairs(parent:GetChildren()) do
                    if child:IsA("ClickDetector") then
                        fireclickdetector(child)
                        collected = collected + 1
                    end
                end
            end
        end
    end
    
    return collected
end

local function CleanDeath()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("death") or v.Name:lower():find("corpse") or v.Name:lower():find("body")) then
            if v ~= LocalPlayer.Character then
                v:Destroy()
            end
        end
    end
    
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Trail") then
            if v.Parent ~= LocalPlayer.Character then
                v.Enabled = false
                task.wait(0.1)
                v:Destroy()
            end
        end
    end
end

local function CleanRespawn()
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")
    for _, v in ipairs(playerGui:GetChildren()) do
        if v:IsA("ScreenGui") and (v.Name:lower():find("death") or v.Name:lower():find("respawn") or v.Name:lower():find("black") or v.Name:lower():find("fade")) then
            v:Destroy()
        end
    end
    
    for _, v in ipairs(playerGui:GetDescendants()) do
        if v:IsA("Frame") and v.BackgroundColor3 == Color3.fromRGB(0, 0, 0) then
            v.BackgroundTransparency = 1
        end
    end
end

local function PerformKnobDeath()
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    KnobDeathCount = KnobDeathCount + 1
    CleanDeath()
    humanoid.Health = 0
    task.wait(1)
    CleanRespawn()
    
    local newCharacter = LocalPlayer.Character
    local timeout = 0
    while not newCharacter and timeout < 10 do
        task.wait(0.1)
        timeout = timeout + 0.1
        newCharacter = LocalPlayer.Character
    end
    
    if newCharacter then
        task.wait(0.5)
        local newHumanoid = newCharacter:FindFirstChildOfClass("Humanoid")
        if newHumanoid then
            newHumanoid.Health = newHumanoid.MaxHealth
        end
        
        if GodModeActive then
            ApplyGodMode()
        end
        if FlyActive then
            StartFly()
        end
    end
end

local function KnobDeathLoop()
    if KnobDeathLoopRunning then return end
    KnobDeathLoopRunning = true
    
    while KnobDeathLoopActive do
        -- Збираємо кноби
        local collected = CollectKnobs()
        
        -- Якщо зібрали хоч щось, вмираємо
        if collected > 0 then
            PerformKnobDeath()
        end
        
        -- Чекаємо перед наступним циклом
        task.wait(Flags.KnobDeathDelay)
    end
    
    KnobDeathLoopRunning = false
end

-- =============================================
-- ESP SYSTEM
-- =============================================
local ESPObjects = {}

local function CreateESP(target, name, color, icon)
    if not target then return end
    
    local hrp = target:FindFirstChild("HumanoidRootPart") or target:FindFirstChild("Torso") or target:FindFirstChild("Head") or target:FindFirstChildWhichIsA("BasePart")
    if not hrp then return end
    
    local oldESP = hrp:FindFirstChild("ESP_" .. name)
    if oldESP then
        oldESP:Destroy()
    end
    
    local esp = Instance.new("BillboardGui")
    esp.Name = "ESP_" .. name
    esp.Size = UDim2.new(0, 160, 0, 50)
    esp.AlwaysOnTop = true
    esp.Adornee = hrp
    esp.Parent = hrp
    
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = color
    bg.BackgroundTransparency = 0.3
    bg.Parent = esp
    
    local bgCorner = Instance.new("UICorner")
    bgCorner.CornerRadius = UDim.new(0, 6)
    bgCorner.Parent = bg
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 2)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = icon .. " " .. name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 13
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = esp
    
    local distLabel = Instance.new("TextLabel")
    distLabel.Size = UDim2.new(1, 0, 0.5, 0)
    distLabel.Position = UDim2.new(0, 0, 0.5, 0)
    distLabel.BackgroundTransparency = 1
    distLabel.Text = "0m"
    distLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    distLabel.TextSize = 11
    distLabel.Font = Enum.Font.Gotham
    distLabel.Parent = esp
    
    table.insert(ESPObjects, {
        Target = target,
        HRP = hrp,
        NameLabel = nameLabel,
        DistLabel = distLabel,
        Type = name
    })
end

local function UpdateESP()
    for _, espObj in ipairs(ESPObjects) do
        if espObj.Target and espObj.Target.Parent and espObj.HRP and espObj.HRP.Parent then
            local character = LocalPlayer.Character
            if character then
                local playerHrp = character:FindFirstChild("HumanoidRootPart")
                if playerHrp then
                    local distance = (playerHrp.Position - espObj.HRP.Position).Magnitude
                    espObj.DistLabel.Text = math.floor(distance) .. "m"
                    
                    if distance < 10 then
                        espObj.DistLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                    elseif distance < 30 then
                        espObj.DistLabel.TextColor3 = Color3.fromRGB(255, 165, 0)
                    else
                        espObj.DistLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
                    end
                end
            end
        end
    end
end

local function ClearESP()
    for _, espObj in ipairs(ESPObjects) do
        if espObj.HRP and espObj.HRP.Parent then
            local esp = espObj.HRP:FindFirstChild("ESP_" .. espObj.Type)
            if esp then
                esp:Destroy()
            end
        end
    end
    ESPObjects = {}
end

-- =============================================
-- ESP ДЛЯ ДВЕРЕЙ
-- =============================================
local function FindDoors()
    local doors = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("door") or v.Name:lower():find("gate") or v.Name:lower():find("entrance") or v.Name:lower():find("exit")) then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Door") or v:FindFirstChild("MainDoor") or v:FindFirstChildWhichIsA("BasePart")
            if hrp then
                table.insert(doors, v)
            end
        end
    end
    return doors
end

local function ApplyESPDoors()
    local doors = FindDoors()
    for _, door in ipairs(doors) do
        CreateESP(door, "Двері", Color3.fromRGB(0, 150, 255), "🚪")
    end
end

-- =============================================
-- ESP ДЛЯ ЗОЛОТА
-- =============================================
local function FindGold()
    local gold = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and (v.Name:lower():find("gold") or v.Name:lower():find("coin") or v.Name:lower():find("money") or v.Name:lower():find("treasure") or v.Name:lower():find("loot")) then
            table.insert(gold, v)
        end
    end
    return gold
end

local function ApplyESPGold()
    local gold = FindGold()
    for _, g in ipairs(gold) do
        local model = Instance.new("Model")
        model.Name = "Gold_" .. g.Name
        g.Parent = model
        model.Parent = workspace
        CreateESP(model, "Золото", Color3.fromRGB(255, 215, 0), "✨")
    end
end

-- =============================================
-- ESP ДЛЯ ШАФ
-- =============================================
local function FindCupboards()
    local cupboards = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("cupboard") or v.Name:lower():find("closet") or v.Name:lower():find("cabinet") or v.Name:lower():find("wardrobe") or v.Name:lower():find("shelf") or v.Name:lower():find("chest") or v.Name:lower():find("drawer") or v.Name:lower():find("locker")) then
            local hrp = v:FindFirstChildWhichIsA("BasePart")
            if hrp then
                table.insert(cupboards, v)
            end
        end
    end
    return cupboards
end

local function ApplyESPCupboards()
    local cupboards = FindCupboards()
    for _, cupboard in ipairs(cupboards) do
        CreateESP(cupboard, "Шафа", Color3.fromRGB(139, 69, 19), "🗄️")
    end
end

-- =============================================
-- ESP ДЛЯ ЛІЖОК
-- =============================================
local function FindBeds()
    local beds = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("bed") or v.Name:lower():find("bunk") or v.Name:lower():find("mattress") or v.Name:lower():find("couch") or v.Name:lower():find("sofa") or v.Name:lower():find("chair") or v.Name:lower():find("seat")) then
            local hrp = v:FindFirstChildWhichIsA("BasePart")
            if hrp then
                table.insert(beds, v)
            end
        end
    end
    return beds
end

local function ApplyESPBeds()
    local beds = FindBeds()
    for _, bed in ipairs(beds) do
        CreateESP(bed, "Ліжко", Color3.fromRGB(255, 100, 100), "🛏️")
    end
end

-- =============================================
-- ПОКРАЩЕНИЙ GOD MODE
-- =============================================
local GodModeActive = false
local VisualCharacter = nil

local function CreateVisualCharacter()
    if VisualCharacter and VisualCharacter.Parent then
        VisualCharacter:Destroy()
    end
    
    local character = LocalPlayer.Character
    if not character then return end
    
    VisualCharacter = character:Clone()
    VisualCharacter.Name = "Visual_" .. character.Name
    VisualCharacter.Parent = workspace
    
    local visualHumanoid = VisualCharacter:FindFirstChildOfClass("Humanoid")
    if visualHumanoid then
        visualHumanoid:Destroy()
    end
    
    for _, part in ipairs(VisualCharacter:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
            part.Anchored = true
        end
    end
    
    return VisualCharacter
end

local function ApplyGodMode()
    local character = LocalPlayer.Character
    if not character then return end
    
    CreateVisualCharacter()
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(hrp.Position.X, -500, hrp.Position.Z)
    end
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 1
            part.CanCollide = false
            part.Anchored = true
        end
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.MaxHealth = math.huge
        humanoid.Health = math.huge
        humanoid.BreakJointsOnDeath = false
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
    end
end

local function DisableGodMode()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(hrp.Position.X, 10, hrp.Position.Z)
    end
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 0
            part.CanCollide = true
            part.Anchored = false
        end
    end
    
    if VisualCharacter and VisualCharacter.Parent then
        VisualCharacter:Destroy()
        VisualCharacter = nil
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.MaxHealth = 100
        humanoid.Health = 100
        humanoid.BreakJointsOnDeath = true
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
    end
end

LocalPlayer.CharacterAdded:Connect(function(character)
    task.wait(0.1)
    if GodModeActive then
        ApplyGodMode()
    end
end)

RunService.RenderStepped:Connect(function()
    if not GodModeActive then return end
    
    local character = LocalPlayer.Character
    if not character then return end
    
    if VisualCharacter and VisualCharacter.Parent then
        local realHrp = character:FindFirstChild("HumanoidRootPart")
        local visualHrp = VisualCharacter:FindFirstChild("HumanoidRootPart")
        if realHrp and visualHrp then
            visualHrp.CFrame = realHrp.CFrame
        end
    end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if hrp then
        if hrp.Position.Y > -400 then
            hrp.CFrame = CFrame.new(hrp.Position.X, -500, hrp.Position.Z)
        end
    end
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.Transparency = 1
            part.CanCollide = false
            part.Anchored = true
        end
    end
end)

-- =============================================
-- COUCH SPOOF
-- =============================================
local CouchSpoofActive = false
local CouchPart = nil

local function ApplyCouchSpoof()
    if CouchPart then return end
    
    CouchPart = Instance.new("Part")
    CouchPart.Name = "CouchSpoof"
    CouchPart.Size = Vector3.new(10, 1, 10)
    CouchPart.Transparency = 1
    CouchPart.CanCollide = true
    CouchPart.Anchored = true
    CouchPart.Parent = workspace
    
    RunService.RenderStepped:Connect(function()
        if CouchSpoofActive and CouchPart and CouchPart.Parent then
            local character = LocalPlayer.Character
            if character then
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    CouchPart.CFrame = hrp.CFrame * CFrame.new(0, -3, 0)
                end
            end
        end
    end)
end

local function RemoveCouchSpoof()
    if CouchPart and CouchPart.Parent then
        CouchPart:Destroy()
        CouchPart = nil
    end
end

-- =============================================
-- FLY SYSTEM
-- =============================================
local FlyActive = false
local FlySpeed = 60
local FlyConnection = nil

local function StartFly()
    if FlyConnection then
        FlyConnection:Disconnect()
        FlyConnection = nil
    end
    
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = 0
        humanoid.JumpPower = 0
    end
    
    FlyConnection = RunService.RenderStepped:Connect(function()
        if not FlyActive then return end
        
        local char = LocalPlayer.Character
        if not char then return end
        
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local moveDirection = Vector3.new()
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - Camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + Camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
            root.CFrame = root.CFrame + moveDirection * FlySpeed * 0.1
        end
    end)
end

local function StopFly()
    if FlyConnection then
        FlyConnection:Disconnect()
        FlyConnection = nil
    end
    
    local character = LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = Flags.WalkSpeed
            humanoid.JumpPower = Flags.JumpPower
        end
    end
end

-- =============================================
-- ВСІ МОНСТРИ ЛІТАЮТЬ
-- =============================================
local MonstersFlyActive = false
local MonsterConnections = {}

local function MakeMonstersFly()
    for _, conn in ipairs(MonsterConnections) do
        conn:Disconnect()
    end
    MonsterConnections = {}
    
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("seek") or v.Name:lower():find("monster") or v.Name:lower():find("entity") or v.Name:lower():find("figure") or v.Name:lower():find("screech") or v.Name:lower():find("ambush") or v.Name:lower():find("halt") or v.Name:lower():find("rush")) then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Torso") or v:FindFirstChild("Head")
            if hrp then
                hrp.Anchored = true
                hrp.CanCollide = false
                
                for _, part in ipairs(v:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Anchored = true
                        part.CanCollide = false
                    end
                end
                
                local conn = RunService.RenderStepped:Connect(function()
                    if MonstersFlyActive and v.Parent then
                        hrp.Anchored = true
                        hrp.CanCollide = false
                    end
                end)
                table.insert(MonsterConnections, conn)
            end
        end
    end
end

local function ResetMonsters()
    for _, conn in ipairs(MonsterConnections) do
        conn:Disconnect()
    end
    MonsterConnections = {}
    
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("seek") or v.Name:lower():find("monster") or v.Name:lower():find("entity") or v.Name:lower():find("figure") or v.Name:lower():find("screech") or v.Name:lower():find("ambush") or v.Name:lower():find("halt") or v.Name:lower():find("rush")) then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Torso") or v:FindFirstChild("Head")
            if hrp then
                hrp.Anchored = false
                hrp.CanCollide = true
                
                for _, part in ipairs(v:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Anchored = false
                        part.CanCollide = true
                    end
                end
            end
        end
    end
end

-- =============================================
-- DEATH FARM SYSTEM
-- =============================================
local DeathFarmActive = false
local DeathFarmDelay = 5
local DeathFarmCount = 0
local IsRespawning = false

local function PerformDeath()
    if IsRespawning then return end
    IsRespawning = true
    
    local character = LocalPlayer.Character
    if not character then
        IsRespawning = false
        return
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        IsRespawning = false
        return
    end
    
    DeathFarmCount = DeathFarmCount + 1
    CleanDeath()
    humanoid.Health = 0
    task.wait(1)
    CleanRespawn()
    
    local newCharacter = LocalPlayer.Character
    local timeout = 0
    while not newCharacter and timeout < 10 do
        task.wait(0.1)
        timeout = timeout + 0.1
        newCharacter = LocalPlayer.Character
    end
    
    if newCharacter then
        task.wait(0.5)
        local newHumanoid = newCharacter:FindFirstChildOfClass("Humanoid")
        if newHumanoid then
            newHumanoid.Health = newHumanoid.MaxHealth
        end
        
        if GodModeActive then
            ApplyGodMode()
        end
        if FlyActive then
            StartFly()
        end
    end
    
    IsRespawning = false
end

-- =============================================
-- HITBOX SYSTEM
-- =============================================
local HitboxActive = false
local HitboxSize = 5
local HitboxParts = {}

local function ApplyHitbox(monster)
    if not monster then return end
    
    local hrp = monster:FindFirstChild("HumanoidRootPart") or monster:FindFirstChild("Torso") or monster:FindFirstChild("Head")
    if not hrp then return end
    
    local hitbox = Instance.new("Part")
    hitbox.Name = "Hitbox"
    hitbox.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
    hitbox.Transparency = 0.5
    hitbox.CanCollide = false
    hitbox.Anchored = true
    hitbox.Color = Color3.fromRGB(255, 0, 0)
    hitbox.Material = Enum.Material.ForceField
    hitbox.CFrame = hrp.CFrame
    hitbox.Parent = monster
    
    local highlight = Instance.new("SelectionBox")
    highlight.Adornee = hitbox
    highlight.Color3 = Color3.fromRGB(255, 0, 0)
    highlight.LineThickness = 0.1
    highlight.Transparency = 0.5
    highlight.Parent = hitbox
    
    table.insert(HitboxParts, hitbox)
    
    RunService.RenderStepped:Connect(function()
        if HitboxActive and hitbox.Parent then
            local monsterHrp = monster:FindFirstChild("HumanoidRootPart") or monster:FindFirstChild("Torso") or monster:FindFirstChild("Head")
            if monsterHrp then
                hitbox.CFrame = monsterHrp.CFrame
            end
        end
    end)
    
    return hitbox
end

local function RemoveHitboxes()
    for _, part in ipairs(HitboxParts) do
        if part and part.Parent then
            part:Destroy()
        end
    end
    HitboxParts = {}
end

local function FindMonsters()
    local monsters = {}
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("seek") or v.Name:lower():find("monster") or v.Name:lower():find("entity") or v.Name:lower():find("figure") or v.Name:lower():find("screech") or v.Name:lower():find("ambush") or v.Name:lower():find("halt") or v.Name:lower():find("rush")) then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Torso") or v:FindFirstChild("Head")
            if hrp then
                table.insert(monsters, v)
            end
        end
    end
    return monsters
end

-- =============================================
-- SEEK HELPER
-- =============================================
local SeekHelperActive = false
local SeekWarningActive = false

local function FindSeek()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Model") and (v.Name:lower():find("seek") or v.Name:lower():find("eyes")) then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Torso") or v:FindFirstChild("Head")
            if hrp then
                return v, hrp
            end
        end
    end
    return nil, nil
end

local function CreateSeekWarning()
    if SeekWarningActive then return end
    SeekWarningActive = true
    
    local Warning = Instance.new("Frame")
    Warning.Name = "SeekWarning"
    Warning.Size = UDim2.new(0, 300, 0, 60)
    Warning.Position = UDim2.new(0.5, -150, 0, -70)
    Warning.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
    Warning.BackgroundTransparency = 1
    Warning.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local WarningCorner = Instance.new("UICorner")
    WarningCorner.CornerRadius = UDim.new(0, 12)
    WarningCorner.Parent = Warning

    local WarningStroke = Instance.new("UIStroke")
    WarningStroke.Color = Color3.fromRGB(255, 0, 0)
    WarningStroke.Thickness = 3
    WarningStroke.Transparency = 0.5
    WarningStroke.Parent = Warning

    local WarningText = Instance.new("TextLabel")
    WarningText.Size = UDim2.new(1, 0, 1, 0)
    WarningText.BackgroundTransparency = 1
    WarningText.Text = "⚠️⚠️⚠️ SEEK НАБЛИЖАЄТЬСЯ! ⚠️⚠️⚠️"
    WarningText.TextColor3 = Color3.fromRGB(255, 0, 0)
    WarningText.TextSize = 18
    WarningText.Font = Enum.Font.GothamBlack
    WarningText.Parent = Warning

    TweenService:Create(Warning, TweenInfo.new(0.3), {BackgroundTransparency = 0.1, Position = UDim2.new(0.5, -150, 0, 10)}):Play()
    
    task.spawn(function()
        while SeekHelperActive and SeekWarningActive do
            TweenService:Create(WarningText, TweenInfo.new(0.3), {TextTransparency = 0.3}):Play()
            task.wait(0.3)
            TweenService:Create(WarningText, TweenInfo.new(0.3), {TextTransparency = 0}):Play()
            task.wait(0.3)
        end
    end)
    
    task.wait(3)
    TweenService:Create(Warning, TweenInfo.new(0.3), {BackgroundTransparency = 1, Position = UDim2.new(0.5, -150, 0, -70)}):Play()
    task.wait(0.3)
    Warning:Destroy()
    SeekWarningActive = false
end

-- =============================================
-- СТВОРЕННЯ GUI
-- =============================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DoorsUltimateHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Головне вікно
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 600, 0, 500)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

-- Преміум ефекти
local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 15)
Corner.Parent = MainFrame

local Stroke = Instance.new("UIStroke")
Stroke.Color = Color3.fromRGB(255, 50, 50)
Stroke.Thickness = 2
Stroke.Transparency = 0.3
Stroke.Parent = MainFrame

local Gradient = Instance.new("UIGradient")
Gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 10, 10)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 10, 20))
})
Gradient.Rotation = 45
Gradient.Parent = MainFrame

-- Заголовок
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 50)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 5, 5)
TitleBar.BackgroundTransparency = 0.5
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 15)
TitleCorner.Parent = TitleBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -50, 1, 0)
Title.BackgroundTransparency = 1
Title.Text = "🚪 DOORS ULTIMATE HUB v8.2"
Title.TextColor3 = Color3.fromRGB(255, 80, 80)
Title.TextSize = 20
Title.Font = Enum.Font.GothamBlack
Title.Parent = TitleBar

-- Кнопка закриття
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -45, 0, 7)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 70, 70)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 16
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 8)
CloseCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ClearESP()
    RemoveHitboxes()
    ResetMonsters()
    RemoveCouchSpoof()
    StopFly()
    ScreenGui:Destroy()
end)

-- Кнопка згортання
local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 35, 0, 35)
MinBtn.Position = UDim2.new(1, -88, 0, 7)
MinBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
MinBtn.Text = "—"
MinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinBtn.TextSize = 16
MinBtn.Font = Enum.Font.GothamBold
MinBtn.Parent = TitleBar

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 8)
MinCorner.Parent = MinBtn

local minimized = false
MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    local targetSize = minimized and UDim2.new(0, 600, 0, 50) or UDim2.new(0, 600, 0, 500)
    TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = targetSize}):Play()
end)

-- Перетягування
local dragging, dragInput, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- =============================================
-- ВКЛАДКИ (TABS)
-- =============================================
local TabBar = Instance.new("Frame")
TabBar.Size = UDim2.new(1, 0, 0, 55)
TabBar.Position = UDim2.new(0, 0, 0, 50)
TabBar.BackgroundTransparency = 1
TabBar.Parent = MainFrame

local Tabs = {}
local CurrentTab = nil

local function CreateTab(name, icon)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(0, 110, 0, 40)
    TabBtn.Position = UDim2.new(0, 10 + (#Tabs * 115), 0, 7)
    TabBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
    TabBtn.Text = icon .. " " .. name
    TabBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    TabBtn.TextSize = 14
    TabBtn.Font = Enum.Font.GothamBold
    TabBtn.Parent = TabBar

    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 8)
    TabCorner.Parent = TabBtn

    local TabStroke = Instance.new("UIStroke")
    TabStroke.Color = Color3.fromRGB(255, 50, 50)
    TabStroke.Thickness = 1
    TabStroke.Transparency = 0.8
    TabStroke.Parent = TabBtn

    local TabContent = Instance.new("ScrollingFrame")
    TabContent.Name = name .. "Content"
    TabContent.Size = UDim2.new(1, -20, 1, -120)
    TabContent.Position = UDim2.new(0, 10, 0, 110)
    TabContent.BackgroundTransparency = 1
    TabContent.ScrollBarThickness = 6
    TabContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
    TabContent.Visible = false
    TabContent.Parent = MainFrame

    local Layout = Instance.new("UIListLayout")
    Layout.Padding = UDim.new(0, 8)
    Layout.Parent = TabContent

    local Padding = Instance.new("UIPadding")
    Padding.PaddingTop = UDim.new(0, 5)
    Padding.PaddingBottom = UDim.new(0, 5)
    Padding.Parent = TabContent

    TabBtn.MouseButton1Click:Connect(function()
        if CurrentTab then
            CurrentTab.Button.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
            CurrentTab.Content.Visible = false
        end
        TabBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        TabContent.Visible = true
        CurrentTab = {Button = TabBtn, Content = TabContent}
    end)

    local tab = {Button = TabBtn, Content = TabContent}
    table.insert(Tabs, tab)
    return tab
end

-- =============================================
-- UI ЕЛЕМЕНТИ
-- =============================================
local function CreateSection(parent, text)
    local Section = Instance.new("Frame")
    Section.Size = UDim2.new(1, -10, 0, 35)
    Section.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    Section.BackgroundTransparency = 0.8
    Section.Parent = parent

    local SectionCorner = Instance.new("UICorner")
    SectionCorner.CornerRadius = UDim.new(0, 8)
    SectionCorner.Parent = Section

    local SectionLabel = Instance.new("TextLabel")
    SectionLabel.Size = UDim2.new(1, 0, 1, 0)
    SectionLabel.BackgroundTransparency = 1
    SectionLabel.Text = text
    SectionLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    SectionLabel.TextSize = 15
    SectionLabel.Font = Enum.Font.GothamBold
    SectionLabel.Parent = Section

    return Section
end

local function CreateButton(parent, text, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -10, 0, 40)
    Btn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    Btn.Text = text
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.TextSize = 14
    Btn.Font = Enum.Font.Gotham
    Btn.Parent = parent

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 10)
    BtnCorner.Parent = Btn

    local BtnStroke = Instance.new("UIStroke")
    BtnStroke.Color = Color3.fromRGB(255, 50, 50)
    BtnStroke.Thickness = 1
    BtnStroke.Transparency = 0.6
    BtnStroke.Parent = Btn

    Btn.MouseEnter:Connect(function()
        TweenService:Create(Btn, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(60, 60, 80)}):Play()
    end)
    Btn.MouseLeave:Connect(function()
        TweenService:Create(Btn, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(40, 40, 55)}):Play()
    end)

    Btn.MouseButton1Click:Connect(function()
        callback()
    end)

    return Btn
end

local function CreateToggle(parent, text, default, callback)
    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(1, -10, 0, 40)
    Toggle.BackgroundColor3 = default and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(40, 40, 55)
    Toggle.Text = text .. (default and " ✅" or " ❌")
    Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    Toggle.TextSize = 14
    Toggle.Font = Enum.Font.Gotham
    Toggle.Parent = parent

    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0, 10)
    ToggleCorner.Parent = Toggle

    local ToggleStroke = Instance.new("UIStroke")
    ToggleStroke.Color = Color3.fromRGB(255, 50, 50)
    ToggleStroke.Thickness = 1
    ToggleStroke.Transparency = 0.6
    ToggleStroke.Parent = Toggle

    local state = default
    Toggle.MouseButton1Click:Connect(function()
        state = not state
        Toggle.BackgroundColor3 = state and Color3.fromRGB(255, 50, 50) or Color3.fromRGB(40, 40, 55)
        Toggle.Text = text .. (state and " ✅" or " ❌")
        callback(state)
    end)

    return Toggle
end

-- СЛАЙДЕР
local function CreateSlider(parent, text, min, max, default, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(1, -10, 0, 50)
    SliderFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    SliderFrame.Parent = parent

    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0, 10)
    SliderCorner.Parent = SliderFrame

    local SliderLabel = Instance.new("TextLabel")
    SliderLabel.Size = UDim2.new(1, -10, 0, 20)
    SliderLabel.Position = UDim2.new(0, 5, 0, 2)
    SliderLabel.BackgroundTransparency = 1
    SliderLabel.Text = text .. ": " .. tostring(default)
    SliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    SliderLabel.TextSize = 13
    SliderLabel.Font = Enum.Font.Gotham
    SliderLabel.Parent = SliderFrame

    local SliderBar = Instance.new("Frame")
    SliderBar.Size = UDim2.new(1, -20, 0, 10)
    SliderBar.Position = UDim2.new(0, 10, 0, 30)
    SliderBar.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    SliderBar.Parent = SliderFrame

    local SliderBarCorner = Instance.new("UICorner")
    SliderBarCorner.CornerRadius = UDim.new(0, 5)
    SliderBarCorner.Parent = SliderBar

    local SliderFill = Instance.new("Frame")
    SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    SliderFill.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    SliderFill.Parent = SliderBar

    local SliderFillCorner = Instance.new("UICorner")
    SliderFillCorner.CornerRadius = UDim.new(0, 5)
    SliderFillCorner.Parent = SliderFill

    local SliderBtn = Instance.new("TextButton")
    SliderBtn.Size = UDim2.new(0, 28, 0, 28)
    SliderBtn.Position = UDim2.new((default - min) / (max - min), -14, 0, -9)
    SliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderBtn.Text = ""
    SliderBtn.Parent = SliderBar

    local SliderBtnCorner = Instance.new("UICorner")
    SliderBtnCorner.CornerRadius = UDim.new(1, 0)
    SliderBtnCorner.Parent = SliderBtn

    local SliderBtnStroke = Instance.new("UIStroke")
    SliderBtnStroke.Color = Color3.fromRGB(255, 50, 50)
    SliderBtnStroke.Thickness = 2
    SliderBtnStroke.Parent = SliderBtn

    local dragging = false
    
    SliderBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            local relativeX = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
            local value = min + (max - min) * relativeX
            value = math.floor(value * 100) / 100
            SliderFill.Size = UDim2.new(relativeX, 0, 1, 0)
            SliderBtn.Position = UDim2.new(relativeX, -14, 0, -9)
            SliderLabel.Text = text .. ": " .. tostring(value)
            callback(value)
        end
    end)
    
    SliderBtn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    SliderBtn.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local relativeX = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
            local value = min + (max - min) * relativeX
            value = math.floor(value * 100) / 100
            SliderFill.Size = UDim2.new(relativeX, 0, 1, 0)
            SliderBtn.Position = UDim2.new(relativeX, -14, 0, -9)
            SliderLabel.Text = text .. ": " .. tostring(value)
            callback(value)
        end
    end)

    SliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            local relativeX = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
            local value = min + (max - min) * relativeX
            value = math.floor(value * 100) / 100
            SliderFill.Size = UDim2.new(relativeX, 0, 1, 0)
            SliderBtn.Position = UDim2.new(relativeX, -14, 0, -9)
            SliderLabel.Text = text .. ": " .. tostring(value)
            callback(value)
        end
    end)

    return SliderFrame
end

local function CreateLabel(parent, text)
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -10, 0, 35)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.fromRGB(255, 50, 50)
    Label.TextSize = 18
    Label.Font = Enum.Font.GothamBlack
    Label.Parent = parent
    return Label
end

-- =============================================
-- СТВОРЕННЯ ВКЛАДОК
-- =============================================

-- ГОЛОВНА ВКЛАДКА
local MainTab = CreateTab("Головна", "🏠")
CreateLabel(MainTab.Content, "🏠 ГОЛОВНІ НАЛАШТУВАННЯ")
CreateSection(MainTab.Content, "Захист")
CreateToggle(MainTab.Content, "GOD MODE (Провал під карту)", false, function(state)
    GodModeActive = state
    if state then
        ApplyGodMode()
    else
        DisableGodMode()
    end
end)
CreateToggle(MainTab.Content, "Couch Spoof", false, function(state)
    CouchSpoofActive = state
    if state then
        ApplyCouchSpoof()
    else
        RemoveCouchSpoof()
    end
end)
CreateToggle(MainTab.Content, "Всі монстри літають", false, function(state)
    MonstersFlyActive = state
    if state then
        MakeMonstersFly()
    else
        ResetMonsters()
    end
end)
CreateToggle(MainTab.Content, "Fly (швидкість 60)", false, function(state)
    FlyActive = state
    if state then
        StartFly()
    else
        StopFly()
    end
end)
CreateToggle(MainTab.Content, "Швидкість", false, function(state)
    Flags.Speed = state
    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = state and Flags.WalkSpeed or 16
    end
end)
CreateToggle(MainTab.Content, "Noclip", false, function(state)
    Flags.Noclip = state
end)
CreateToggle(MainTab.Content, "Нескінченний стрибок", false, function(state)
    Flags.InfiniteJump = state
end)
CreateSlider(MainTab.Content, "Швидкість ходьби", 16, 500, 16, function(value)
    Flags.WalkSpeed = value
    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid and Flags.Speed then
        humanoid.WalkSpeed = value
    end
end)
CreateSlider(MainTab.Content, "Сила стрибка", 50, 500, 50, function(value)
    Flags.JumpPower = value
    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.JumpPower = value
    end
end)

-- ВКЛАДКА KNOB DEATH LOOP (НОВА)
local KnobDeathTab = CreateTab("Knob Death", "💀🔘")
CreateLabel(KnobDeathTab.Content, "💀🔘 KNOB DEATH LOOP")
CreateSection(KnobDeathTab.Content, "Збір → Смерть → Респавн")
CreateToggle(KnobDeathTab.Content, "Knob Death Loop (Увімкнути)", false, function(state)
    KnobDeathLoopActive = state
    if state then
        task.spawn(KnobDeathLoop)
    end
end)
CreateSlider(KnobDeathTab.Content, "Швидкість збору", 0.1, 1, 0.3, function(value)
    Flags.KnobFarmSpeed = value
end)
CreateSlider(KnobDeathTab.Content, "Затримка між циклами", 1, 10, 3, function(value)
    Flags.KnobDeathDelay = value
end)
CreateButton(KnobDeathTab.Content, "Зібрати кноби зараз", function()
    local collected = CollectKnobs()
    print("🔘 Зібрано кнобів: " .. collected)
end)
CreateButton(KnobDeathTab.Content, "Померти зараз", function()
    PerformKnobDeath()
end)
CreateButton(KnobDeathTab.Content, "Скинути лічильник", function()
    KnobDeathCount = 0
end)

-- ВКЛАДКА ESP
local ESPTab = CreateTab("ESP", "👁️")
CreateLabel(ESPTab.Content, "👁️ ESP СИСТЕМА")
CreateSection(ESPTab.Content, "ESP для об'єктів")
CreateToggle(ESPTab.Content, "ESP Монстри", false, function(state)
    Flags.ESP = state
    if not state then
        ClearESP()
    end
end)
CreateToggle(ESPTab.Content, "ESP Двері 🚪", false, function(state)
    Flags.ESPDoors = state
    if state then
        ApplyESPDoors()
    else
        ClearESP()
        if Flags.ESPGold then ApplyESPGold() end
        if Flags.ESPCupboards then ApplyESPCupboards() end
        if Flags.ESPBeds then ApplyESPBeds() end
    end
end)
CreateToggle(ESPTab.Content, "ESP Золото ✨", false, function(state)
    Flags.ESPGold = state
    if state then
        ApplyESPGold()
    else
        ClearESP()
        if Flags.ESPDoors then ApplyESPDoors() end
        if Flags.ESPCupboards then ApplyESPCupboards() end
        if Flags.ESPBeds then ApplyESPBeds() end
    end
end)
CreateToggle(ESPTab.Content, "ESP Шафи 🗄️", false, function(state)
    Flags.ESPCupboards = state
    if state then
        ApplyESPCupboards()
    else
        ClearESP()
        if Flags.ESPDoors then ApplyESPDoors() end
        if Flags.ESPGold then ApplyESPGold() end
        if Flags.ESPBeds then ApplyESPBeds() end
    end
end)
CreateToggle(ESPTab.Content, "ESP Ліжка 🛏️", false, function(state)
    Flags.ESPBeds = state
    if state then
        ApplyESPBeds()
    else
        ClearESP()
        if Flags.ESPDoors then ApplyESPDoors() end
        if Flags.ESPGold then ApplyESPGold() end
        if Flags.ESPCupboards then ApplyESPCupboards() end
    end
end)
CreateButton(ESPTab.Content, "Очистити весь ESP", function()
    ClearESP()
    Flags.ESP = false
    Flags.ESPDoors = false
    Flags.ESPGold = false
    Flags.ESPCupboards = false
    Flags.ESPBeds = false
end)

-- ВКЛАДКА DEATH FARM
local DeathTab = CreateTab("Смерть", "💀")
CreateLabel(DeathTab.Content, "💀 DEATH FARM")
CreateSection(DeathTab.Content, "Автоматична смерть")
CreateToggle(DeathTab.Content, "Death Farm (Увімкнути)", false, function(state)
    DeathFarmActive = state
end)
CreateSlider(DeathTab.Content, "Затримка між смертями", 1, 30, 5, function(value)
    DeathFarmDelay = value
end)
CreateButton(DeathTab.Content, "Померти зараз", function()
    PerformDeath()
end)
CreateButton(DeathTab.Content, "Скинути лічильник", function()
    DeathFarmCount = 0
end)

-- ВКЛАДКА KNOB FARM (старий)
local KnobTab = CreateTab("Knob", "🔘")
CreateLabel(KnobTab.Content, "🔘 KNOB FARM")
CreateSection(KnobTab.Content, "Автоматичний фарм")
CreateToggle(KnobTab.Content, "Knob Farm (Увімкнути)", false, function(state)
    KnobFarmActive = state
end)
CreateSlider(KnobTab.Content, "Швидкість фарму", 0.1, 2, 0.5, function(value)
    KnobFarmSpeed = value
end)
CreateButton(KnobTab.Content, "Знайти всі кнопки", function()
    local knobs = FindKnobs()
    local count = #knobs
    print("🔘 Знайдено кнопок: " .. count)
end)
CreateButton(KnobTab.Content, "Телепорт до найближчої кнопки", function()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local knobs = FindKnobs()
    local nearestKnob = nil
    local nearestDistance = math.huge
    
    for _, knob in ipairs(knobs) do
        local distance = (hrp.Position - knob.Position).Magnitude
        if distance < nearestDistance then
            nearestDistance = distance
            nearestKnob = knob
        end
    end
    
    if nearestKnob then
        hrp.CFrame = nearestKnob.CFrame * CFrame.new(0, 2, 0)
    end
end)

-- ВКЛАДКА HITBOX
local HitboxTab = CreateTab("Hitbox", "🎯")
CreateLabel(HitboxTab.Content, "🎯 HITBOX SYSTEM")
CreateSection(HitboxTab.Content, "Хітбокси для монстрів")
CreateToggle(HitboxTab.Content, "Увімкнути Hitbox", false, function(state)
    HitboxActive = state
    if state then
        local monsters = FindMonsters()
        for _, monster in ipairs(monsters) do
            ApplyHitbox(monster)
        end
    else
        RemoveHitboxes()
    end
end)
CreateSlider(HitboxTab.Content, "Розмір хітбокса", 1, 20, 5, function(value)
    HitboxSize = value
    if HitboxActive then
        RemoveHitboxes()
        local monsters = FindMonsters()
        for _, monster in ipairs(monsters) do
            ApplyHitbox(monster)
        end
    end
end)
CreateButton(HitboxTab.Content, "Оновити хітбокси", function()
    RemoveHitboxes()
    local monsters = FindMonsters()
    for _, monster in ipairs(monsters) do
        ApplyHitbox(monster)
    end
end)
CreateButton(HitboxTab.Content, "Видалити всі хітбокси", function()
    RemoveHitboxes()
end)

-- ВКЛАДКА SEEK HELPER
local SeekTab = CreateTab("Seek", "👁️")
CreateLabel(SeekTab.Content, "👁️ SEEK HELPER")
CreateSection(SeekTab.Content, "Seek Helper")
CreateToggle(SeekTab.Content, "Seek Helper (Увімкнути)", false, function(state)
    SeekHelperActive = state
end)
CreateButton(SeekTab.Content, "Знайти Seek зараз", function()
    local seek, hrp = FindSeek()
    if seek and hrp then
        local character = LocalPlayer.Character
        if character then
            local playerHrp = character:FindFirstChild("HumanoidRootPart")
            if playerHrp then
                local distance = (playerHrp.Position - hrp.Position).Magnitude
                print("👁️ Seek знайдено! Дистанція: " .. math.floor(distance))
            end
        end
    else
        print("✅ Seek не знайдено (безпечно)")
    end
end)
CreateButton(SeekTab.Content, "Телепорт до Seek", function()
    local seek, hrp = FindSeek()
    if seek and hrp then
        local character = LocalPlayer.Character
        if character then
            local playerHrp = character:FindFirstChild("HumanoidRootPart")
            if playerHrp then
                playerHrp.CFrame = hrp.CFrame * CFrame.new(0, 0, 10)
            end
        end
    end
end)

-- ВКЛАДКА ІНШЕ
local MiscTab = CreateTab("Інше", "⚙️")
CreateLabel(MiscTab.Content, "⚙️ ІНШЕ")
CreateSection(MiscTab.Content, "Візуал")
CreateToggle(MiscTab.Content, "Fullbright", false, function(state)
    Flags.Fullbright = state
    if state then
        Lighting.Ambient = Color3.fromRGB(255, 255, 255)
        Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
        Lighting.Brightness = 1.5
        Lighting.ClockTime = 12
    else
        Lighting.Ambient = Color3.fromRGB(127, 127, 127)
        Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
        Lighting.Brightness = 1
    end
end)
CreateToggle(MiscTab.Content, "Авто відкриття дверей", false, function(state)
    Flags.AutoOpenDoors = state
end)
CreateSection(MiscTab.Content, "Телепорт")
CreateButton(MiscTab.Content, "Телепорт до наступних дверей", function()
    local doors = workspace:GetDescendants()
    local nearestDoor = nil
    local nearestDistance = math.huge
    for _, v in ipairs(doors) do
        if v:IsA("Model") and v.Name:lower():find("door") then
            local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Door")
            if hrp then
                local distance = (LocalPlayer.Character:FindFirstChild("HumanoidRootPart").Position - hrp.Position).Magnitude
                if distance < nearestDistance and distance > 10 then
                    nearestDistance = distance
                    nearestDoor = hrp
                end
            end
        end
    end
    if nearestDoor then
        LocalPlayer.Character:FindFirstChild("HumanoidRootPart").CFrame = nearestDoor.CFrame
    end
end)
CreateButton(MiscTab.Content, "Знайти ключ", function()
    local keys = workspace:GetDescendants()
    for _, v in ipairs(keys) do
        if v:IsA("BasePart")        if v:IsA("BasePart") and v.Name:lower():find("key") then
            LocalPlayer.Character:FindFirstChild("HumanoidRootPart").CFrame = v.CFrame
            break
        end
    end
end)
CreateSection(MiscTab.Content, "Сервер")
CreateButton(MiscTab.Content, "Перезайти на сервер", function()
    local ts = game:GetService("TeleportService")
    ts:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
end)
CreateButton(MiscTab.Content, "Змінити сервер", function()
    local ts = game:GetService("TeleportService")
    local placeId = game.PlaceId
    local jobId = game.JobId
    ts:TeleportToPlaceInstance(placeId, jobId, LocalPlayer)
end)
CreateSection(MiscTab.Content, "Персонаж")
CreateButton(MiscTab.Content, "Відродити персонажа", function()
    LocalPlayer.Character:BreakJoints()
end)
CreateButton(MiscTab.Content, "Скопіювати Loadstring", function()
    setclipboard("loadstring(game:HttpGet('https://raw.githubusercontent.com/yourname/doorshub/main/DoorsUltimateHub.lua'))()")
end)

-- =============================================
-- ГОЛОВНИЙ ЦИКЛ
-- =============================================
RunService.RenderStepped:Connect(function()
    local character = LocalPlayer.Character
    if not character then return end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end

    -- Оновлюємо ESP
    if Flags.ESP or Flags.ESPDoors or Flags.ESPGold or Flags.ESPCupboards or Flags.ESPBeds then
        UpdateESP()
    end

    -- Нескінченний стрибок
    if Flags.InfiniteJump then
        UserInputService.JumpRequest:Connect(function()
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end

    -- Noclip
    if Flags.Noclip then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end

    -- SEEK HELPER
    if SeekHelperActive then
        local seek, seekHrp = FindSeek()
        if seek and seekHrp and hrp then
            local distance = (hrp.Position - seekHrp.Position).Magnitude
            if distance < 100 then
                CreateSeekWarning()
            end
        end
    end

    -- KNOB FARM (старий)
    if KnobFarmActive then
        task.spawn(function()
            if not KnobFarmActive then return end
            
            local char = LocalPlayer.Character
            if not char then return end
            
            local root = char:FindFirstChild("HumanoidRootPart")
            if not root then return end
            
            local knobs = FindKnobs()
            for _, knob in ipairs(knobs) do
                local distance = (root.Position - knob.Position).Magnitude
                if distance < 50 then
                    root.CFrame = knob.CFrame * CFrame.new(0, 2, 0)
                    task.wait(Flags.KnobFarmSpeed)
                    
                    for _, child in ipairs(knob:GetChildren()) do
                        if child:IsA("ClickDetector") then
                            fireclickdetector(child)
                        end
                    end
                    
                    local parent = knob.Parent
                    if parent then
                        for _, child in ipairs(parent:GetChildren()) do
                            if child:IsA("ClickDetector") then
                                fireclickdetector(child)
                            end
                        end
                    end
                end
            end
        end)
    end

    -- Авто відкриття дверей
    if Flags.AutoOpenDoors then
        for _, v in ipairs(workspace:GetDescendants()) do
            if v:IsA("Model") and v.Name:lower():find("door") then
                local door = v:FindFirstChild("Door") or v:FindFirstChild("MainDoor")
                if door then
                    local distance = (hrp.Position - door.Position).Magnitude
                    if distance < 15 then
                        for _, child in ipairs(door:GetChildren()) do
                            if child:IsA("ClickDetector") then
                                fireclickdetector(child)
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- DEATH FARM LOOP
task.spawn(function()
    while task.wait(DeathFarmDelay) do
        if DeathFarmActive and not IsRespawning then
            PerformDeath()
        end
    end
end)

-- ESP для монстрів (старий)
task.spawn(function()
    while task.wait(0.1) do
        if Flags.ESP then
            for _, v in ipairs(workspace:GetDescendants()) do
                if v:IsA("Model") and (v.Name:lower():find("seek") or v.Name:lower():find("monster") or v.Name:lower():find("entity") or v.Name:lower():find("figure") or v.Name:lower():find("screech") or v.Name:lower():find("ambush") or v.Name:lower():find("halt") or v.Name:lower():find("rush")) then
                    local hrp = v:FindFirstChild("HumanoidRootPart") or v:FindFirstChild("Torso") or v:FindFirstChild("Head")
                    if hrp and not hrp:FindFirstChild("ESP_Monster") then
                        local esp = Instance.new("BillboardGui")
                        esp.Name = "ESP_Monster"
                        esp.Size = UDim2.new(0, 120, 0, 40)
                        esp.AlwaysOnTop = true
                        esp.Adornee = hrp
                        esp.Parent = hrp

                        local name = Instance.new("TextLabel")
                        name.Size = UDim2.new(1, 0, 0.5, 0)
                        name.BackgroundTransparency = 1
                        name.Text = "⚠️ " .. v.Name
                        name.TextColor3 = Color3.fromRGB(255, 0, 0)
                        name.TextSize = 14
                        name.Font = Enum.Font.GothamBold
                        name.Parent = esp

                        local dist = Instance.new("TextLabel")
                        dist.Size = UDim2.new(1, 0, 0.5, 0)
                        dist.Position = UDim2.new(0, 0, 0.5, 0)
                        dist.BackgroundTransparency = 1
                        dist.Text = "Дистанція: " .. math.floor((LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart").Position - hrp.Position).Magnitude)
                        dist.TextColor3 = Color3.fromRGB(255, 255, 255)
                        dist.TextSize = 11
                        dist.Font = Enum.Font.Gotham
                        dist.Parent = esp
                    end
                end
            end
        end
    end
end)

-- =============================================
-- ПОВІДОМЛЕННЯ
-- =============================================
local Notification = Instance.new("Frame")
Notification.Size = UDim2.new(0, 350, 0, 60)
Notification.Position = UDim2.new(0.5, -175, 0, -70)
Notification.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
Notification.BackgroundTransparency = 1
Notification.Parent = ScreenGui

local NotificationCorner = Instance.new("UICorner")
NotificationCorner.CornerRadius = UDim.new(0, 12)
NotificationCorner.Parent = Notification

local NotificationStroke = Instance.new("UIStroke")
NotificationStroke.Color = Color3.fromRGB(255, 50, 50)
NotificationStroke.Thickness = 2
NotificationStroke.Transparency = 0.5
NotificationStroke.Parent = Notification

local NotificationText = Instance.new("TextLabel")
NotificationText.Size = UDim2.new(1, 0, 1, 0)
NotificationText.BackgroundTransparency = 1
NotificationText.Text = "🚪 DOORS HUB v8.2 Завантажено!\n💀🔘 Knob Death Loop: збір → смерть → респавн!"
NotificationText.TextColor3 = Color3.fromRGB(255, 80, 80)
NotificationText.TextSize = 14
NotificationText.Font = Enum.Font.GothamBlack
NotificationText.Parent = Notification

TweenService:Create(Notification, TweenInfo.new(0.5), {BackgroundTransparency = 0.1, Position = UDim2.new(0.5, -175, 0, 10)}):Play()
task.wait(3)
TweenService:Create(Notification, TweenInfo.new(0.5), {BackgroundTransparency = 1, Position = UDim2.new(0.5, -175, 0, -70)}):Play()
task.wait(0.5)
Notification:Destroy()

print("✅ DOORS ULTIMATE HUB v8.2 Завантажено успішно!")


