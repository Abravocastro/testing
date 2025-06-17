# Lista de cuentas: dominio, usuario, contraseña antigua, nueva
$cuentas = @(
    @{ Dominio = "DOMINIO1"; Usuario = "usuario1"; Antigua = "ViejaPass1"; Nueva = "NuevaPass1!" },
    @{ Dominio = "DOMINIO2"; Usuario = "usuario2"; Antigua = "ViejaPass2"; Nueva = "NuevaPass2!" }
)

foreach ($cuenta in $cuentas) {
    $contextType = [System.DirectoryServices.AccountManagement.ContextType]::Domain
    $context = New-Object System.DirectoryServices.AccountManagement.PrincipalContext $contextType, $cuenta.Dominio

    try {
        $usuario = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($context, $cuenta.Usuario)

        if ($usuario -ne $null) {
            $context.ValidateCredentials($cuenta.Usuario, $cuenta.Antigua) | Out-Null
            $usuario.ChangePassword($cuenta.Antigua, $cuenta.Nueva)
            $usuario.Save()
            Write-Output "Contraseña cambiada para $($cuenta.Usuario) en $($cuenta.Dominio)"
        } else {
            Write-Warning "Usuario no encontrado en $($cuenta.Dominio): $($cuenta.Usuario)"
        }
    } catch {
        Write-Warning "Fallo al cambiar contraseña de $($cuenta.Usuario) en $($cuenta.Dominio): $_"
    }
}

---
La máquina donde ejecutas el script debe tener conectividad con los DC de los dominios.
El script debe ejecutarse como el propio usuario autenticado o deberás hacer que cada usuario lo ejecute.
Se requiere .NET Framework ≥ 3.5 (ya incluido en la mayoría de sistemas Windows).

Dominio,Usuario,Antigua,Nueva
DOMINIO1,usuario1,ViejaPass1,NuevaPass1!
DOMINIO2,usuario2,ViejaPass2,NuevaPass2!

$cuentas = Import-Csv "usuarios.csv"
