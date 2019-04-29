# <a name="known-issues-for-insider-builds"></a>Problemas conocidos de las compilaciones de insider

## <a name="build-16237"></a>Compilación 16237

- Aislamiento de Hyper-V no está funcionando correctamente. Esta solución es necesario para usar el aislamiento de Hyper-V en la compilación 16237. Ejecuta estos comandos en PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server ahora se ejecuta como usuario, por lo que se producirá un error en los comandos que requieran privilegios de administrador. La inclusión de una línea como "RUN setx /M PATH" provocará errores en la compilación. Para este escenario, puedes usar esta alternativa:

```dockerfile
RUN setx PATH <path>
```
