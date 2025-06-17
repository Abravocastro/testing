Add-Type -AssemblyName System.DirectoryServices.AccountManagement
# Lista de cuentas: dominio, usuario, contraseña antigua, nueva
$cuentas = Import-Csv "usuarios.csv"

foreach ($cuenta in $cuentas) {
    $contextType = [System.DirectoryServices.AccountManagement.ContextType]::Domain

    # Check for empty domain or user
    if ([string]::IsNullOrWhiteSpace($cuenta.Dominio) -or [string]::IsNullOrWhiteSpace($cuenta.Usuario)) {
        Write-Warning "Dominio o usuario vacío para una de las cuentas."
        continue
    }

    try {
        $context = New-Object System.DirectoryServices.AccountManagement.PrincipalContext($contextType, $cuenta.Dominio)
        $usuario = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($context, $cuenta.Usuario)

        if ($usuario -ne $null) {
            $context.ValidateCredentials($cuenta.Usuario, $cuenta.Antigua) | Out-Null
            $usuario.ChangePassword($cuenta.Antigua, $cuenta.Nueva)
            $usuario.Save()
            Write-Output "Contraseña cambiada para $($cuenta.Usuario) en $($cuenta.Dominio)"
        }
        else {
            Write-Warning "Usuario no encontrado en $($cuenta.Dominio): $($cuenta.Usuario)"
        }
    }
    catch {
        Write-Warning "Fallo al cambiar contraseña de $($cuenta.Usuario) en $($cuenta.Dominio): $_"
    }
}
