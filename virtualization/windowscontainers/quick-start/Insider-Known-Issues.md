# Problemas conocidos de las compilaciones de Insider

## Compilación 16237

- Los contenedores de Hyper-V no funcionan correctamente. Es necesario aplicar esta solución alternativa para usar contenedores de Hyper-V en la compilación 16237. Ejecuta estos comandos en PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server pasa a ejecutarse como usuario, por lo que se producirán errores en los comandos que requieran privilegios de administrador. La inclusión de una línea como "RUN setx /M PATH" provocará errores en la compilación. Para este escenario, puedes usar esta alternativa:

```dockerfile
RUN setx PATH <path>
```
