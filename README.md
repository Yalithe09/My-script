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
local humanoid = character:WaitForChild("Humanoid")

-- כפתור לשיגור ל-Bag ול-SafetyStatue
Tab:CreateButton({
    Name = "Teleport to Bag & SafetyStatue",
    Callback = function()
        local bag = workspace:FindFirstChild("Idols") and workspace.Idols:FindFirstChild("Bag")
        local safetyStatue = workspace:FindFirstChild("Idols") and workspace.Idols:FindFirstChild("SafetyStatue")

        if bag and bag:FindFirstChild("hit") then
            rootPart.CFrame = bag.hit.CFrame + Vector3.new(0, 3, 0)
            task.wait(1)
        end

        if safetyStatue and safetyStatue:FindFirstChild("hit") then
            rootPart.CFrame = safetyStatue.hit.CFrame + Vector3.new(0, 3, 0)
        end
    end,
})

-- כפתור לשיגור לקורס הנוכחי
Tab:CreateButton({
    Name = "Teleport to Current Finish",
    Callback = function()
        -- בודקים את כל הקורסים בתוך Assets
        if workspace:FindFirstChild("Assets") then
            local assets = workspace.Assets:GetChildren()

            for _, course in ipairs(assets) do
                if course:IsA("Model") and course:FindFirstChild("Finish") then
                    local finish = course.Finish
                    if finish then
                        -- שיגור לשחקן לנקודת ה-Finish
                        rootPart.CFrame = finish.CFrame + Vector3.new(0, 3, 0)
                        return
                    end
                end
            end
            warn("לא נמצאה Finish באף קורס ב-Assets.")
        else
            warn("לא נמצאו Assets.")
        end
    end,
})

-- כפתור לשינוי מהירות
Tab:CreateButton({
    Name = "Set Speed to 18",
    Callback = function()
        humanoid.WalkSpeed = 18  -- שינוי המהירות ל-18
    end,
})

-- כפתור לחשיפת הצבעות
Tab:CreateButton({
    Name = "Reveal Votes",
    Callback = function()
        print("Reveal Votes executed.")

        local revealInChat = false

        local votes = game:GetService("ReplicatedStorage").Season.Voting.Votes
        local players = game:GetService("ReplicatedStorage").Season.Players

        local randomPremsg = {"Guys I think ", "idk if its true but maybe ", "hmm if I had to take a guess I'd say ", "you didn't hear this from me but ", "this is random but im pretty sure ", "Do you guys think ", "pretty sure ", "umm so like ", "guys ", "um ", "... I think "}

        votes.ChildAdded:Connect(function(vote)
            local msg = players[vote.Value].Value.." voted "..players[vote.Name].Value
            game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {Text = msg})
            if revealInChat then
                game.ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(randomPremsg[math.random(1, #randomPremsg)]..msg, 'All')
            end
        end)
    end,
})
