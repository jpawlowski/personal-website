---
translationKey: create-app-consent-custom-role-for-microsoft-graph
slug: benutzerdefinierte-app-zustimmung-fur-microsoft-graph-erstellen
title: Benutzerdefinierte App-Zustimmung für Microsoft Graph erstellen
subtitle: So lässt sich die Admin-Zustimmung für Microsoft Graph API-Berechtigungen sicher delegieren
description: Wie man eine benutzerdefinierte Rolle in Microsoft Entra implementiert, um die Admin-Zustimmung für Microsoft Graph API-Berechtigungen zu delegieren.
date: 2024-05-06T14:31:14.019Z
lastmod: 2024-05-10T11:14:00.0Z
preview: featured-image.jpg
comment: true
draft: false
tags:
  - Microsoft Graph API
  - Admin-Zustimmung
  - Benutzerdefinierte Rolle
  - Geringste Berechtigungen
  - Sicherheit
  - PowerShell
  - Azure AD
  - RBAC
  - IAM
categories:
  - Microsoft Entra
  - Cybersicherheit
  - Identitäts- und Zugriffsmanagement
  - Rollenbasierte Zugriffskontrolle
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
type: posts
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

{{< gist jpawlowski ca1bde7e979f367e8007b056bc032b6e >}}

Das Verhalten basiert auf dem, was die Rollen [Cloud-Anwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) oder [Anwendungsadministrator](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#application-administrator) zulassen, wobei jedoch Microsoft Graph zusammen mit einer schwarzen Liste von Berechtigungen hinzugefügt wird, die wir nicht wünschen.

Die Liste der ausgeschlossenen Anwendungsrollen kann je nach Anforderung geändert werden. Sie enthält im Wesentlichen App-Rollen, die Schreibberechtigungen mit potenziellem Schaden oder unkontrolliertem Informationsabfluss erteilen.

Eine Liste möglicher App-Rollen findet sich in der [Microsoft Graph-Berechtigungsreferenz](https://learn.microsoft.com/de-de/graph/permissions-reference), oder über eine direkte Abfrage der Microsoft Graph API und eines Filters:

````powershell
# Details zum Dienstprinzipal lesen
$MSGraphSP = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Suche nach App-Rollen, die das Schlüsselwort 'Write' enthalten
$MSGraphSP.AppRoles | Where-Object { $_.Value -like '*Write*' } | Format-Table Value, Description
`````
