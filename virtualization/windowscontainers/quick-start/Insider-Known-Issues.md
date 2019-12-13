# <a name="known-issues-for-insider-builds"></a>Problemas conocidos de las compilaciones de Insider

## <a name="build-16237"></a>Compilación 16237

- El aislamiento de Hyper-V no funciona correctamente. Esta solución es necesaria para usar el aislamiento de Hyper-V en la compilación 16237. Ejecuta estos comandos en PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server se ejecuta ahora como usuario, por lo que se producirá un error en los comandos que requieren privilegios de administrador. La inclusión de una línea como "RUN setx /M PATH" provocará errores en la compilación. Para este escenario, puedes usar esta alternativa:

```dockerfile
RUN setx PATH <path>
```
