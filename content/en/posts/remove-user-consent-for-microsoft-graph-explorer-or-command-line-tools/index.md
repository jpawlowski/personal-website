---
translationKey: remove-user-consent-for-microsoft-graph-explorer-or-command-line-tools
slug: ""
title: Remove user consent for Graph Explorer or Command Line Tools
subtitle: So widerrufst du die Microsoft Graph-Berechtigungen, die ein Benutzer erteilt hat
description: Erfahre, wie du mit einem praktischen PowerShell-Skript die Benutzerzustimmung für Microsoft Graph API-Zugriffsrechte verwalten kannst.
date: 2024-05-11T08:07:00.323Z
lastmod: 2024-05-11T08:07:00.323Z
preview: featured-image.jpg
draft: false
tags:
  - Microsoft Graph API
  - PowerShell
  - User Consent
  - Authorization
categories:
  - Microsoft 365
  - Microsoft Entra
  - Cyber Security
  - Scripting
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
type: posts
---

## Understanding User Consent for Microsoft Graph API Access

Microsoft provides several ways to interact with the Microsoft Graph API, as outlined in their [SDK overview](https://learn.microsoft.com/en-us/graph/sdks/sdks-overview). While many of these methods are designed for software developers, administrators often interact with the API through the [Microsoft Graph Command Line Tools](https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview) for scripting, or the [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer) for learning and investigating the API.

These tools use a method known as [user-delegated authorization](https://learn.microsoft.com/en-us/graph/security-authorization), where a cloud application provided by Microsoft acts as an intermediary between the user's account and the Microsoft Graph API. This method requires the user to grant permissions to the application to act on their behalf, separate from their assigned role permissions in Microsoft Entra.

## How to Revoke User-Delegated Authorization

Currently, the Microsoft Entra or Microsoft Azure portals do not support the revocation of permissions once they have been granted to an application like the Microsoft Graph Command Line Tools or Graph Explorer. While these permissions can be viewed, there is no option to remove them.

The Graph Explorer does offer some capabilities to revoke consent, but this requires additional permissions (`Directory.Read.All` and `DelegatedPermissionGrant.ReadWrite.All`) that may not be available to all users.

To manage user consents for the Microsoft Graph Command Line Tools and Graph Explorer, one must use the Microsoft Graph API itself.

## Script for Removing Microsoft Graph Scopes from User Consent

To simplify the process of resetting a user's consents, I've developed a PowerShell script. This script connects to the Graph API, provides guidance on missing permissions, and offers an interactive guide through the process.

Please note that this script still requires elevated administrative privileges. However, it significantly simplifies the task for administrators.

Required roles in Microsoft Entra:

- either [`Cloud Application Administrator`](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator), [`Application Administrator`](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) *or* [`Global Administrator`](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#global-administrator)

Required Graph scopes:

- `AppRoleAssignment.ReadWrite.All`
- `Directory.Read.All`
- `DelegatedPermissionGrant.ReadWrite.All`

Feel free to use this script to remove user consent, either all at once or selectively. You can even clean up all authorizations for all users at once, or just remove individual authorizations for all users.

As always, the source code can be found as a Gist snippet on GitHub:

{{< gist jpawlowski 7d4f2e76851349800e1cf86ff00ca43c >}}
