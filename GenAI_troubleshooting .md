

| # | Problem | Solution | Worked? |
|---|---------|----------|---------|
| 1 | `wsl --list --verbose` showed Ubuntu as Stopped | Ran `wsl -d Ubuntu` to launch Ubuntu directly | Yes |
| 2 | `wsl --start` invalid command | Used `wsl -d Ubuntu` instead | Yes |
| 3 | `sudo wsl -d Ubuntu` inside WSL — command not found | Already inside WSL — wsl not available inside itself | Understood |
| 4 | `New-Item` not recognised | Running in CMD not PowerShell — switched to winget | Yes |
| 5 | Run as Administrator not working | Used winget which requires no admin rights | Yes |
| 6 | `ErrImagePull` for `kamailio/kamailio:5.7.2-alpine` | Switched to `ghcr.io/kamailio/kamailio-ci:latest` | Yes |
| 7 | `ErrImageNeverPull` | Used `minikube image load` to transfer image into Minikube | Yes |
| 8 | `for /f` IP capture failed in Windows CMD | Retrieved IP manually from `kubectl get pods -o wide` | Yes |
| 9 | iperf3 client — Name or service not known | IPERF_IP variable was empty — used hardcoded IP | Yes |
| 10 | `kubectl top pods` — Metrics API not available | `minikube addons enable metrics-server`, waited 60s | Yes |


