-- Estado original dos prompts
local originalHoldTimes = {}

-- Função interação instantânea
local function enableInstant()
    if not instantConn then
        instantConn = game.DescendantAdded:Connect(function(obj)
            if obj:IsA("ProximityPrompt") then
                if originalHoldTimes[obj] == nil then
                    originalHoldTimes[obj] = obj.HoldDuration
                end
                obj.HoldDuration = 0
            end
        end)
        -- aplicar aos já existentes
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                if originalHoldTimes[obj] == nil then
                    originalHoldTimes[obj] = obj.HoldDuration
                end
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
    -- restaurar valores originais
    for prompt, time in pairs(originalHoldTimes) do
        if prompt and prompt.Parent then
            prompt.HoldDuration = time
        end
    end
    originalHoldTimes = {}
end

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
