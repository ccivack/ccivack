- 👋 Hi, I’m @ccivack
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...-- تعيين متغيرات التحكم
local switch = 9    -- زر تشغيل/إيقاف الارتداد (mouse button: 3 -> on/off)
local minus = 7     -- زر تقليل سرعة الإطلاق
local plus = 8      -- زر زيادة سرعة الإطلاق
local aimBotKey = 4  -- مفتاح التصويب التلقائي (G4)

-- متغيرات للتحكم في الارتداد وسرعة الإطلاق
local recoilOn = false
local step_recoil = 1
local aimBotOn = false

-- متغيرات للتحكم في الاهتزاز الأفقي
local horizontalVibration = 5  -- سرعة الاهتزاز الأفقي (يمكنك ضبطها بناءً على تفضيلاتك)

-- متغير لون الهدف ومدى الاستهداف
local targetColor = { r = 255, g = 0, b = 0 }  -- هنا سنقوم بالتصويب نحو اللون الأحمر
local targetRange = 100

-- تعيين متغيرات لوحة المفاتيح للتحكم في الارتداد وسرعة الإطلاق (اختياري)
local g_switch = nil
local g_plus = nil
local g_minus = nil

function AutoAim(targetX, targetY)
    local currentX, currentY = GetMousePosition()
    local moveX, moveY = targetX - currentX, targetY - currentY
    MoveMouseTo(currentX + moveX, currentY + moveY)
end

function FindTarget(screen, startX, startY, targetColor, targetRange)
    local screenWidth, screenHeight = GetScreenResolution()

    local function colorDistance(c1, c2)
        local dr, dg, db = c1.r - c2.r, c1.g - c2.g, c1.b - c2.b
        return dr * dr + dg * dg + db * db
    end

    local minDistance = targetRange * targetRange
    local targetX, targetY

    for y = startY, screenHeight do
        for x = startX, screenWidth do
            local pixelColor = ReadColor(screen, x, y)
            local distance = colorDistance(pixelColor, targetColor)
            if distance < minDistance then
                minDistance = distance
                targetX, targetY = x, y
            end
        end
    end

    return targetX, targetY
end

function OnEvent(event, arg)
    -- تشغيل/إيقاف تفعيل أحداث زر الماوس الأساسي (النقر الأيسر) عند بدء أو انتهاء تشغيل السكريبت
    if event == "PROFILE_ACTIVATED" then
        EnablePrimaryMouseButtonEvents(true)
    elseif event == "PROFILE_DEACTIVATED" then
        ReleaseMouseButton(2)
    end

    -- التحكم في تشغيل/إيقاف الارتداد
    if (event == "MOUSE_BUTTON_PRESSED" and arg == switch)
    or (event == "G_PRESSED" and arg == g_switch) then
        recoilOn = not recoilOn
        PressAndReleaseKey("scrolllock")
        step_recoil = 1
    end

    -- زيادة سرعة الإطلاق
    if (event == "MOUSE_BUTTON_PRESSED" and arg == plus)
    or (event == "G_PRESSED" and arg == g_plus) then
        step_recoil = step_recoil + 1
        OutputLogMessage("Speed Increased: %d\n", step_recoil)
        OutputLCDMessage("Speed: %d", step_recoil)
    end

    -- تقليل سرعة الإطلاق
    if (event == "MOUSE_BUTTON_PRESSED" and arg == minus)
    or (event == "G_PRESSED" and arg == g_minus) then
        if step_recoil > 1 then
            step_recoil = step_recoil - 1
            OutputLogMessage("Speed Decreased: %d\n", step_recoil)
            OutputLCDMessage("Speed: %d", step_recoil)
        end
    end

    -- التحكم في الارتداد عند النقر بزر الماوس الأيسر
    if IsMouseButtonPressed(1) and recoilOn then
        while IsMouseButtonPressed(1) do
            Sleep(10)  -- يمكن تعديل هذه القيمة حسب حجم الارتداد في اللعبة
            MoveMouseRelative(0, step_recoil)
        end
    end

    -- وظيفة التصويب التلقائي
    if event == "MOUSE_BUTTON_PRESSED" and arg == 1 then
        local screenX, screenY = GetMousePosition()
        local screen = ReadImage(GetCurrentImage())
        local targetX, targetY = FindTarget(screen, screenX, screenY, targetColor, targetRange)
        if targetX and targetY then
            AutoAim(targetX, targetY)
        end
    end

    -- التحكم في الاهتزاز الأفقي أثناء إطلاق النار
    if IsMouseButtonPressed(1) and aimBotOn then
        for i = 1, horizontalVibration do
            MoveMouseRelative(-1, 0)  -- حرك الماوس لليسار
            Sleep(5)
            MoveMouseRelative(1, 0)  -- حرك الماوس لليمين
            Sleep(5)
        end
    end
end

<!---
ccivack/ccivack is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
