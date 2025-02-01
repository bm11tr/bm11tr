-- خدمات اللعبة
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- الحصول على اللاعب المحلي
local LocalPlayer = Players.LocalPlayer

-- إنشاء شاشة لعرض العناصر
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- جدول لتخزين بيانات ESP لكل لاعب
local ESPData = {}

-- دالة لإنشاء ESP للشخصية
local function CreateESP(player)
    if player == LocalPlayer then return end -- تجاهل اللاعب المحلي

    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    local rootPart = character:WaitForChild("HumanoidRootPart")

    -- إنشاء BillboardGui
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Size = UDim2.new(5, 0, 5, 0)
    billboardGui.Adornee = rootPart
    billboardGui.AlwaysOnTop = true
    billboardGui.Parent = ScreenGui

    -- إنشاء إطار للمربع
    local box = Instance.new("Frame")
    box.Size = UDim2.new(1, 0, 1, 0)
    box.BorderSizePixel = 2
    box.BorderColor3 = Color3.new(1, 1, 1)
    box.BackgroundColor3 = Color3.new(0, 0, 0)
    box.BackgroundTransparency = 0.7
    box.Parent = billboardGui

    -- إنشاء اسم اللاعب
    local nameTag = Instance.new("TextLabel")
    nameTag.Text = player.Name
    nameTag.TextColor3 = Color3.new(1, 1, 1)
    nameTag.TextScaled = true
    nameTag.Size = UDim2.new(1, 0, 0.2, 0)
    nameTag.BackgroundTransparency = 1
    nameTag.Parent = billboardGui

    -- إنشاء شريط الصحة
    local healthBarBackground = Instance.new("Frame")
    healthBarBackground.Size = UDim2.new(1, 0, 0.1, 0)
    healthBarBackground.Position = UDim2.new(0, 0, 1, 0)
    healthBarBackground.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    healthBarBackground.BorderSizePixel = 0
    healthBarBackground.Parent = billboardGui

    local healthBar = Instance.new("Frame")
    healthBar.Size = UDim2.new(1, 0, 1, 0)
    healthBar.BackgroundColor3 = Color3.new(0, 1, 0)
    healthBar.BorderSizePixel = 0
    healthBar.Parent = healthBarBackground

    -- تخزين البيانات في الجدول
    ESPData[player] = {
        BillboardGui = billboardGui,
        Box = box,
        NameTag = nameTag,
        HealthBar = healthBar,
        HealthBarBackground = healthBarBackground,
        Character = character,
        Humanoid = humanoid
    }

    -- تحديث شريط الصحة
    local function UpdateHealth()
        local health = humanoid.Health
        local maxHealth = humanoid.MaxHealth
        local healthPercentage = math.clamp(health / maxHealth, 0, 1)

        healthBar.Size = UDim2.new(healthPercentage, 0, 1, 0)
        if healthPercentage > 0.5 then
            healthBar.BackgroundColor3 = Color3.new(0, 1, 0) -- أخضر
        elseif healthPercentage > 0.2 then
            healthBar.BackgroundColor3 = Color3.new(1, 1, 0) -- أصفر
        else
            healthBar.BackgroundColor3 = Color3.new(1, 0, 0) -- أحمر
        end
    end

    -- ربط تحديث الصحة مع تغيير الصحة
    humanoid.HealthChanged:Connect(UpdateHealth)

    -- تحديث الصحة عند التحميل
    UpdateHealth()
end

-- دالة لإزالة ESP عند مغادرة اللاعب
local function RemoveESP(player)
    if ESPData[player] then
        ESPData[player].BillboardGui:Destroy()
        ESPData[player] = nil
    end
end

-- تتبع اللاعبين الحاليين
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

-- تتبع اللاعبين الجدد
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        CreateESP(player)
    end)
end)

-- إزالة ESP عند مغادرة اللاعب
Players.PlayerRemoving:Connect(RemoveESP)

-- تحديث الديناميكي للعناصر
RunService.RenderStepped:Connect(function()
    for player, data in pairs(ESPData) do
        if not data.Character or not data.Character.Parent then
            RemoveESP(player)
        elseif data.Humanoid.Health <= 0 then
            RemoveESP(player)
        else
            -- تحديث موقع Adornee إذا تغيرت الشخصية
            local rootPart = data.Character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                data.BillboardGui.Adornee = rootPart
            end
        end
    end
end)

-- تحديث تلقائي للمسافة
RunService.Heartbeat:Connect(function()
    for player, data in pairs(ESPData) do
        local rootPart = data.Character and data.Character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            local distance = (rootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            data.NameTag.Text = string.format("%s\n[%d studs]", player.Name, math.floor(distance))
        end
    end
end)

-- التنفيذ التلقائي باستخدام loadstring
local function AutoExecute()
    -- يمكنك هنا إضافة أي تعديلات إضافية إذا كنت تريد تنفيذ الكود من مصدر خارجي
    print("ESP Script has been executed automatically!")
end

-- تنفيذ البرنامج النصي تلقائيًا
AutoExecute()

-- إذا كنت تستخدم أدوات مثل Synapse X أو Krnl، يمكنك استخدام هذا السطر لتحميل الكود من مصدر خارجي
-- loadstring(game:HttpGet("رابط_الكود"))()
