-- LocalScript - EzzHub Roubar (Steal A Fish) com Noclip
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
frame.Size = UDim2.new(0, 200, 0, 120)
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

-- Estado
local savedCFrame = nil
local roubarOn = false
local noclipConn

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

local function disableNoclip()
    if noclipConn then
        noclipConn:Disconnect()
        noclipConn = nil
    end
end

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
                bv.Velocity = dir.Unit * 50 -- velocidade mais lenta
                task.wait()
            end
            bv:Destroy()
            disableNoclip()
            roubarOn = false
            toggleBtn.Text = "Roubar: OFF"
        end)
    else
        disableNoclip()
    end
end)
