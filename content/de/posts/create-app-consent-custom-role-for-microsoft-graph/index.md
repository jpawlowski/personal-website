---
translationKey: create-app-consent-custom-role-for-microsoft-graph
slug: benutzerdefinierte-app-zustimmung-fur-microsoft-graph-erstellen
title: Benutzerdefinierte App-Zustimmung für Microsoft Graph erstellen
subtitle: So lässt sich die Admin-Zustimmung für Microsoft Graph API-Berechtigungen sicher delegieren
description: Wie man eine benutzerdefinierte Rolle in Microsoft Entra implementiert, um die Admin-Zustimmung für Microsoft Graph API-Berechtigungen zu delegieren.
date: 2024-05-06T14:31:14.019Z
draft: false
type: posts
lastmod: 2024-05-10T11:14:00.0Z
preview: featured-image.jpg
tags:
  - Admin Consent
  - Development
  - IAM
  - RBAC
  - Security
categories:
  - Microsoft Entra
  - Cyber Security
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
---

## Über die Zustimmung zu Anwendungen für Microsoft Graph

Die [App-Zustimmung in Microsoft Entra](https://learn.microsoft.com/de-de/entra/identity-platform/application-consent-experience) ist ein entscheidender Bestandteil der Aufrechterhaltung von Sicherheit und Datenschutz in jeder Microsoft 365-Umgebung. Sie ermöglicht Anwendungen den Zugriff auf bestimmte Ressourcen, wie Benutzerdaten oder Gruppen, und stellt sicher, dass sie nur auf die Daten zugreifen können, die sie benötigen.

Normalerweise sind die Rollen [Cloudanwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) oder [Anwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#application-administrator) für die Verwaltung dieser App-Zustimmungsanfragen verantwortlich, da sie über die notwendigen Berechtigungen verfügen, um den Zugriff auf diese Ressourcen zu genehmigen oder zu verweigern.

Für [Erstanbieteranwendungen](https://learn.microsoft.com/de-de/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in) wie Microsoft Graph API und Azure AD Graph API sind jedoch aufgrund ihrer umfassenden Zugriffsmöglichkeiten höhere Berechtigungen ([Globaler Administrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#global-administrator) oder [Administrator für privilegierte Rollen](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#privileged-role-administrator)) erforderlich.

## Verwendung der geringsten Berechtigungen für die Zustimmung der App zur Microsoft Graph API

Dies stellt eine Herausforderung dar, wenn es um das [Prinzip der geringsten Rechte](https://learn.microsoft.com/de-de/entra/identity-platform/secure-least-privileged-access) geht, ein Sicherheitsprinzip, das vorschreibt, dass einem Benutzer nur die minimalen Zugriffsrechte gewährt werden sollten, die für die Erfüllung seiner Aufgaben erforderlich sind. Während wir die Zustimmung für bestimmte App-Rollen erteilen möchten, wollen wir keinen umfassenden Zugriff und keine umfassende Kontrolle über alle Microsoft 365-Dienste gewähren.

Eine Lösung für diese Herausforderung ist die Verwendung von [benutzerdefinierten Rollen](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/custom-create) mit Zustimmungsrichtlinien. Eine benutzerdefinierte Rolle ermöglicht es Ihnen, eine Reihe von Berechtigungen zu definieren, die auf die spezifischen Anforderungen Ihrer Betriebsteams und Supportorganisation zugeschnitten sind.

Die Implementierung einer benutzerdefinierten Rolle bringt jedoch eine Reihe von Herausforderungen mit sich, einschließlich des Umgangs mit [Zustimmungsrichtlinien](https://learn.microsoft.com/de-de/entra/identity/enterprise-apps/manage-app-consent-policies) und der Anpassung an die neuesten Änderungen im Berechtigungsmodell von Microsoft Entra.

Trotz dieser Herausforderungen kann eine benutzerdefinierte Rolle einen Weg bieten, um das Bedürfnis nach Sicherheit mit dem Bedürfnis nach Benutzerfreundlichkeit in Einklang zu bringen. Sie ermöglicht es, den Teams den Zugriff zu gewähren, den sie benötigen, und gleichzeitig das Prinzip der geringsten Privilegien einzuhalten.

## Implementierung der Rolle "Privilegierte Anwendungen Zustimmungs-Administrator"

Dieses PowerShell-Skript erstellt eine benutzerdefinierte Rolle in Microsoft Entra, die die Fähigkeit zur Zustimmung für delegierte Berechtigungen und Anwendungsberechtigungen, einschließlich der meisten Anwendungsberechtigungen für Microsoft Graph—mit Ausnahme einiger sensibler Berechtigungen—gewährt.

Azure AD Graph-Berechtigungen sind explizit ausgeschlossen.

````powershell
<#PSScriptInfo
.VERSION 1.0.1
.GUID a17ec91c-0f75-42ab-b4ef-8766c1a25fca
.AUTHOR Julian Pawlowski
.COMPANYNAME Julian Pawlowski
.COPYRIGHT © 2024 Julian Pawlowski
.TAGS
.LICENSEURI
.PROJECTURI https://gist.github.com/jpawlowski/ca1bde7e979f367e8007b056bc032b6e
.ICONURI
.EXTERNALMODULEDEPENDENCIES 'Microsoft.Graph.Identity.SignIns','Microsoft.Graph.Identity.Governance','Microsoft.Graph.Applications'
.REQUIREDSCRIPTS
.EXTERNALSCRIPTDEPENDENCIES
.RELEASENOTES
    Version 1.0.1 (2024-05-10)
    - Add module dependencies.

    Version 1.0.0 (2024-05-03)
    - Initial release.
#>

<#
.SYNOPSIS
    Create a custom role in Microsoft Entra that grants the ability to consent for delegated permissions and application permissions, including most application permissions for Microsoft Graph, except for a few sensitive permissions. Azure AD Graph permissions are NOT included.  

.DESCRIPTION
    This script creates a custom role in Microsoft Entra that grants the ability to consent for delegated permissions and application permissions, including most application permissions for Microsoft Graph, except for a few sensitive permissions. Azure AD Graph permissions are NOT included. Note that to approve Microsoft Graph application permissions, Microsoft Entra roles Cloud Application Administrator and Application Administrator MUST NOT be active as otherwise, their exclusion policy will take precedence.
#>

#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Identity.SignIns'; ModuleVersion = '2.0.0' }
#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Identity.Governance'; ModuleVersion = '2.0.0' }
#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Applications'; ModuleVersion = '2.0.0' }

Connect-MgGraph -ContextScope Process -Scopes @(
    'Application.Read.All',
    'DelegatedPermissionGrant.Read.All',
    'RoleManagement.ReadWrite.Directory',
    'Policy.ReadWrite.PermissionGrant'
)

#region Create a new permission grant policy
$PermissionGrantPolicy = New-MgPolicyPermissionGrantPolicy -Id 'custom-appconsent-admin' -Description 'Permissions consentable by Privileged Application Consent Administrators.' -DisplayName 'Custom App Consent Admin Policy' -ErrorAction Stop

# Include all delegated and application permissions
New-MgPolicyPermissionGrantPolicyInclude -PermissionGrantPolicyId $PermissionGrantPolicy.Id -PermissionType 'delegated' -PermissionClassification 'all' -ClientApplicationsFromVerifiedPublisherOnly:$false
New-MgPolicyPermissionGrantPolicyInclude -PermissionGrantPolicyId $PermissionGrantPolicy.Id -PermissionType 'application' -PermissionClassification 'all' -ClientApplicationsFromVerifiedPublisherOnly:$false

# Exclude all Azure Graph application permissions
New-MgPolicyPermissionGrantPolicyExclude -PermissionGrantPolicyId $PermissionGrantPolicy.Id -ResourceApplication '00000002-0000-0000-c000-000000000000' -PermissionType 'application'
#endregion

#region Exclude specific application permissions from Microsoft Graph
$ServicePrincipal = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'" -ErrorAction Stop
$ExcludedAppRoleValues = @(
    'AdministrativeUnit.ReadWrite.All'
    'Application.ReadWrite.All'
    'AppRoleAssignment.ReadWrite.All'
    'AuthenticationContext.ReadWrite.All'
    'BillingConfiguration.ReadWrite.All'
    'ConsentRequest.ReadWrite.All'
    'CrossTenantUserProfileSharing.ReadWrite.All'
    'CustomAuthenticationExtension.ReadWrite.All'
    'DelegatedAdminRelationship.ReadWrite.All'
    'DelegatedPermissionGrant.ReadWrite.All'
    'Device.ReadWrite.All'
    'DeviceManagementApps.ReadWrite.All'
    'DeviceManagementConfiguration.ReadWrite.All'
    'DeviceManagementManagedDevices.PrivilegedOperations.All'
    'DeviceManagementManagedDevices.ReadWrite.All'
    'DeviceManagementRBAC.ReadWrite.All'
    'DeviceManagementServiceConfig.ReadWrite.All'
    'Directory.ReadWrite.All'
    'DirectoryRecommendations.ReadWrite.All'
    'Domain.ReadWrite.All'
    'eDiscovery.ReadWrite.All'
    'ExternalConnection.ReadWrite.All'
    'ExternalConnection.ReadWrite.OwnedBy'
    'Files.ReadWrite.All'
    'Group.ReadWrite.All'
    'GroupMember.ReadWrite.All'
    'IdentityProvider.ReadWrite.All'
    'IdentityRiskEvent.ReadWrite.All'
    'IdentityRiskyServicePrincipal.ReadWrite.All'
    'IdentityRiskyUser.ReadWrite.All'
    'MultiTenantOrganization.ReadWrite.All'
    'OnPremDirectorySynchronization.ReadWrite.All'
    'OnPremisesPublishingProfiles.ReadWrite.All'
    'Organization.ReadWrite.All'
    'OrganizationalBranding.ReadWrite.All'
    'Policy.ReadWrite.AccessReview'
    'Policy.ReadWrite.ApplicationConfiguration'
    'Policy.ReadWrite.AuthenticationFlows'
    'Policy.ReadWrite.AuthenticationMethod'
    'Policy.ReadWrite.Authorization'
    'Policy.ReadWrite.ConditionalAccess'
    'Policy.ReadWrite.ConsentRequest'
    'Policy.ReadWrite.CrossTenantAccess'
    'Policy.ReadWrite.ExternalIdentities'
    'Policy.ReadWrite.FeatureRollout'
    'Policy.ReadWrite.FedTokenValidation'
    'Policy.ReadWrite.IdentityProtection'
    'Policy.ReadWrite.PermissionGrant'
    'Policy.ReadWrite.SecurityDefaults'
    'Policy.ReadWrite.TrustFramework'
    'PrivilegedAccess.ReadWrite.AzureAD'
    'PrivilegedAccess.ReadWrite.AzureADGroup'
    'PrivilegedAccess.ReadWrite.AzureResources'
    'PrivilegedAssignmentSchedule.ReadWrite.AzureADGroup'
    'PrivilegedEligibilitySchedule.ReadWrite.AzureADGroup'
    'PublicKeyInfrastructure.ReadWrite.All'
    'RoleAssignmentSchedule.ReadWrite.Directory'
    'RoleEligibilitySchedule.ReadWrite.Directory'
    'RoleManagement.ReadWrite.CloudPC'
    'RoleManagement.ReadWrite.Directory'
    'RoleManagement.ReadWrite.Exchange'
    'RoleManagementAlert.ReadWrite.Directory'
    'RoleManagementPolicy.ReadWrite.AzureADGroup'
    'RoleManagementPolicy.ReadWrite.Directory'
    'SharePointTenantSettings.ReadWrite.All'
    'Synchronization.ReadWrite.All'
    'TrustFrameworkKeySet.ReadWrite.All'
    'User.EnableDisableAccount.All'
    'User.Export.All'
    'User.Invite.All'
    'User.ManageIdentities.All'
    'User.ReadWrite.All'
    'UserAuthenticationMethod.ReadWrite.All'
)

# Convert app role values to app role IDs
$ExcludedPermissions = $ExcludedAppRoleValues | ForEach-Object {
    $item = $_
    $roles = $ServicePrincipal.AppRoles | Where-Object { $_.Value -eq $item }
    
    if ($roles) {
        $roles | ForEach-Object { $_.Id }
    }
    else {
        Write-Error "No matching app role found for value: $item"
    }
}

if ($ExcludedPermissions.Count -gt 100) {
    Throw "Excluded permissions count exceeds 100"
}
else {
    $params = @{
        PermissionType      = 'application'
        ResourceApplication = $ServicePrincipal.AppId
        Permissions         = [array]$ExcludedPermissions
        ErrorAction         = 'Stop'
    }
    New-MgPolicyPermissionGrantPolicyExclude -PermissionGrantPolicyId $PermissionGrantPolicy.Id -BodyParameter $params
}
#endregion

#region Create new custom admin role
$params = @{
    DisplayName     = 'Privileged Application Consent Administrator'
    Description     = 'This role grants the ability to consent for delegated permissions and application permissions, including most application permissions for Microsoft Graph, except for a few sensitive permissions. Azure AD Graph permissions are NOT included. Note that to approve Microsoft Graph application permissions, Microsoft Entra roles Cloud Application Administrator and Application Administrator MUST NOT be active as otherwise, their exclusion policy will take precedence.'
    TemplateId      = (New-Guid).Guid
    IsEnabled       = $true
    RolePermissions = @(
        @{
            AllowedResourceActions = @(
                'microsoft.directory/servicePrincipals/credentials/update'
                'microsoft.directory/servicePrincipals/managePermissionGrantsForAll.' + $PermissionGrantPolicy.Id
            )
        }
    )
    ErrorAction     = 'Stop'
}
$customAdmin = New-MgRoleManagementDirectoryRoleDefinition @params
#endregion
````

Das Verhalten basiert auf dem, was die Rollen [Cloud-Anwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) oder [Anwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#application-administrator) zulassen, wobei jedoch Microsoft Graph zusammen mit einer schwarzen Liste von Berechtigungen hinzugefügt wird, die wir nicht wünschen.

Die Liste der ausgeschlossenen Anwendungsrollen kann je nach Anforderung geändert werden. Sie enthält im Wesentlichen App-Rollen, die Schreibberechtigungen mit potenziellem Schaden oder unkontrolliertem Informationsabfluss erteilen.

Eine Liste möglicher App-Rollen findet sich in der [Microsoft Graph-Berechtigungsreferenz](https://learn.microsoft.com/de-de/graph/permissions-reference), oder über eine direkte Abfrage der Microsoft Graph API und eines Filters:

````powershell
# Details zum Dienstprinzipal lesen
$MSGraphSP = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Suche nach App-Rollen, die das Schlüsselwort 'Write' enthalten
$MSGraphSP.AppRoles | Where-Object { $_.Value -like '*Write*' } | Format-Table Value, Description
`````
