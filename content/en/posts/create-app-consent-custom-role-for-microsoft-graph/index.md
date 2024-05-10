---
translationKey: create-app-consent-custom-role-for-microsoft-graph
slug: ""
title: Create app consent custom role for Microsoft Graph
subtitle: How to securely delegate admin consent for Microsoft Graph API permissions
description: "How to implement a custom role in Microsoft Entra to delegate admin consent for Microsoft Graph API permissions."
date: 2024-05-06T14:31:14.019Z
lastmod: 2024-05-10T11:14:00.05Z
preview: featured-image.jpg
draft: false
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
type: posts
---

## About app consent for Microsoft Graph

[App consent in Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity-platform/application-consent-experience) is a crucial part of maintaining security and privacy within your Microsoft 365 environment. It allows applications to access specific resources, like user data or groups, ensuring that they only have access to the data they need.

Typically, the roles of [Cloud Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) or [Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) are responsible for managing these app consent requests, as they have the necessary permissions to approve or deny access to these resources.

However, for [first-party applications](https://learn.microsoft.com/en-us/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in) like Microsoft Graph API and Azure AD Graph API, higher privileges ([Global Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#global-administrator) or [Privileged Role Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#privileged-role-administrator)) are required due to their extensive access capabilities.

## Using least privileges for app consent to Microsoft Graph API

This presents a challenge when it comes to the [principle of least privilege](https://learn.microsoft.com/en-us/entra/identity-platform/secure-least-privileged-access), a security principle dictating that a user should be given the minimum levels of access necessary to complete their job functions. While we want to give consent for specific app roles, we don't want to provide extensive access and control across all Microsoft 365 services.

One solution to this challenge is the use of [custom roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-create) with consent policies. A custom role allows you to define a set of permissions tailored to the specific needs of your operations teams and support organization.

However, implementing a custom role comes with its own set of challenges, including dealing with [Permission Grant Policies](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/manage-app-consent-policies) and keeping up-to-date with the latest changes in Microsoft Entra's permission model.

Despite these challenges, a custom role can provide a way to balance the need for security with the need for usability. It allows you to give your teams the access they need while still adhering to the principle of least privilege.

## Implementing the "Privileged Application Consent Administrator" role

This PowerShell script creates a custom role in Microsoft Entra that grants the ability to consent for delegated permissions and application permissions, including most application permissions for Microsoft Graph, except for a few sensitive permissions.

Azure AD Graph permissions are explicitly excluded.

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

#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Identity.SignIns'; RequiredVersion = '2.0.0' }
#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Identity.Governance'; RequiredVersion = '2.0.0' }
#Requires -Modules @{ ModuleName = 'Microsoft.Graph.Applications'; RequiredVersion = '2.0.0' }

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

The behavior is based on what the [Cloud Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) or [Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) roles allow, but adding Microsoft Graph together with a blacklist of permissions we don't want.

You may change the list of excluded app roles based on your requirements. The list essentially includes app roles that give write permissions with potential damage or uncontrolled leak of information.

For a list of possible app roles, read the [Microsoft Graph permissions reference](https://learn.microsoft.com/en-us/graph/permissions-reference), or have a look to all app roles that Microsoft Graph uses and filter by their names:

````powershell
# Read service principal details
$MSGraphSP = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Search for app roles with the key word 'Write' in it
$MSGraphSP.AppRoles | Where-Object { $_.Value -like '*Write*' } | Format-Table Value, Description
`````
