name: 2022RDP
on: workflow_dispatch

jobs:
  build:

    runs-on: windows-2022
    timeout-minutes: 99999
  
    steps:

    - name: Tải Ngrok
      run: |
        Invoke-WebRequest https://github.com/Triducdinh/adminDucTri/raw/kaal/resources/ngrok.zip -OutFile ngrok.zip
        Invoke-WebRequest https://raw.githubusercontent.com/Triducdinh/ngrok-rdp/kaal/resources/start.bat -OutFile start.bat
        Invoke-WebRequest https://github.com/Triducdinh/AdminDucTri/blob/kaal/README.txt

    - name: Giải nén file
      run: Expand-Archive ngrok.zip

    - name: kết nối tài khoản ngrok
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}

    - name: bật rdp
      run: |
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        
    - name: tạo ip tcp
      run: Start-Process Powershell -ArgumentList '-Noexit -Command ".\ngrok\ngrok.exe --region ap tcp 3389"'

    - name: kết nối với rdp  [CPU 2 Core - 7GB Ram - 256 SSD]
      run: cmd /c start.bat

    - name: TimeCount
      run: |
      
        Invoke-WebRequest https://raw.githubusercontent.com/adtitas/ngrok-rdp/main/resources/loop.ps1 -OutFile loop.ps1
        ./loop.ps1
