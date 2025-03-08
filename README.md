local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Yali's total roblox drama Script",
    Icon = 0,
    LoadingTitle = "Yali's total roblox drama Script",
    LoadingSubtitle = "by Yali",
    Theme = "Default",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "TeleportHub"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    },
    KeySystem = false
})

local Tab = Window:CreateTab("Fun cheats", nil)
local Section = Tab:CreateSection("Trd")

local player = game.Players.LocalPlayer
local rootPart = player.Character and player.Character:WaitForChild("HumanoidRootPart")

local function teleportPlayer(player)
    local idolsFolder = workspace:WaitForChild("Idols")
    local bag = idolsFolder:FindFirstChild("Bag")
    local safetyStatue = idolsFolder:FindFirstChild("SafetyStatue")

    if bag and safetyStatue then
        local bagHit = bag:FindFirstChild("hit")
        local statueHit = safetyStatue:FindFirstChild("hit")

        if bagHit and statueHit then
            local character = player.Character or player.CharacterAdded:Wait()
            local hrp = character:WaitForChild("HumanoidRootPart")

            hrp.CFrame = bagHit.CFrame
            task.wait(1)
            hrp.CFrame = statueHit.CFrame
        else
            warn("לא נמצאו חלקי hit")
        end
    else
        warn("לא נמצאו Bag או SafetyStatue")
    end
end

Tab:CreateButton({
    Name = "Teleport to Bag and Safety Statue",
    Callback = function()
        teleportPlayer(player)
    end,
})

Tab:CreateButton({
    Name = "Teleport to Finishes in Obbies",
    Callback = function()
        local character = player.Character or player.CharacterAdded:Wait()
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if not rootPart then
            warn("❌ לא נמצא HumanoidRootPart!")
            return
        end

        local assets = workspace:FindFirstChild("Assets")
        if assets then
            for _, folder in ipairs(assets:GetChildren()) do
                if folder:IsA("Folder") then
                    local finish = folder:FindFirstChild("Finish")
                    if finish and finish:IsA("BasePart") then
                        rootPart.CFrame = finish.CFrame + Vector3.new(0, 5, 0)
                        print("✅ שוגר ל-" .. folder.Name .. " (Finish)")
                        task.wait(1) -- הארכתי את ההמתנה שתהיה בטוחה יותר
                    else
                        print("⚠️ אין Finish בתוך " .. folder.Name)
                    end
                end
            end
        else
            warn("❌ Assets לא נמצאה ב-Workspace!")
        end
    end,
})


Tab:CreateButton({
    Name = "Set Speed to 18 (helpful in obbies to pass other people faster)",
    Callback = function()
        if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            player.Character.Humanoid.WalkSpeed = 18
        else
            warn("❌ לא נמצא Humanoid לשחקן.")
        end
    end,
})

