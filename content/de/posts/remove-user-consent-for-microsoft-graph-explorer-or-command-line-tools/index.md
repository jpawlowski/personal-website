---
translationKey: remove-user-consent-for-microsoft-graph-explorer-or-command-line-tools
slug: benutzererlaubnis-fur-Graph-explorer-oder-command-line-tools-entfernen
title: Benutzererlaubnis für Graph Explorer oder Command Line Tools entfernen
subtitle: Microsoft Graph-Berechtigungen, die ein Benutzer erteilt hat, werden nicht berücksichtigt
description: Erfahre, wie du mit einem praktischen PowerShell-Skript die Benutzerzustimmung für Microsoft Graph API-Zugriffsrechte verwalten kannst.
date: 2024-05-11T08:07:00.323Z
lastmod: 2024-05-11T08:07:00.323Z
preview: featured-image.jpg
comment: true
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
toc: true
type: posts
---

## Die Zustimmung des Benutzers für den Zugriff auf Microsoft Graph API verstehen

Microsoft bietet mehrere Möglichkeiten zur Interaktion mit der Microsoft Graph API, wie in der [SDK-Übersicht](https://learn.microsoft.com/de-de/graph/sdks/sdks-overview) beschrieben. Während viele dieser Methoden für Softwareentwickler gedacht sind, interagieren Administratoren häufig mit der API über die [Microsoft Graph Command Line Tools](https://learn.microsoft.com/de-de/powershell/microsoftgraph/overview) für die Skripterstellung oder den [Graph Explorer](https://developer.microsoft.com/de-de/graph/graph-explorer) zum Erlernen und Untersuchen der API.

Diese Tools verwenden eine Methode, die als [Vom Benutzer delegierte Autorisierung](https://learn.microsoft.com/de-de/graph/security-authorization) bekannt ist und bei der eine von Microsoft bereitgestellte Cloud-Anwendung als Vermittler zwischen dem Benutzerkonto und der Microsoft Graph-API fungiert. Bei dieser Methode muss der Benutzer der Anwendung die Berechtigung erteilen, in seinem Namen zu handeln, unabhängig von den ihm zugewiesenen Rollenberechtigungen in Microsoft Entra.

## Widerruf der vom Benutzer delegierten Berechtigungen

Derzeit unterstützen die Microsoft Entra- oder Microsoft Azure-Portale nicht den Entzug von Berechtigungen, die einer Anwendung wie den Microsoft Graph Command Line Tools oder dem Graph Explorer erteilt wurden. Diese Berechtigungen können zwar eingesehen werden, aber es gibt keine Möglichkeit, sie zu entfernen.

Der Graph Explorer bietet einige Möglichkeiten, die Zustimmung zu widerrufen, aber dies erfordert zusätzliche Berechtigungen (`Directory.Read.All` und `DelegatedPermissionGrant.ReadWrite.All`), die möglicherweise nicht für alle Benutzer verfügbar sind.

Zur Verwaltung der Benutzerzustimmungen für die Microsoft Graph Command Line Tools und den Graph Explorer muss die Microsoft Graph API selbst verwendet werden.

## Skript zum Entfernen von Microsoft Graph-Berechtigungen aus der Benutzerzustimmung

Um den Prozess des Zurücksetzens der Benutzerzustimmungen zu vereinfachen, habe ich ein PowerShell-Skript entwickelt. Dieses Skript stellt eine Verbindung zur Graph-API her, gibt Hinweise auf fehlende Berechtigungen und bietet eine interaktive Anleitung durch den Prozess.

Bitte beachte, dass für dieses Skript nach wie vor erhöhte Administratorrechte erforderlich sind. Allerdings vereinfacht es die Aufgabe für Administratoren erheblich.

Erforderliche Rollen in Microsoft Entra:

- entweder [`Cloudanwendungsadministrator`](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator), [`Anwendungsadministrator`](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#application-administrator) *oder* [`Globaler Administrator`](https://learn.microsoft.com/de-de/entra/identity/role-based-access-control/permissions-reference#global-administrator)

Erforderliche Graph Scopes:

- `AppRoleAssignment.ReadWrite.All`
- `Directory.Read.All`
- `DelegatedPermissionGrant.ReadWrite.All`

Du kannst dieses Skript verwenden, um die Benutzerberechtigung zu entfernen, entweder alle auf einmal oder selektiv. Es lassen sich sogar komplett alle Berechtigungen für alle Benutzer auf einmal aufräumen, oder auch nur einzelne Berechtigungen bei allen Benutzern entfernen.

{{< gist jpawlowski 7d4f2e76851349800e1cf86ff00ca43c >}}
