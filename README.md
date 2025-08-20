# Script-test
--[[  
    Pistola destructiva universal
    Destruye todo lo que toque
    Script completo listo para StarterPack
    Rayo aparece unos studs delante del cañón
--]]

local PlayersService = game:GetService("Players")
local DebrisService = game:GetService("Debris")
local WorkspaceService = game:GetService("Workspace")

local player = PlayersService.LocalPlayer
local backpack = player:WaitForChild("Backpack")

-- Configuración del disparo
local SHOT_SPEED = 200
local SHOT_TIME = 5
local NOZZLE_OFFSET = Vector3.new(0,0.4,-1.1)
local RAY_OFFSET = 2 -- distancia para que el rayo no destruya la pistola

-- Crear modelo temporal
local mas = Instance.new("Model")
mas.Name = "CompiledModel"

-- Crear herramienta
local Tool = Instance.new("Tool")
Tool.Name = "LaserGun"
Tool.TextureId = "http://www.roblox.com/asset?id=13838639"
Tool.GripPos = Vector3.new(0, -0.1, 0.75)
Tool.CanBeDropped = false
Tool.Parent = mas

-- Crear handle y mesh
local Handle = Instance.new("Part")
Handle.Name = "Handle"
Handle.Size = Vector3.new(0.58,1.34,2.48)
Handle.FormFactor = Enum.FormFactor.Custom
Handle.CanCollide = false
Handle.BottomSurface = Enum.SurfaceType.Smooth
Handle.TopSurface = Enum.SurfaceType.Smooth
Handle.Parent = Tool

local Mesh = Instance.new("SpecialMesh")
Mesh.MeshId = "http://www.roblox.com/asset?id=130099641"
Mesh.TextureId = "http://www.roblox.com/asset/?id=130267361"
Mesh.Scale = Vector3.new(0.65,0.65,0.65)
Mesh.Parent = Handle

-- Sonidos
local FireSound = Instance.new("Sound")
FireSound.Name = "Fire"
FireSound.SoundId = "http://www.roblox.com/asset?id=114488148"
FireSound.Pitch = 1.2
FireSound.Volume = 1
FireSound.Parent = Handle

local ReloadSound = Instance.new("Sound")
ReloadSound.Name = "Reload"
ReloadSound.SoundId = "http://www.roblox.com/asset?id=94255248"
ReloadSound.Pitch = 0.5
ReloadSound.Parent = Handle

local HitFadeSound = Instance.new("Sound")
HitFadeSound.Name = "HitFade"
HitFadeSound.SoundId = "http://www.roblox.com/asset?id=130113415"
HitFadeSound.Parent = Tool

local PointLight = Instance.new("PointLight")
PointLight.Color = Color3.new(1,0,0)
PointLight.Brightness = 50
PointLight.Range = 10
PointLight.Parent = Handle

-- Base del disparo
local BaseShot = Instance.new("Part")
BaseShot.Name = "Effect"
BaseShot.Size = Vector3.new(0.2,0.2,3)
BaseShot.BrickColor = BrickColor.new("Really red")
BaseShot.CanCollide = false
BaseShot.Anchored = false
BaseShot.Parent = Tool

-- Función que destruye cualquier objeto tocado
local function OnTouched(shot, otherPart)
    if otherPart and otherPart:IsA("BasePart") then
        otherPart:Destroy()
        if shot then
            shot:Destroy()
        end
    end
end

-- Disparo
local function OnActivated()
    local character = Tool.Parent
    local humanoid = character:FindFirstChild("Humanoid")
    if Tool.Enabled and humanoid and humanoid.Health > 0 then
        Tool.Enabled = false
        FireSound:Play()

        -- Obtener punto de disparo
        local handleCFrame = Handle.CFrame
        local firingPoint = handleCFrame.Position + handleCFrame:VectorToWorldSpace(NOZZLE_OFFSET)
        firingPoint = firingPoint + handleCFrame.LookVector * RAY_OFFSET -- desplazar rayo para no destruir pistola

        -- Obtener dirección del mouse
        local mouse = player:GetMouse()
        local targetPoint = mouse.Hit.p
        local shotCFrame = CFrame.new(firingPoint, targetPoint)

        -- Crear el láser
        local laserShotClone = BaseShot:Clone()
        laserShotClone.CFrame = shotCFrame
        laserShotClone.Anchored = false

        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = (targetPoint - firingPoint).Unit * SHOT_SPEED
        bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
        bodyVelocity.Parent = laserShotClone

        laserShotClone.Touched:Connect(function(part)
            OnTouched(laserShotClone, part)
        end)

        laserShotClone.Parent = WorkspaceService
        DebrisService:AddItem(laserShotClone, SHOT_TIME)

        wait(0.1)
        ReloadSound:Play()
        Tool.Enabled = true
    end
end

-- Mouse icon
local function SetupMouseIcon(mouse)
    local MOUSE_ICON = 'rbxasset://textures/GunCursor.png'
    local RELOADING_ICON = 'rbxasset://textures/GunWaitCursor.png'
    mouse.Icon = Tool.Enabled and MOUSE_ICON or RELOADING_ICON
    Tool.Changed:Connect(function(prop)
        if prop == "Enabled" then
            mouse.Icon = Tool.Enabled and MOUSE_ICON or RELOADING_ICON
        end
    end)
end

-- Conexiones
Tool.Activated:Connect(OnActivated)
Tool.Equipped:Connect(function(mouse)
    SetupMouseIcon(mouse)
end)

-- Entregar la herramienta al jugador
for _, item in pairs(mas:GetChildren()) do
    item.Parent = backpack
end

mas:Destroy()