Tab:CreateButton({
    Name = "Enable Chat Spy (see private chats of other people)",
    Callback = function()
        enabled = true
        spyOnMyself = true
        public = false
        publicItalics = true
        privateProperties = {
            Color = Color3.fromRGB(0, 255, 255),
            Font = Enum.Font.SourceSansBold,
            TextSize = 18,
        }

        local StarterGui = game:GetService("StarterGui")
        local Players = game:GetService("Players")
        local saymsg = game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("SayMessageRequest")
        local getmsg = game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("OnMessageDoneFiltering")
        local instance = (_G.chatSpyInstance or 0) + 1
        _G.chatSpyInstance = instance

        local function onChatted(p, msg)
            if _G.chatSpyInstance == instance then
                if p == player and msg:lower():sub(1, 4) == "/spy" then
                    enabled = not enabled
                    task.wait(0.3)
                    privateProperties.Text = "{SPY " .. (enabled and "EN" or "DIS") .. "ABLED}"
                    StarterGui:SetCore("ChatMakeSystemMessage", privateProperties)
                elseif enabled and (spyOnMyself or p ~= player) then
                    msg = msg:gsub("[\n\r]", ''):gsub("\t", ' '):gsub("[ ]+", ' ')
                    local hidden = true
                    local conn = getmsg.OnClientEvent:Connect(function(packet, channel)
                        if packet.SpeakerUserId == p.UserId and packet.Message == msg:sub(#msg - #packet.Message + 1) and (channel == "All" or (channel == "Team" and not public and Players[packet.FromSpeaker].Team == player.Team)) then
                            hidden = false
                        end
                    end)
                    task.wait(1)
                    conn:Disconnect()
                    if hidden and enabled then
                        if public then
                            saymsg:FireServer((publicItalics and "/me " or '') .. "{SPY} [" .. p.Name .. "]: " .. msg, "All")
                        else
                            privateProperties.Text = "{SPY} [" .. p.Name .. "]: " .. msg
                            StarterGui:SetCore("ChatMakeSystemMessage", privateProperties)
                        end
                    end
                end
            end
        end

        for _, p in ipairs(Players:GetPlayers()) do
            p.Chatted:Connect(function(msg) onChatted(p, msg) end)
        end
        Players.PlayerAdded:Connect(function(p)
            p.Chatted:Connect(function(msg) onChatted(p, msg) end)
        end)
        privateProperties.Text = "{SPY " .. (enabled and "EN" or "DIS") .. "ABLED}"
        StarterGui:SetCore("ChatMakeSystemMessage", privateProperties)
    end,
})

Tab:CreateButton({
    Name = "See Votes (VERY OP and click it ONLY ONCE and then wait for elimination and look in the chat. Only you can see the votes!)",
    Callback = function()
        print("Reveal Votes executed.")

        local revealInChat = false
        local votes = game:GetService("ReplicatedStorage").Season.Voting.Votes
        local players = game:GetService("ReplicatedStorage").Season.Players
        local randomPremsg = {"Guys I think ", "idk if its true but maybe ", "hmm if I had to take a guess I'd say ", "you didn't hear this from me but ", "this is random but im pretty sure ", "Do you guys think ", "pretty sure ", "umm so like ", "guys ", "um ", "... I think "}

        votes.ChildAdded:Connect(function(vote)
            local msg = players[vote.Value].Value .. " voted " .. players[vote.Name].Value
            game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {Text = msg})
            if revealInChat then
                game.ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(randomPremsg[math.random(1, #randomPremsg)] .. msg, "All")
            end
        end)
    end,
})

Tab:CreateButton({
    Name = "Show Username Above All Players (except me)",
    Callback = function()
        local function addBillboard(plr)
            if plr == player then return end -- לא להוסיף לעצמי

            local function onCharacterAdded(character)
                local head = character:WaitForChild("Head")
                if head:FindFirstChild("UsernameBillboard") then return end

                local billboard = Instance.new("BillboardGui")
                billboard.Name = "UsernameBillboard"
                billboard.Parent = head
                billboard.Adornee = head
                billboard.Size = UDim2.new(0, 100, 0, 25) -- גודל קבוע
                billboard.StudsOffset = Vector3.new(0, 1.7, 0) -- טיפה מעל הראש אבל לא גבוה מדי
                billboard.AlwaysOnTop = false -- לא תמיד מעל דברים
                billboard.MaxDistance = 100 -- מקסימום מרחק להצגה

                local textLabel = Instance.new("TextLabel")
                textLabel.Parent = billboard
                textLabel.Size = UDim2.new(1, 0, 1, 0)
                textLabel.BackgroundTransparency = 1
                textLabel.Text = plr.Name
                textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                textLabel.TextStrokeTransparency = 0.5
                textLabel.Font = Enum.Font.SourceSansBold
                textLabel.TextScaled = true
            end

            if plr.Character then
                onCharacterAdded(plr.Character)
            end
            plr.CharacterAdded:Connect(onCharacterAdded)
        end

        -- לשחקנים שכבר במשחק
        for _, plr in ipairs(game.Players:GetPlayers()) do
            addBillboard(plr)
        end

        -- לשחקנים חדשים
        game.Players.PlayerAdded:Connect(function(plr)
            addBillboard(plr)
        end)
    end,
})


