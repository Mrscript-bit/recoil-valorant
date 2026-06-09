-- Vandal Recoil Control Script
-- Logitech G HUB
-- Game: Valorant

-- ┌─────────────────────────────────────┐
-- │         SETTINGS                    │
-- └─────────────────────────────────────┘
local yBase      = 5      -- base downward pull strength
local xDrift     = 0      -- horizontal correction (+ = right, - = left)
local interval   = 10     -- milliseconds between each step (match your fire rate)
local sensitivity = 0.45  -- tuned for 0.177 sens @ 1600 DPI

-- ┌─────────────────────────────────────┐
-- │         TOGGLE SETTINGS             │
-- └─────────────────────────────────────┘
-- Toggle key: set to the G-key or button you want to use
-- G-keys: "G1", "G2", "G3", etc.
-- Mouse buttons: use arg numbers (4 = Mouse4, 5 = Mouse5)
-- Set TOGGLE_TYPE to "GKEY" or "MOUSE"
local TOGGLE_TYPE   = "MOUSE" -- "GKEY" or "MOUSE"
local TOGGLE_KEY    = "G1"    -- G-key name (used when TOGGLE_TYPE = "GKEY")
local TOGGLE_BUTTON = 5       -- Mouse button number (G304 front side button = Mouse4)

-- ┌─────────────────────────────────────┐
-- │  VANDAL RECOIL PATTERN (25 bullets) │
-- └─────────────────────────────────────┘
-- Each entry is {x, y} relative movement per bullet
-- Negative x = pull left, Positive x = pull right
-- Positive y = pull down
local recoilPattern = {
    {  0,  4 },  -- 1
    {  0,  5 },  -- 2
    { -1,  6 },  -- 3
    { -1,  6 },  -- 4
    { -2,  5 },  -- 5
    { -2,  5 },  -- 6
    { -1,  4 },  -- 7
    {  0,  4 },  -- 8
    {  1,  4 },  -- 9
    {  2,  4 },  -- 10
    {  2,  4 },  -- 11
    {  1,  3 },  -- 12
    {  0,  3 },  -- 13
    { -1,  3 },  -- 14
    { -2,  3 },  -- 15
    { -2,  3 },  -- 16
    { -1,  3 },  -- 17
    {  0,  3 },  -- 18
    {  1,  3 },  -- 19
    {  2,  3 },  -- 20
    {  2,  3 },  -- 21
    {  1,  2 },  -- 22
    {  0,  2 },  -- 23
    {  0,  2 },  -- 24
    {  0,  2 },  -- 25
}

-- ┌─────────────────────────────────────┐
-- │         SCRIPT LOGIC                │
-- └─────────────────────────────────────┘
local shooting  = false
local rcEnabled = true  -- recoil compensation starts ON

-- Helper: print current toggle state to G HUB log
local function printState()
    if rcEnabled then
        OutputLogMessage("[Vandal RCS] >> ENABLED\n")
    else
        OutputLogMessage("[Vandal RCS] >> DISABLED\n")
    end
end

function OnEvent(event, arg)
    if event == "PROFILE_ACTIVATED" then
        EnablePrimaryMouseButtonEvents(true)
        OutputLogMessage("[Vandal RCS] Profile loaded. RCS is ON. Press toggle key to switch.\n")
    end

    -- ── Toggle logic ──────────────────────────────────────────
    if TOGGLE_TYPE == "GKEY" then
        if event == "G_PRESSED" and arg == tonumber(string.sub(TOGGLE_KEY, 2)) then
            rcEnabled = not rcEnabled
            printState()
        end
    elseif TOGGLE_TYPE == "MOUSE" then
        if event == "MOUSE_BUTTON_PRESSED" and arg == TOGGLE_BUTTON then
            rcEnabled = not rcEnabled
            printState()
            return  -- don't let this button do anything else
        end
    end

    -- ── Recoil compensation (only when enabled) ───────────────
    if event == "MOUSE_BUTTON_PRESSED" and arg == 1 then
        if not rcEnabled then return end

        shooting = true
        local step = 1

        repeat
            if step <= #recoilPattern then
                local dx = math.floor(recoilPattern[step][1] * sensitivity)
                local dy = math.floor(recoilPattern[step][2] * sensitivity)
                MoveMouseRelative(dx, dy)
                step = step + 1
            else
                -- After pattern ends, apply steady base pull
                MoveMouseRelative(xDrift, yBase)
            end
            Sleep(interval)
        until not IsMouseButtonPressed(1)

        shooting = false
    end
end
