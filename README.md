# MF0490-Active-Directory – Laboratorio Active Directory

**Alumno:** Volodymyr Osebchuk

## Nota sobre el entorno

La máquina virtual Windows Server 2019 (DC01) se bloquea a los pocos minutos de arrancar en VirtualBox (problema compartido por la mayoría del grupo y comentado con el profesorado), aunque se probó a aumentar memoria y núcleos de CPU. Por ello **no he podido ejecutar los comandos ni obtener capturas reales**.

Los valores de este documento son los **esperados**, deducidos de los datos del laboratorio (dominio `corp.local`, equipo `DC01`, IP `192.168.60.5`, DNS `127.0.0.1`) y de la configuración por defecto de AD DS. Todo lo que no se puede deducir está marcado con ⚠ y se completará cuando la VM funcione.

---

## [AD 01] Reconocer el entorno

Ruta: Administrador del servidor → Herramientas → Usuarios y equipos de Active Directory (`dsa.msc`).

Estructura esperada bajo `corp.local`:

| Elemento | Tipo |
|---|---|
| Builtin | Contenedor |
| Computers | Contenedor predeterminado |
| Domain Controllers | **Unidad organizativa (OU)** – contiene DC01 |
| ForeignSecurityPrincipals | Contenedor |
| Managed Service Accounts | Contenedor |
| Users | Contenedor predeterminado (usuario Administrator, grupos Domain Admins, Domain Users…) |

Checks de validación:
- Dominio: `corp.local`.
- Consola ADUC localizada en el menú Herramientas.
- Objeto de usuario: `Administrator` (contenedor Users).
- Controladores de dominio: OU *Domain Controllers* (contiene DC01).
- Diferencia: un **usuario** es una cuenta que se identifica y accede; un **grupo** reúne objetos para asignar permisos de forma conjunta; una **OU** es un contenedor para organizar objetos y aplicar directivas o delegar administración.

---

## [AD 02] Reconocimiento de la arquitectura

Comandos a ejecutar (solo lectura): `Get-ADDomain`, `Get-ADForest`, `Get-ADDomainController -Filter *`, `ipconfig /all`.

### Registro de resultados (valores esperados)

| Dato | Valor |
|---|---|
| Nombre del dominio | corp.local (NetBIOS: CORP) |
| Nombre del bosque | corp.local |
| Nombre del controlador de dominio | DC01 (DC01.corp.local) |
| Dirección IPv4 del servidor | 192.168.60.5 |
| Servidor DNS configurado | 127.0.0.1 (el propio servidor) |
| Nombre del sitio de AD | Default-First-Site-Name |
| Número de controladores de dominio | 1 |

### Cuestiones para reflexionar

1. **¿Hace falta un dominio por oficina?** No. Un dominio puede abarcar varias oficinas; las ubicaciones físicas se representan con sitios y subredes.
2. **¿Qué organiza los usuarios por departamentos?** Las unidades organizativas (OU).
3. **¿Qué representa las ubicaciones de red?** Los sitios (Sites) junto con las subredes.
4. **¿Función de un segundo controlador de dominio?** Redundancia y tolerancia a fallos, reparto de carga de autenticación y copia del directorio mediante replicación.
5. **¿Por qué es importante el DNS?** Los equipos localizan los controladores de dominio y servicios (Kerberos, LDAP) mediante registros SRV en DNS; sin un DNS adecuado no pueden unirse al dominio ni autenticarse.

---

## [AD 03] Exploración de las herramientas administrativas

| Consola | Comando | Para qué sirve |
|---|---|---|
| Usuarios y equipos de AD | `dsa.msc` | Usuarios, grupos, equipos y OU |
| Centro de administración de AD | `dsac.exe` | Administración con historial de PowerShell |
| Sitios y servicios de AD | `dssite.msc` | Estructura física |
| Dominios y confianzas | `domain.msc` | Relaciones de confianza |
| Directivas de grupo | `gpmc.msc` | GPO |
| Administrador DNS | `dnsmgmt.msc` | Zonas y registros DNS |
| Visor de eventos | `eventvwr.msc` | Registros del sistema |

