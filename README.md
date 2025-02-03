local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

local targetPlayer = nil
local aiming = false
local bulletSpeed = 500  -- 子彈速度

-- UI 元件
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "AimControl"

local aimButton = Instance.new("TextButton")
aimButton.Size = UDim2.new(0, 200, 0, 50)
aimButton.Position = UDim2.new(1, -210, 0, 10)  -- 顯示在右上角
aimButton.Text = "Start Aiming"
aimButton.Visible = false  -- 初始隱藏
aimButton.Parent = screenGui

-- 判斷設備類型
local isMobile = UserInputService.TouchEnabled

-- 顯示設備控制方式
if isMobile then
    -- 在手機上顯示按鈕
    aimButton.Visible = true
else
    -- 在PC上，不顯示按鈕
    aimButton.Visible = false
end

-- 獲取隨機玩家作為目標
local function getRandomPlayer()
    local allPlayers = {}
    for _, player in pairs(Players:GetPlayers()) do
        -- 排除自己，沒有角色或 HumanoidRootPart 的玩家，並排除死亡的玩家
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            -- 排除死亡的玩家
            if player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                table.insert(allPlayers, player)
            end
        end
    end

    if #allPlayers > 0 then
        return allPlayers[math.random(1, #allPlayers)]  -- 隨機選擇一個可攻擊的玩家
    end
end

-- 子彈穿牆射擊
local function shootBullet(targetPosition)
    local startPosition = LocalPlayer.Character.HumanoidRootPart.Position
    local direction = (targetPosition - startPosition).unit * bulletSpeed

    -- 使用 Raycast 發射射線
    local rayOrigin = startPosition
    local rayDirection = direction

    -- 在發射射線時忽略所有物體（除了目標玩家）
    local ray = Ray.new(rayOrigin, rayDirection)
    local ignoreList = {LocalPlayer.Character}  -- 忽略自己的角色

    local hit, position = Workspace:FindPartOnRayWithWhitelist(ray, ignoreList)
    
    -- 如果射線擊中某個玩家
    if hit then
        local hitPlayer = Players:GetPlayerFromCharacter(hit.Parent)
        if hitPlayer and hitPlayer ~= LocalPlayer then
            print("Hit player: " .. hitPlayer.Name)
        end
    end
end

-- 設置自動瞄準目標
local function aimAtTarget()
    if targetPlayer and targetPlayer.Character then
        local targetPosition = targetPlayer.Character.HumanoidRootPart.Position

        -- 這裡移除了範圍檢查，將瞄準範圍設為無限
        local direction = (targetPosition - LocalPlayer.Character.HumanoidRootPart.Position).unit
        local targetLookAt = CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position, targetPosition)

        -- 設置角色的瞄準方向（使用 Camera CFrame）
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)

        -- 檢查目標的血量，如果血量為 0 則停止瞄準
        if targetPlayer.Character.Humanoid.Health <= 0 then
            stopAiming()  -- 目標血量為 0，停止瞄準
        end

        -- 當按下鍵盤按鈕時，射擊子彈
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            shootBullet(targetPosition)
        end
    end
end

-- 開始瞄準
local function startAiming()
    targetPlayer = getRandomPlayer()  -- 隨機選擇一個目標
    if targetPlayer then
        aiming = true
    end
end

-- 停止瞄準
local function stopAiming()
    aiming = false
    targetPlayer = nil
end

-- 監控每一幀，並執行自動瞄準
RunService.RenderStepped:Connect(function()
    if aiming then
        aimAtTarget()  -- 持續瞄準目標
    end
end)

-- 使用 N 鍵來切換瞄準
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if not isMobile and input.KeyCode == Enum.KeyCode.N then
        if aiming then
            stopAiming()
        else
            startAiming()
        end
    end
end)

-- 手機上使用 UI 按鈕切換瞄準
aimButton.MouseButton1Click:Connect(function()
    if aiming then
        stopAiming()
        aimButton.Text = "Start Aiming"  -- 改變按鈕文字
    else
        startAiming()
        aimButton.Text = "Stop Aiming"  -- 改變按鈕文字
    end
end)
