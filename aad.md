# ORGA App Onboarding - Azure Active Directory


## Checkliste

- Admin Consent durchführen
- Azure Active Directory Gruppen anlegen
  - Technical Admins: Führen technische Konfigurationen wie z.B. Azure Ad, SMTP etc. durch
  - Admins: Führen Inhaltliche KOnfigurationen wie z.B. Mandanten Einstellungen durch, sehen alle Funktionsbereiche und Rollen
  - Benutzer: alle Benutzer
- Optional: orgarena Benutzer anlegen

## Nützliche Links

Diese Links stellen Informationen zum Mandanten bereit. "common" durch den Mandanten ersetzen. Z.B. "orga-app.de".

https://login.microsoftonline.com/common/.well-known/openid-configuration
https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration
https://orgarena.de/apps.html

## Admin Consent durchführen

Die API benötigt Zugriff auf das Azure Ad. Mit dem Admin Consent wird dieser Zugriff gewährt.

https://login.microsoftonline.com/common/adminconsent?client_id=77991b59-7577-4292-b4ec-bdf115bb872e&response_mode=query&redirect_uri=https%3A%2F%2Forga-app-api-prod.azurewebsites.net%2Fazuread%2Fcallback

Wird der Admin Consent durch einen external Guest User druchgeführt, muss "common" durch den Mandanten ersetzt (z.B. "orga-app.de") werden. 

Dies ist zum Beispiel der Fall, wenn ein externer Dienstleister involviert ist, und dieser sich mit seinem persönlichem Login aus dem Home-Directory (z.B. user@softwarepioniere.de) im AD des Kunden (z.B. orga-app.de) anmeldet.


## Azure Active Directory Gruppen anlegen

Die Azure Ad Gruppen können mit diesem Powershell Script in der AZ CLI angelegt werden. 
Das Skript kann entweder auf dem lokalen Rechner oder in der Azure Cloud Shell ausgeführt werden. 

```powershell

# Gruppen anlegen und aktuellen Benutzer in die TecAdmin Gruppe einfügen

$currentUserId = (az ad signed-in-user show --query 'objectId' --output tsv)
$adminGroupId = (az ad group create --display-name 'ORGA App Admins' --mail-nickname 'orga-app-admins' --query 'objectId' --output tsv)
$benutzerGroupId = (az ad group create --display-name 'ORGA App Benutzer' --mail-nickname 'orga-app-benutzer' --query 'objectId' --output tsv)
$tecadminGroupId = (az ad group create --display-name 'ORGA App Tech Admins' --mail-nickname 'orga-app-tec-admins' --query 'objectId' --output tsv)
az ad group member add --group $tecadminGroupId --member-id $currentUserId
$tenantId = (az account show --query 'tenantId' --output tsv)
$result = @{
   tenant_id = $tenantId
   user_group_id = $benutzerGroupId
   admin_group_id = $adminGroupId
   technical_admin_group_id = $tecadminGroupId
}

# Ausgabe der Informationen als JSON zum Update der ORGA App AzureAd Config
ConvertTo-Json -InputObject $result

# Ausgabe in die Console
Write-Host "Azure Ad TenantId: $tenantId`nORGA App Admins: $adminGroupId `nORGA App Benutzer: $benutzerGroupId `nORGA-App Tech Admins $tecadminGroupId"

```

Hier ist eine Variante mit PowerShell Modulen

```powershell

# vorrausetzung: AzureAd PS-Modul installiert
# Install-Module AzureAD
Connect-AzureAD -Confirm -TenantId 'XXX'

$adminGroup= New-AzureADGroup -DisplayName 'ORGA App Admins' -MailNickName 'orga-app-admins' -MailEnabled $false -SecurityEnabled $true
$benutzerGroup= New-AzureADGroup -DisplayName 'ORGA App Benutzer' -MailNickName 'orga-app-benutzer' -MailEnabled $false -SecurityEnabled $true
$tecadminGroup= New-AzureADGroup -DisplayName 'ORGA App Tech Admins' -MailNickName 'orga-app-tec-admins' -MailEnabled $false -SecurityEnabled $true

$currentUserName = (Get-AzureADCurrentSessionInfo).Account.Id
$currentUserId = (Get-AzureADUser -Filter "UserPrincipalName eq '$currentUserName'").ObjectId
Add-AzureADGroupMember -ObjectId $tecadminGroup.ObjectId -RefObjectId $currentUserId

$tenant = Get-AzureADTenantDetail

$result = @{
   tenant_id = $tenant.ObjectId
   user_group_id = $benutzerGroup.ObjectId
   admin_group_id = $adminGroup.ObjectId
   technical_admin_group_id = $tecadminGroup.ObjectId
}

# Ausgabe der Informationen als JSON zum Update der ORGA App AzureAd Config
ConvertTo-Json -InputObject $result

```

```powershell

# login bei lokaler ausführung
az account clear
az login --allow-no-subscriptions --tenant xxx.de

```


Sollen mehrere Mandanten in der ORGA App auf das gleiche Azure AD zugreifen, können verschiedene Gruppen angelegt werden. Das folgende Skript stellt eine Postfix Variable für den Mandanten bereit.

```powershell

# Variante für mehrere ORGA App Mandanten  im Azure Ad Tenant

$postfix = 'DEMO 1'
$postfix_nick = 'demo1'
# Gruppen anlegen und aktuellen Benutzer in die TecAdmin Gruppe einfügen

$currentUserId = (az ad signed-in-user show --query 'objectId' --output tsv)
$adminGroupId = (az ad group create --display-name "ORGA App Admins $postfix" --mail-nickname "orga-app-admins-$postfix_nick" --query  'objectId' --output tsv)
$benutzerGroupId = (az ad group create --display-name "ORGA App Benutzer $postfix" --mail-nickname "orga-app-benutzer-$postfix_nick" --query 'objectId' --output tsv)
$tecadminGroupId = (az ad group create --display-name "ORGA App Tech Admins $postfix" --mail-nickname "orga-app-tec-admins-$postfix_nick" --query 'objectId' --output tsv)
az ad group member add --group $tecadminGroupId --member-id $currentUserId
$tenantId = (az account show --query 'tenantId' --output tsv)
$result = @{
   tenant_id = $tenantId
   user_group_id = $benutzerGroupId
   admin_group_id = $adminGroupId
   technical_admin_group_id = $tecadminGroupId
}

# Ausgabe der Informationen als JSON zum Update der ORGA App AzureAd Config
ConvertTo-Json -InputObject $result

# Ausgabe in die Console
Write-Host "Azure Ad TenantId: $tenantId`nORGA App Admins: $adminGroupId `nORGA App Benutzer: $benutzerGroupId `nORGA-App Tech Admins $tecadminGroupId"

```