### Registro de resultados

| Dato | Valor |
|---|---|
| Nombre DNS del dominio | corp.local |
| Nombre del controlador de dominio | DC01 |
| Número de controladores identificados | 1 |
| Una OU existente | Domain Controllers (`OU=Domain Controllers,DC=corp,DC=local`) |
| Una cuenta de usuario consultada | Administrator |
| Una GPO localizada | Default Domain Policy (también Default Domain Controllers Policy) |
| Un evento del registro Sistema (ID y origen) | ⚠ pendiente de la VM (típico: ID 7036, origen Service Control Manager) |
| ¿Disponible el módulo ActiveDirectory? | Sí (se instala con AD DS y sus herramientas de administración) |

---

## [AD 04] Caso práctico

| # | Solicitud | Herramienta | Por qué |
|---|---|---|---|
| 1 | Grupos de un usuario | Usuarios y equipos de AD (pestaña *Miembro de*) o `Get-ADPrincipalGroupMembership` | Muestra directamente las pertenencias del objeto usuario |
| 2 | Localizar un DC en un sitio | Sitios y servicios de AD | Es la consola de la estructura física: sitio → *Servers* |
| 3 | Revisar una directiva de seguridad | Administración de directivas de grupo (GPMC) | Ahí se consultan y enlazan las GPO |
| 4 | Registros de error de Windows Server | Visor de eventos | Centraliza los registros Sistema, Aplicación y Seguridad |
| 5 | Lista de equipos del directorio | Usuarios y equipos de AD (*Computers*) o `Get-ADComputer -Filter *` | Los equipos son objetos del directorio |
| 6 | Registros DNS del dominio | Administrador DNS | Permite ver zonas y registros (A, SRV…) |

---

## [AD 05] Ciclo de vida de una cuenta de dominio

> No ejecutado (VM inoperativa). Se documenta el procedimiento previsto. ⚠ `XX` = identificador que asigne el profesor.

Datos de trabajo: Julia Torres – N3, `lab.n3.XX`, UPN `lab.n3.XX@corp.local`, Departamento *Administracion*, Cargo *Auxiliar administrativo*, Descripción *Practica formativa - Nivel 3*.

Procedimiento previsto:
1. En `dsa.msc`, buscar `lab.n3.XX` para comprobar que no existe.
2. Nuevo → Usuario en la OU/contenedor autorizado, con contraseña temporal, cambio obligatorio en el siguiente inicio y cuenta **deshabilitada**.
3. Rellenar Departamento, Cargo y Descripción; verificar con:
   ```powershell
   $usuario = 'lab.n3.XX'
   Get-ADUser -Identity $usuario -Properties Department,Title,Description,Enabled |
     Select-Object Name,SamAccountName,Department,Title,Description,Enabled
   ```
4. Habilitar (con autorización del profesor) y comprobar `Enabled = True`; restablecer contraseña; deshabilitar y comprobar `Enabled = False`.
5. Comprobación final:
   ```powershell
   Get-ADUser -Identity $usuario -Properties UserPrincipalName,DistinguishedName,Enabled |
     Select-Object Name,SamAccountName,UserPrincipalName,DistinguishedName,Enabled
   ```

### Ficha de evidencias

| Campo | Valor |
|---|---|
| Dominio consultado | corp.local |
| Controlador de dominio utilizado | DC01 |
| sAMAccountName | lab.n3.XX ⚠ |
| UPN | lab.n3.XX@corp.local |
| DN | ⚠ depende de la OU indicada (p. ej. `CN=Julia Torres - N3,CN=Users,DC=corp,DC=local`) |
| Departamento y cargo | Administracion / Auxiliar administrativo |
| Estado inicial | Enabled = False |
| Estado tras la habilitación | Enabled = True |
| ¿Se restableció la contraseña? | Previsto: Sí, sin indicar su valor |
| Estado final | Enabled = False |
| Problemas encontrados | La VM Windows Server se bloquea a los pocos minutos de arrancar; no se pudo ejecutar el laboratorio |
