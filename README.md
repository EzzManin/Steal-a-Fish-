-- LocalScript - EzzHub Roubar (Steal A Fish) com Noclip e Interação Instantânea
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Função para pegar HumanoidRootPart
local function getHRP()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:WaitForChild("HumanoidRootPart")
end

-- GUI
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.ResetOnSpawn = false

-- Botão abrir/fechar
local openBtn = Instance.new("ImageButton", gui)
openBtn.Size = UDim2.new(0, 50, 0, 50)
openBtn.Position = UDim2.new(0, 10, 0, 10)
openBtn.Image = "rbxassetid://133393242441626"
openBtn.BackgroundTransparency = 1
openBtn.Active = true
openBtn.Draggable = true

-- Menu
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 200)
frame.Position = UDim2.new(0.3, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.Active = true
frame.Draggable = true
frame.Visible = false

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "EzzHub"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20

-- Botão salvar local
local saveBtn = Instance.new("TextButton", frame)
saveBtn.Size = UDim2.new(1, -10, 0, 30)
saveBtn.Position = UDim2.new(0, 5, 0, 40)
saveBtn.Text = "Salvar Local"
saveBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
saveBtn.TextColor3 = Color3.new(1, 1, 1)

-- Toggle Roubar
local toggleBtn = Instance.new("TextButton", frame)
toggleBtn.Size = UDim2.new(1, -10, 0, 30)
toggleBtn.Position = UDim2.new(0, 5, 0, 80)
toggleBtn.Text = "Roubar: OFF"
toggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
toggleBtn.TextColor3 = Color3.new(1, 1, 1)

-- Toggle Noclip
local noclipBtn = Instance.new("TextButton", frame)
noclipBtn.Size = UDim2.new(1, -10, 0, 30)
noclipBtn.Position = UDim2.new(0, 5, 0, 120)
noclipBtn.Text = "Noclip: OFF"
noclipBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
noclipBtn.TextColor3 = Color3.new(1, 1, 1)

-- Toggle Interação Instantânea
local instantBtn = Instance.new("TextButton", frame)
instantBtn.Size = UDim2.new(1, -10, 0, 30)
instantBtn.Position = UDim2.new(0, 5, 0, 160)
instantBtn.Text = "Interassao Instantânea: OFF"
instantBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
instantBtn.TextColor3 = Color3.new(1, 1, 1)

-- Estado
local savedCFrame = nil
local roubarOn = false
local noclipOn = false
local instantOn = false
local noclipConn
local instantConn

-- Abrir/fechar menu
openBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

-- Salvar local
saveBtn.MouseButton1Click:Connect(function()
    local hrp = getHRP()
    savedCFrame = hrp.CFrame
    saveBtn.Text = "Local Salvo!"
    task.delay(1, function()
        saveBtn.Text = "Salvar Local"
    end)
end)

-- Função noclip
local function enableNoclip()
    if not noclipConn then
        noclipConn = RunService.Stepped:Connect(function()
            local char = player.Character
            if char then
                for _, part in ipairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end
end

local function disableNoclip()
    if noclipConn then
        noclipConn:Disconnect()
        noclipConn = nil
    end
end

-- Função interação instantânea
local function enableInstant()
    if not instantConn then
        instantConn = game.DescendantAdded:Connect(function(obj)
            if obj:IsA("ProximityPrompt") then
                obj.HoldDuration = 0 -- sem delay
            end
        end)
        -- também aplicar aos já existentes
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                obj.HoldDuration = 0
            end
        end
    end
end

local function disableInstant()
    if instantConn then
        instantConn:Disconnect()
        instantConn = nil
    end
end

-- Toggle Noclip manual
noclipBtn.MouseButton1Click:Connect(function()
    noclipOn = not noclipOn
    noclipBtn.Text = "Noclip: " .. (noclipOn and "ON" or "OFF")
    if noclipOn then
        enableNoclip()
    else
        disableNoclip()
    end
end)

-- Toggle Interação Instantânea
instantBtn.MouseButton1Click:Connect(function()
    instantOn = not instantOn
    instantBtn.Text = "Interassao Instantânea: " .. (instantOn and "ON" or "OFF")
    if instantOn then
        enableInstant()
    else
        disableInstant()
    end
end)

-- Toggle Roubar
toggleBtn.MouseButton1Click:Connect(function()
    if not savedCFrame then
        toggleBtn.Text = "Salve antes!"
        task.delay(1, function()
            toggleBtn.Text = "Roubar: OFF"
        end)
        return
    end

    roubarOn = not roubarOn
    toggleBtn.Text = "Roubar: " .. (roubarOn and "ON" or "OFF")

    if roubarOn then
        enableNoclip()
        task.spawn(function()
            local hrp = getHRP()
            local bv = Instance.new("BodyVelocity")
            bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            bv.Parent = hrp
            while roubarOn and hrp and savedCFrame do
                local targetPos = savedCFrame.Position + Vector3.new(0, 5, 0)
                local dir = (targetPos - hrp.Position)
                if dir.Magnitude < 4 then break end
                bv.Velocity = dir.Unit * 50
                task.wait()
            end
            bv:Destroy()
            if not noclipOn then
                disableNoclip()
            end
            roubarOn = false
            toggleBtn.Text = "Roubar: OFF"
        end)
    else
        if not noclipOn then
            disableNoclip()
        end
    end
end)
