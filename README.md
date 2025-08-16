-- Toggle Interassao Instantânea
local instantToggle = Instance.new("TextButton", frame)
instantToggle.Size = UDim2.new(1, -10, 0, 30)
instantToggle.Position = UDim2.new(0, 5, 0, 160)
instantToggle.Text = "Interassao Instantânea: OFF"
instantToggle.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
instantToggle.TextColor3 = Color3.new(1, 1, 1)

local instantEnabled = false
local originalTimes = {} -- para restaurar depois
local instantConn -- conexão para novos prompts

local function setInstant(enabled)
    if enabled then
        -- salvar e alterar prompts existentes
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                originalTimes[obj] = obj.HoldDuration
                obj.HoldDuration = 0
            end
        end
        -- monitorar novos prompts
        instantConn = game.DescendantAdded:Connect(function(obj)
            if obj:IsA("ProximityPrompt") then
                originalTimes[obj] = obj.HoldDuration
                obj.HoldDuration = 0
            end
        end)
    else
        -- parar monitoramento
        if instantConn then
            instantConn:Disconnect()
            instantConn = nil
        end
        -- restaurar tempos originais
        for obj, time in pairs(originalTimes) do
            if obj and obj:IsA("ProximityPrompt") then
                obj.HoldDuration = time
            end
        end
        originalTimes = {}
    end
end

instantToggle.MouseButton1Click:Connect(function()
    instantEnabled = not instantEnabled
    setInstant(instantEnabled)
    instantToggle.Text = "Interassao Instantânea: " .. (instantEnabled and "ON" or "OFF")
    instantToggle.BackgroundColor3 = instantEnabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(80, 80, 80)
end)
