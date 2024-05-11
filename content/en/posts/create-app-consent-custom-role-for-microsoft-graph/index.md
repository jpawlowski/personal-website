---
translationKey: create-app-consent-custom-role-for-microsoft-graph
slug: ""
title: Create app consent custom role for Microsoft Graph
subtitle: How to securely delegate admin consent for Microsoft Graph API permissions
description: How to implement a custom role in Microsoft Entra to delegate admin consent for Microsoft Graph API permissions.
date: 2024-05-06T14:31:14.019Z
lastmod: 2024-05-10T11:14:00.05Z
preview: featured-image.jpg
comment: true
draft: false
tags:
  - Microsoft Graph API
  - Admin Consent
  - Custom Role
  - Least Privilege
  - Security
  - PowerShell
  - Azure AD
  - RBAC
  - IAM
categories:
  - Microsoft Entra
  - Cyber Security
  - Identity and Access Management
  - Role-Based Access Control
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
type: posts
---

## About App Consent for Microsoft Graph

[App consent in Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity-platform/application-consent-experience) is a crucial part of maintaining security and privacy within your Microsoft 365 environment. It allows applications to access specific resources, like user data or groups, ensuring that they only have access to the data they need.

Typically, the roles of [Cloud Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) or [Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) are responsible for managing these app consent requests, as they have the necessary permissions to approve or deny access to these resources.

However, for [first-party applications](https://learn.microsoft.com/en-us/troubleshoot/azure/entra/entra-id/governance/verify-first-party-apps-sign-in) like Microsoft Graph API and Azure AD Graph API, higher privileges ([Global Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#global-administrator) or [Privileged Role Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#privileged-role-administrator)) are required due to their extensive access capabilities.

## Using Least Privileges for App Consent to Microsoft Graph API

This presents a challenge when it comes to the [principle of least privilege](https://learn.microsoft.com/en-us/entra/identity-platform/secure-least-privileged-access), a security principle dictating that a user should be given the minimum levels of access necessary to complete their job functions. While we want to give consent for specific app roles, we don't want to provide extensive access and control across all Microsoft 365 services.

One solution to this challenge is the use of [custom roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-create) with consent policies. A custom role allows you to define a set of permissions tailored to the specific needs of your operations teams and support organization.

However, implementing a custom role comes with its own set of challenges, including dealing with [Permission Grant Policies](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/manage-app-consent-policies) and keeping up-to-date with the latest changes in Microsoft Entra's permission model.

Despite these challenges, a custom role can provide a way to balance the need for security with the need for usability. It allows you to give your teams the access they need while still adhering to the principle of least privilege.

## Implementing the "Privileged Application Consent Administrator" Role

This PowerShell script creates a custom role in Microsoft Entra that grants the ability to consent for delegated permissions and application permissions, including most application permissions for Microsoft Graph, except for a few sensitive permissions.

Azure AD Graph permissions are explicitly excluded.

Naturally, the source code can be found as a Gist snippet on GitHub:

{{< gist jpawlowski ca1bde7e979f367e8007b056bc032b6e >}}

The behavior is based on what the [Cloud Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) or [Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) roles allow, but adding Microsoft Graph together with a blacklist of permissions we don't want.

You may change the list of excluded app roles based on your requirements. The list essentially includes app roles that give write permissions with potential damage or uncontrolled leak of information.

For a list of possible app roles, read the [Microsoft Graph permissions reference](https://learn.microsoft.com/en-us/graph/permissions-reference), or have a look to all app roles that Microsoft Graph uses and filter by their names:

````powershell
# Read service principal details
$MSGraphSP = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Search for app roles with the key word 'Write' in it
$MSGraphSP.AppRoles | Where-Object { $_.Value -like '*Write*' } | Format-Table Value, Description
`````
