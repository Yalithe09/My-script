local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Teleport Hub",
   Icon = 0,
   LoadingTitle = "Teleport GUI",
   LoadingSubtitle = "by Yahli",
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

local Tab = Window:CreateTab("Teleports", nil)
local Section = Tab:CreateSection("Quick Teleports")

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")

-- כפתור לשיגור ל-Bag ול-SafetyStatue
Tab:CreateButton({
    Name = "Teleport to Bag & SafetyStatue",
    Callback = function()
        -- חפש את האובייקטים Bag ו-SafetyStatue תחת Idols
        local bag = workspace:FindFirstChild("Idols") and workspace.Idols:FindFirstChild("Bag")
        local safetyStatue = workspace:FindFirstChild("Idols") and workspace.Idols:FindFirstChild("SafetyStatue")

        -- אם Bag נמצא ויש לו חלק 'hit'
        if bag and bag:FindFirstChild("hit") then
            print("Teleporting to Bag!")
            rootPart.CFrame = bag.hit.CFrame + Vector3.new(0, 3, 0)  -- שיגור ל-CFrame של hit + גובה קטן
            task.wait(1)  -- המתן לפני השיגור הבא
        else
            warn("Bag or hit part not found!")
        end

        -- אם SafetyStatue נמצא ויש לו חלק 'hit'
        if safetyStatue and safetyStatue:FindFirstChild("hit") then
            print("Teleporting to SafetyStatue!")
            rootPart.CFrame = safetyStatue.hit.CFrame + Vector3.new(0, 3, 0)  -- שיגור ל-CFrame של hit + גובה קטן
        else
            warn("SafetyStatue or hit part not found!")
        end
    end,
})

-- כפתור לשיגור לכל ה-Finish
Tab:CreateButton({
    Name = "Teleport to All Finishes",
    Callback = function()
        local courses = {
            "Bootcamp",
            "Colosseum Climb",
            "Obstacle Course",
            "Pond Pier",
            "Lava Dash",
            "Hill Hike",
            "Cliff Diving",
            "Construct Course",
            "Rickety Rails",
            "Rockwall",
            "Tightrope Obby",
            "Unstable Savannah",
            "Cave Chaos"
        }

        if workspace:FindFirstChild("Assets") then
            for _, courseName in ipairs(courses) do
                local course = workspace.Assets:FindFirstChild(courseName)
                if course and course:FindFirstChild("Finish") then
                    -- סיימנו עם touchinterest
                    rootPart.CFrame = course.Finish.CFrame + Vector3.new(0, 3, 0)  -- שיגור ישירות ל-Finish
                    task.wait(0.1) -- הוספתי השהייה קטנה כדי למנוע בעיות
                end
            end
        else
            warn("Assets לא נמצאו")
        end
    end,
})

-- כפתור לשינוי המהירות
Tab:CreateButton({
    Name = "Set Speed to 18",
    Callback = function()
        player.Character.Humanoid.WalkSpeed = 18
    end,
})

-- כפתור להפעיל את Chat Spy
Tab:CreateButton({
    Name = "Enable Chat Spy",
    Callback = function()
        --[[ 
        Warning: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
        ]]
        
        enabled = true --chat "/spy" to toggle!
        spyOnMyself = true --if true will check your messages too
        public = false --if true will chat the logs publicly (fun, risky)
        publicItalics = true --if true will use /me to stand out
        privateProperties = { --customize private logs
            Color = Color3.fromRGB(0,255,255); 
            Font = Enum.Font.SourceSansBold;
            TextSize = 18;
        }

        local StarterGui = game:GetService("StarterGui")
        local Players = game:GetService("Players")
        local player = Players.LocalPlayer or Players:GetPropertyChangedSignal("LocalPlayer"):Wait() or Players.LocalPlayer
        local saymsg = game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("SayMessageRequest")
        local getmsg = game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("OnMessageDoneFiltering")
        local instance = (_G.chatSpyInstance or 0) + 1
        _G.chatSpyInstance = instance

        local function onChatted(p,msg)
            if _G.chatSpyInstance == instance then
                if p==player and msg:lower():sub(1,4)=="/spy" then
                    enabled = not enabled
                    wait(0.3)
                    privateProperties.Text = "{SPY "..(enabled and "EN" or "DIS").."ABLED}"
                    StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
                elseif enabled and (spyOnMyself==true or p~=player) then
                    msg = msg:gsub("[\n\r]",''):gsub("\t",' '):gsub("[ ]+",' ')
                    local hidden = true
                    local conn = getmsg.OnClientEvent:Connect(function(packet,channel)
                        if packet.SpeakerUserId==p.UserId and packet.Message==msg:sub(#msg-#packet.Message+1) and (channel=="All" or (channel=="Team" and public==false and Players[packet.FromSpeaker].Team==player.Team)) then
                            hidden = false
                        end
                    end)
                    wait(1)
                    conn:Disconnect()
                    if hidden and enabled then
                        if public then
                            saymsg:FireServer((publicItalics and "/me " or '').."{SPY} [".. p.Name .."]: "..msg,"All")
                        else
                            privateProperties.Text = "{SPY} [".. p.Name .."]: "..msg
                            StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
                        end
                    end
                end
            end
        end

        for _,p in ipairs(Players:GetPlayers()) do
            p.Chatted:Connect(function(msg) onChatted(p,msg) end)
        end
        Players.PlayerAdded:Connect(function(p)
            p.Chatted:Connect(function(msg) onChatted(p,msg) end)
        end)
        privateProperties.Text = "{SPY "..(enabled and "EN" or "DIS").."ABLED}"
        StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
        if not player.PlayerGui:FindFirstChild("Chat") then wait(3) end
        local chatFrame = player.PlayerGui.Chat.Frame
        chatFrame.ChatChannelParentFrame.Visible = true
        chatFrame.ChatBarParentFrame.Position = chatFrame.ChatChannelParentFrame.Position+UDim2.new(UDim.new(),chatFrame.ChatChannelParentFrame.Size.Y)
    end,
})
