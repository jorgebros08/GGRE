--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
getgenv().silentaim_settings = {
    fov = 150,
    fovcircle = true,
}

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Player
local Player = Players.LocalPlayer
local Mouse = Player:GetMouse()
local CurrentCamera = workspace.CurrentCamera

local function GetClosestPart()
    local Target, Closest = nil, getgenv().silentaim_settings.fov or math.huge

    for _, v in pairs(Players:GetPlayers()) do
        if v ~= Player and v.Character and v.Character:FindFirstChild("Humanoid") then
            local humanoid = v.Character:FindFirstChild("Humanoid")
            local humanoidRootPart = v.Character:FindFirstChild("HumanoidRootPart")
            
            if humanoid and humanoidRootPart then
                local partsToCheck = {
                    v.Character.Head,
                    v.Character.Torso,
                    v.Character["Right Leg"],
                    v.Character["Left Leg"],
                    v.Character["Right Arm"],
                    v.Character["Left Arm"],
                }

                for _, part in pairs(partsToCheck) do
                    if part then
                        local position, onScreen = CurrentCamera:WorldToScreenPoint(part.Position)
                        local distance = (Vector2.new(position.X, position.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude

                        if distance < Closest and onScreen then
                            Closest = distance
                            Target = part
                        end
                    end
                end
            end
        end
    end

    return Target
end

local Target
local CircleInline = Drawing.new("Circle")
local CircleOutline = Drawing.new("Circle")

RunService.RenderStepped:Connect(function()
    CircleInline.Radius = getgenv().silentaim_settings.fov
    CircleInline.Thickness = 2
    CircleInline.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
    CircleInline.Transparency = 1
    CircleInline.Color = Color3.fromRGB(255, 255, 255)
    CircleInline.Visible = getgenv().silentaim_settings.fovcircle
    CircleInline.ZIndex = 2

    CircleOutline.Radius = getgenv().silentaim_settings.fov
    CircleOutline.Thickness = 4
    CircleOutline.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
    CircleOutline.Transparency = 1
    CircleOutline.Color = Color3.new()
    CircleOutline.Visible = getgenv().silentaim_settings.fovcircle
    CircleOutline.ZIndex = 1

    Target = GetClosestPart()
end)

local Old;
Old = hookmetamethod(game, "__namecall", function(Self, ...)
    local Args = {...}
    
    if (not checkcaller() and getnamecallmethod() == "FindPartOnRayWithIgnoreList") then
        if (table.find(Args[2], workspace.WorldIgnore.Ignore) and Target) then
            local Origin = Args[1].Origin
            Args[1] = Ray.new(Origin, Target.Position - Origin)
        end
    end
    
    return Old(Self, unpack(Args))
end)
