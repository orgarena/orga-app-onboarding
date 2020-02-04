# ORGA App Onboarding

Onboarding für die ORGA App

## Azure Ad Links

Diese Links stellen Informationen zum Mandanten bereit. "common" durch den Mandanten ersetzen. Z.B. "orga-app.de".

https://login.microsoftonline.com/common/.well-known/openid-configuration
https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration


## Admin Consent durchführen

Die API benötigt Zugriff auf das Azure Ad. Mit dem Admin Consent wird dieser Zugriff gewährt. "common" durch den Mandanten ersetzen. Z.B. "orga-app.de". Dann den Link im Browser aufrufen.

https://login.microsoftonline.com/common/adminconsent?client_id=8ab20a21-5fa0-4d89-8392-125469e74cc1&response_mode=query&redirect_uri=https%3A%2F%2Forga-app-api-prod.azurewebsites.net%2Fazuread%2Fcallback

## Azure Active Directory Gruppen anlegen

Die Azure Ad Gruppen können mit diesem Script über die AZ CLI angelegt werden. Dies kann entweder auf dem lokalen Rechner oder in der Azure Cloud Shell ausgeführt werden. 

```powershell

# login bei lokaler ausführung
az login --allow-no-subscriptions

```

```powershell

# Gruppen anlegen und aktuellen Benutzer in die TecAdmin Gruppe einfügen

$currentUserId = (az ad signed-in-user show --query 'objectId' --output tsv)
$adminGroupId = (az ad group create --display-name 'ORGA App Admins' --query 'objectId' --output tsv)
$benutzerGroupId = (az ad group create --display-name 'ORGA App Benutzer' --query 'objectId' --output tsv)
$tecadminGroupId = (az ad group create --display-name 'ORGA App Tech Admins' --query 'objectId' --output tsv)
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
