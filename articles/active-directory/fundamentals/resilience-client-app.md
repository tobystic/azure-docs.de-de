---
title: Erhöhen der Resilienz bei Authentifizierung und Autorisierung in von Ihnen entwickelten Clientanwendungen
titleSuffix: Microsoft identity platform
description: Leitfaden zur Steigerung der Resilienz bei Authentifizierung und Autorisierung in einer Clientanwendung mithilfe von Microsoft Identity Platform
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: knicholasa
ms.author: nichola
manager: martinco
ms.date: 11/23/2020
ms.openlocfilehash: b32f9dd10d9bd03a7e446616d9941e7bd1a9c3ed
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "98724907"
---
# <a name="increase-the-resilience-of-authentication-and-authorization-in-client-applications-you-develop"></a>Erhöhen der Resilienz bei Authentifizierung und Autorisierung in von Ihnen entwickelten Clientanwendungen

Dieser Abschnitt enthält eine Anleitung zum Erzielen von Resilienz in Clientanwendungen, die Microsoft Identity Platform und Azure Active Directory verwenden, um Benutzer anzumelden und Aktionen im Namen dieser Benutzer auszuführen.

## <a name="use-the-microsoft-authentication-library-msal"></a>Verwenden von Microsoft Authentication Library (MSAL)

[Microsoft Authentication Library (MSAL)](../develop/msal-overview.md) ist ein elementarer Teil der [Microsoft Identity Platform](../develop/index.yml). MSAL vereinfacht das Abrufen, Verwalten, Zwischenspeichern und Aktualisieren von Token und nutzt bewährte Methoden für die Resilienz. MSAL ist konzeptionell für eine sichere Lösung ausgelegt, ohne dass Entwickler sich über die Implementierungsdetails Gedanken machen müssen.

MSAL übernimmt das Zwischenspeichern von Token und verwendet ein Muster für den automatischen Tokenabruf. Außerdem wird der Tokencache auf Plattformen, die nativ sicheren Speicher bereitstellen (z. B. Windows UWP, iOS und Android), automatisch serialisiert. Entwickler können bei Verwendung von [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization), [MSAL.NET](../develop/msal-net-token-cache-serialization.md), [MSAL für Java](../develop/msal-java-token-cache-serialization.md)und [MSAL für Python](../develop/msal-python-token-cache-serialization.md) das Serialisierungsverhalten anpassen.

![Abbildung des Geräts mit MSAL; die Anwendung verwendet MSAL zum Aufrufen von Microsoft Identity](media/resilience-client-app/resilience-with-microsoft-authentication-library.png)

Bei der Verwendung von MSAL wird automatisch das Zwischenspeichern, Aktualisieren und automatische Abrufen von Token unterstützt. Sie können einfache Muster verwenden, um die für die moderne Authentifizierung erforderlichen Token abzurufen. Wir unterstützen viele Sprachen. Auf der Seite [Beispiele](../develop/sample-v2-code.md) finden Sie auch ein Beispiel, das Ihrer Sprache und Ihrem Szenario entspricht.

## <a name="c"></a>[C#](#tab/csharp)

```csharp
try
{
    result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
    result = await app.AcquireToken(scopes).WithClaims(ex.Claims).ExecuteAsync()
}
```

## <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
return myMSALObj.acquireTokenSilent(request).catch(error => {
    console.warn("silent token acquisition fails. acquiring token using redirect");
    if (error instanceof msal.InteractionRequiredAuthError) {
        // fallback to interaction when silent call fails
        return myMSALObj.acquireTokenPopup(request).then(tokenResponse => {
            console.log(tokenResponse);

            return tokenResponse;
        }).catch(error => {
            console.error(error);
        });
    } else {
        console.warn(error);
    }
});
```

---

MSAL kann in manchen Fällen Token proaktiv aktualisieren. Wenn Microsoft Identity ein langlebiges Token ausgibt, kann es Informationen zum optimalen Aktualisierungszeitpunkt des Tokens an den Client senden („Aktualisieren\_in“). Anhand dieser Informationen aktualisiert MSAL proaktiv das Token. Die App wird weiterhin ausgeführt, während das alte Token gültig ist. Das Zeitfenster für einen weiteren erfolgreichen Tokenabruf wird jedoch vergrößert.

### <a name="stay-up-to-date"></a>Auf dem Laufenden bleiben

Entwickler sollten einen Prozess zum Aktualisieren auf die neueste MSAL-Version einrichten. Die Authentifizierung ist Bestandteil der App-Sicherheit, und Ihre App muss mit den in neuen MSAL-Versionen enthaltenen Sicherheitsverbesserungen auf dem neuesten Stand bleiben. Dies ist allgemein eine bewährte Methode für Bibliotheken, die ständig weiterentwickelt werden. Dadurch wird sichergestellt, dass Sie im Hinblick auf die Resilienz der App immer über den neuesten Datencode verfügen. Microsoft Identity wird auch weiterhin innovative Möglichkeiten zum Erhöhen der Resilienz von Anwendungen bieten. Daher sind Apps, welche die aktuelle MSAL verwenden, am besten auf die Nutzung dieser Innovationen vorbereitet.

[Überprüfen auf aktuelle MSAL.js-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)

[Überprüfen auf aktuelle MSAL .NET-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/releases)

[Überprüfen auf aktuelle MSAL Python-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-python/releases)

[Überprüfen auf aktuelle MSAL Java-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-java/releases)

[Überprüfen auf aktuelle MSAL iOS- und MacOS-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-objc/releases)

[Überprüfen auf aktuelle MSAL Android-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-android/releases)

[Überprüfen auf aktuelle MSAL Angular-Version und Versionshinweise](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)

[Überprüfen auf aktuelle MSAL.Identity.Web-Version und Versionshinweise](https://github.com/AzureAD/microsoft-identity-web/releases)

## <a name="use-resilient-patterns-for-token-handling"></a>Verwenden von robusten Mustern für die Tokenbehandlung

Wenn Sie MSAL nicht verwenden, können Sie diese robusten Muster für die Tokenbehandlung verwenden. Diese bewährten Methoden werden automatisch durch die MSAL-Bibliothek implementiert. 

Im Allgemeinen ruft eine Anwendung, welche die moderne Authentifizierung verwendet, einen Endpunkt auf, um Token zum Authentifizieren des Benutzers oder zum Autorisieren der Anwendung zum Aufrufen geschützter APIs abzurufen. MSAL soll die Einzelheiten der Authentifizierung verarbeiten und verschiedene Muster implementieren, um die Resilienz bei diesem Prozess zu verbessern. Verwenden Sie die Anleitung zum Implementieren bewährter Methoden in diesem Abschnitt, wenn Sie sich für die Verwendung einer anderen Bibliothek als MSAL entscheiden. Wenn Sie MSAL verwenden, erhalten Sie alle bewährten Methoden kostenlos, da MSAL diese automatisch implementiert.

### <a name="cache-tokens"></a>Zwischenspeichern von Token

Apps müssen von Microsoft Identity empfangene Token ordnungsgemäß zwischenspeichern. Wenn Ihre App Token empfängt, enthält die HTTP-Antwort mit den Token auch die Eigenschaft „expires_in“, mit der die Anwendung informiert wird, wie lange das Token zwischengespeichert (und wiederverwendet) werden soll. Es ist wichtig, dass Anwendungen die Eigenschaft „expires_in“ verwenden, um die Gültigkeitsdauer des Tokens zu bestimmen. Die Anwendung darf niemals versuchen, ein API-Zugriffstoken zu decodieren.

![Eine Anwendung ruft Microsoft Identity auf, der Aufruf durchläuft jedoch einen Tokencache auf dem Gerät, auf dem die Anwendung ausgeführt wird.](media/resilience-client-app/token-cache.png)

Die Verwendung des zwischengespeicherten Tokens vermeidet unnötigen Datenverkehr zwischen Ihrer App und Microsoft Identity. Durch die Reduzierung der Aufrufe zum Abrufen von Token ist Ihre App weniger anfällig für Fehler beim Tokenabruf. Zwischengespeicherte Token verbessern außerdem die Leistung Ihrer Anwendung, da die App das Abrufen von Token weniger blockieren muss. Der Benutzer kann für die Länge der Lebensdauer des Tokens bei Ihrer Anwendung angemeldet bleiben.

### <a name="serialize-and-persist-tokens"></a>Serialisieren und Beibehalten von Token

Apps sollten in der Lage sein, den Tokencache sicher zu serialisieren, um die Token zwischen den Instanzen der App beizubehalten. Token können wiederverwendet werden, solange sie sich innerhalb der Gültigkeitsdauer befinden. [Aktualisierungstoken](../develop/v2-oauth2-auth-code-flow.md#refresh-the-access-token) und zunehmend auch [Zugriffstoken](../develop/access-tokens.md) werden für viele Stunden ausgegeben. Während dieser Gültigkeitsdauer kann ein Benutzer die Anwendung mehrmals starten. Beim Start Ihrer App muss diese überprüfen, ob ein gültiges Zugriffs- oder Aktualisierungstoken vorhanden ist, das verwendet werden kann. Dadurch erhöhen sich Resilienz und Leistung der App, da unnötige Aufrufe von Microsoft Identity vermieden werden.

![Eine Anwendung ruft Microsoft Identity auf, der Aufruf durchläuft jedoch einen Tokencache sowie einen Tokenspeicher auf dem Gerät, auf dem die Anwendung ausgeführt wird.](media/resilience-client-app/token-store.png)

Der Zugriff auf den persistenten Tokenspeicher muss gesteuert und für den besitzenden Benutzer oder die Prozessidentität verschlüsselt werden. Auf Plattformen wie Mobile, Windows und Mac sollte der Entwickler die integrierten Funktionen zum Speichern von Anmeldeinformationen nutzen.

### <a name="acquire-tokens-silently"></a>Automatisches Abrufen von Token

Für den Prozess zum Authentifizieren eines Benutzers oder zum Abrufen der Autorisierung zum Aufrufen einer API können in Microsoft Identity mehrere Schritte erforderlich sein. Wenn sich der Benutzer beispielsweise zum ersten Mal anmeldet, muss er möglicherweise Anmeldeinformationen eingeben und eine mehrstufige Authentifizierung (MFA) über eine Textnachricht ausführen. Jeder Schritt erhöht die Abhängigkeit von der Ressource, die diesen Dienst bereitstellt. Das automatische Abrufen eines Tokens, um vor dem Anfordern der Benutzerinteraktion diese zusätzlichen Schritte zu vermeiden, sorgt für eine optimale Benutzererfahrung und minimiert die Abhängigkeiten.

![Diagramm der verschiedenen Dienste in Microsoft Identity, die möglicherweise ausgeführt werden müssen, um den Prozess der Authentifizierung oder Autorisierung eines Benutzers abzuschließen.](media/resilience-client-app/external-dependencies.png)

Das automatische Abrufen eines Tokens beginnt mit der Verwendung eines gültigen Tokens aus dem Tokencache der App. Wenn kein gültiges Token verfügbar ist, muss die App versuchen, mithilfe eines Aktualisierungstokens (falls verfügbar) und des Tokenendpunkts ein Token abzurufen. Wenn keine dieser Optionen verfügbar ist, muss die App mithilfe des Parameters „prompt=none“ ein Token abrufen. Dabei wird der Autorisierungsendpunkt verwendet, dem Benutzer wird jedoch keine Benutzeroberfläche angezeigt. Wenn Microsoft Identity ohne Interaktion mit dem Benutzer ein Token für die App bereitstellen kann, kommt diese Vorgehensweise zur Anwendung. Wenn keine dieser Methoden zu einem Token führt, muss sich der Benutzer interaktiv erneut authentifizieren.

> [!NOTE]
> Im Allgemeinen sollten Sie bei Apps die Verwendung von Eingabeaufforderungen wie „login“ und „consent“ vermeiden, da sie die Benutzerinteraktion selbst dann erzwingen, wenn keine Interaktion erforderlich ist.

## <a name="handle-service-responses-properly"></a>Ordnungsgemäßes Verarbeiten von Dienstantworten

Auch wenn die Anwendungen alle Fehlerantworten verarbeiten sollten, gibt es einige Antworten, die sich auf die Resilienz auswirken können. Wenn Ihre Anwendung einen HTTP-Antwortcode 429 (zu viele Anforderungen) empfängt, drosselt Microsoft Identity Ihre Anforderungen. Wenn Ihre App weiterhin zu viele Anforderungen vornimmt, wird sie weiterhin gedrosselt, damit Ihre App keine Token empfängt. Ihre Anwendung sollte nicht versuchen, vor der im Antwortfeld „Retry-After“ angegebenen Zeit (in Sekunden) erneut ein Token abzurufen. Das Empfangen einer Antwort vom Typ 429 ist häufig ein Hinweis darauf, dass die Anwendung Token nicht ordnungsgemäß zwischenspeichert und wiederverwendet. Entwickler sollten überprüfen, wie Token in der Anwendung zwischengespeichert und wiederverwendet werden.

Wenn eine Anwendung einen HTTP-Antwortcode 5xx empfängt, darf die App nicht in eine schnelle Wiederholungsschleife eintreten. Wenn dies der Fall ist, sollte für die Anwendung dieselbe „Retry-After“-Behandlung gelten wie bei einer Antwort vom Typ 429. Wenn von der Antwort kein „Retry-After“-Header bereitgestellt wird, empfehlen wir die Implementierung einer Wiederholung mit exponentiellem Backoff. Dabei sollte der erste Wiederholungsversuch mindestens 5 Sekunden nach der Antwort erfolgen.

Wenn für eine Anforderung ein Timeout auftritt, sollten Anwendungen nicht sofort Wiederholungsversuche ausführen. Implementieren Sie einen exponentiellen Backoff-Wiederholungsversuch mit dem ersten Wiederholungsversuch mindestens 5 Sekunden nach der Antwort.

## <a name="evaluate-options-for-retrieving-authorization-related-information"></a>Auswerten von Optionen zum Abrufen von autorisierungsbezogenen Informationen

Viele Anwendungen und APIs benötigen spezielle Informationen zum Benutzer, um Autorisierungsentscheidungen treffen zu können. Es gibt einige Methoden, mit denen eine Anwendung an diese Informationen gelangen kann. Jede Methode hat Vor- und Nachteile. Entwickler sollten diese gegeneinander abwägen, um herauszufinden, welche Strategie unter Resilienzgesichtspunkten für ihre App am besten geeignet ist.

### <a name="tokens"></a>Token

Identitäts- und Zugriffstoken enthalten Standardansprüche, die Informationen zum Antragsteller bereitstellen. Diese sind unter [Microsoft Identity Platform – ID-Token](../develop/id-tokens.md) und unter [Microsoft Identity Platform – Zugriffstoken](../develop/access-tokens.md) dokumentiert. Wenn die von Ihrer App benötigten Informationen bereits im Token enthalten sind, ist die effizienteste Methode zum Abrufen dieser Daten die Verwendung von Tokenansprüchen, da damit der Aufwand eines zusätzlichen Netzwerkaufrufs zum separaten Abrufen der Informationen entfällt. Weniger Netzwerkaufrufe bedeuten eine höhere allgemeine Resilienz für die Anwendung.

> [!NOTE]
> Einige Anwendungen rufen zum Abrufen der Ansprüche zum authentifizierten Benutzer den UserInfo-Endpunkt auf. Die im ID-Token verfügbaren Informationen, die Ihre App empfangen kann, sind eine übergeordnete Menge der Informationen, die sie vom UserInfo-Endpunkt abrufen kann. Ihre App sollte zum Abrufen der Informationen zum Benutzer das ID-Token verwenden und nicht den UserInfo-Endpunkt aufrufen.

Ein Anwendungsentwickler kann die Standardtokenansprüche durch [optionale Ansprüche](../develop/active-directory-optional-claims.md) ausweiten. Ein häufig verwendeter optionaler Anspruch ist [Gruppen](../develop/active-directory-optional-claims.md#configuring-groups-optional-claims). Es gibt verschiedene Möglichkeiten, Gruppenansprüche hinzuzufügen. Die Option „Anwendungsgruppe“ enthält nur Gruppen, die der Anwendung zugewiesen sind. Mit den Optionen „Alle“ oder „Sicherheitsgruppen“, die Gruppen aus allen Apps im selben Mandanten enthalten, können Sie dem Token viele Gruppen hinzufügen. Es ist wichtig, im Einzelfall die Auswirkungen auszuwerten, da eine Tokenüberfrachtung entstehen kann und sogar zusätzliche Aufrufe erforderlich werden, um die vollständige Liste der Gruppen abzurufen. Dadurch würde möglicherweise die Effizienz, die durch das Anfordern von Gruppen im Token erzielt wird, zunichte gemacht.

Anstelle von Gruppen in Ihrem Token können Sie App-Rollen verwenden und einbeziehen. Entwickler können [App-Rollen](../develop/howto-add-app-roles-in-azure-ad-apps.md) für ihre Apps und APIs definieren, die der Kunde im Portal oder mithilfe von APIs aus seinem Verzeichnis verwalten kann. IT-Fachleute können anschließend verschiedenen Benutzern und Gruppen Rollen zuweisen, um zu steuern, wer auf welche Inhalte und Funktionen zugreifen kann. Wenn ein Token für die Anwendung oder API ausgestellt wird, stehen die dem Benutzer zugewiesenen Rollen im Rollenanspruch im Token zur Verfügung. Wenn Sie diese Informationen direkt in einem Token erhalten, können Sie auf weitere API-Aufrufe verzichten.

Schließlich können IT-Administratoren auch Ansprüche anhand bestimmter Informationen in einem Mandanten hinzufügen. So kann z. B. ein Unternehmen eine Erweiterung verwenden, um eine unternehmensspezifische Benutzer-ID zu bilden.

In jedem Fall kann das direkte Hinzufügen von Informationen aus dem Verzeichnis zu einem Token effizient sein und durch Reduzierung der Abhängigkeiten die Resilienz der App erhöhen. Andererseits werden auf diese Weise keine Resilienzprobleme gelöst, die dadurch entstehen, dass ein Token nicht abgerufen werden kann. Sie sollten nur für die wichtigsten Szenarien Ihrer Anwendung optionale Ansprüche hinzufügen. Wenn die App nur Informationen für die Administratorfunktion benötigt, ist es für die Anwendung am besten, diese Informationen nur bei Bedarf zu erhalten.

### <a name="microsoft-graph"></a>Microsoft Graph

Microsoft Graph stellt einen einheitlichen API-Endpunkt für den Zugriff auf Microsoft 365-Daten bereit, mit denen die Muster von Produktivität, Identität und Sicherheit in einer Organisation beschrieben werden. Anwendungen, die Microsoft Graph verwenden, können potenziell alle Informationen von Microsoft 365 für Autorisierungsentscheidungen verwenden.

Apps benötigen nur ein einziges Token für den Zugriff auf alle Bestandteile von Microsoft 365. Dies ist resilienter als die Verwendung der älteren APIs, die speziell für Microsoft 365-Komponenten wie Microsoft Exchange oder Microsoft SharePoint entwickelt wurden und bei denen mehrere Token erforderlich sind.

Bei Verwendung von Microsoft Graph-APIs empfehlen wir, ein [Microsoft Graph-SDK](/graph/sdks/sdks-overview) zu verwenden. Microsoft Graph-SDKs sollen die Entwicklung hochwertiger, effizienter und resilienter Anwendungen, die auf Microsoft Graph zugreifen, vereinfachen.

Bei Autorisierungsentscheidungen sollten Entwickler überlegen, wann die in einem Token verfügbaren Ansprüche als Alternative zu einigen Microsoft Graph-Aufrufen verwendet werden können. Wie bereits erwähnt, könnten Entwickler in ihren Token Gruppen, App-Rollen und optionale Ansprüche anfordern. Unter Resilienzgesichtspunkten sind bei der Autorisierung mit Microsoft Graph nicht nur Microsoft Graph selbst, sondern auch zusätzliche Netzwerkaufrufe auf Basis von Microsoft Identity erforderlich (um das Token für den Zugriff auf Microsoft Graph abzurufen). Wenn Ihre Anwendung jedoch bereits Microsoft Graph als Datenschicht nutzt, stellt die Verwendung von Microsoft Graph für die Autorisierung kein zusätzliches Risiko dar.

## <a name="use-broker-authentication-on-mobile-devices"></a>Verwenden der Broker-Authentifizierung auf mobilen Geräten

Bei mobilen Geräten sorgt die Verwendung eines Authentifizierungsbrokers wie Microsoft Authenticator für mehr Resilienz. Der Broker bietet weitere Vorteile, die über die Funktionen anderer Optionen (z. B. Systembrowser oder eingebettete Webansicht) hinausgehen. Der Authentifizierungsbroker kann ein [primäres Aktualisierungstoken](../devices/concept-primary-refresh-token.md) nutzen, das Ansprüche zum Benutzer und zum Gerät enthält und zum Abrufen von Authentifizierungstoken für den Zugriff auf andere Anwendungen über das Gerät verwendet werden kann. Wenn für die Anforderung des Zugriffs auf eine Anwendung ein primäres Aktualisierungstoken verwendet wird, werden die zugehörigen Geräte- und MFA-Ansprüche von Azure AD als vertrauenswürdig eingestuft. Dadurch erhöht sich die Resilienz, da zusätzliche Schritte zum erneuten Authentifizieren des Geräts vermieden werden. Benutzer werden auf demselben Gerät nicht mehrmals zur Eingabe von MFA-Informationen aufgefordert. Dadurch reduziert sich die Abhängigkeit von externen Diensten, und die Benutzererfahrung wird optimiert. Das alles sorgt für mehr Resilienz.

![Eine Anwendung ruft Microsoft Identity auf. Der Aufruf durchläuft jedoch auf dem Gerät, auf dem die Anwendung ausgeführt wird, einen Tokencache sowie einen Tokenspeicher und einen Authentifizierungsbroker.](media/resilience-client-app/authentication-broker.png)

Die Broker-Authentifizierung wird automatisch von MSAL unterstützt. Weitere Informationen zur Verwendung der Broker-Authentifizierung finden Sie auf den folgenden Seiten:

- [Konfigurieren von SSO unter macOS und iOS](../develop/single-sign-on-macos-ios.md#sso-through-authentication-broker-on-ios)
- [Aktivieren des anwendungsübergreifenden einmaligen Anmeldens (SSO) unter Android mit MSAL](../develop/msal-android-single-sign-on.md)

## <a name="adopt-continuous-access-evaluation"></a>Einführen der fortlaufenden Zugriffsevaluierung (CAE)

Die [fortlaufende Zugriffsevaluierung](../conditional-access/concept-continuous-access-evaluation.md) (Continuous Access Evaluation, CAE) ist eine aktuelle Entwicklung, die durch langlebige Token Anwendungssicherheit und Resilienz erhöhen kann. CAE ist ein neu entstehender Industriestandard, der in der Arbeitsgruppe „Shared Signals and Events Working Group“ der OpenID Foundation entwickelt wird. Statt Token mit kurzer Gültigkeitsdauer zu verwenden, können Sie bei der fortlaufenden Zugriffsevaluierung (CAE) ein Zugriffstoken auf Basis von [kritischen Ereignissen](../conditional-access/concept-continuous-access-evaluation.md#critical-event-evaluation) und [Richtlinienauswertung](../conditional-access/concept-continuous-access-evaluation.md#conditional-access-policy-evaluation-preview) widerrufen. Dank der Risiko- und Richtlinienauswertung in Echtzeit kann die fortlaufende Zugriffsevaluierung (CAE) bei einigen Ressourcen-APIs die Tokengültigkeitsdauer wesentlich verlängern (auf bis zu 28 Stunden). Während die fortlaufende Zugriffsevaluierung (CAE) für Ressourcen-APIs und Anwendungen eingeführt wird, wird auch Microsoft Identity Zugriffstoken ausstellen können, die widerrufbar und für längere Zeiträume gültig sind. Diese langlebigen Token werden proaktiv von MSAL aktualisiert.

Auch wenn sich die fortlaufende Zugriffsevaluierung (CAE) noch in frühen Entwicklungsphasen befindet, ist es möglich, [heute Clientanwendungen zu entwickeln, die von der CAE profitieren](../develop/app-resilience-continuous-access-evaluation.md), wenn für die von der Anwendung verwendeten Ressourcen (APIs) die CAE eingeführt wird. Wenn die CAE für mehr Ressourcen eingeführt wird, kann Ihre Anwendung auch für diese Ressourcen CAE-fähige Token abrufen. Die CAE-Funktion wird Anfang 2021 für die Microsoft Graph-API und für [Microsoft Graph-SDKs](/graph/sdks/sdks-overview) als Vorschauversion verfügbar sein. Wenn Sie die öffentliche Vorschau von Microsoft Graph mit CAE nutzen möchten, können Sie uns hier Ihr Interesse mitteilen: [https://aka.ms/GraphCAEPreview](https://aka.ms/GraphCAEPreview).

Wenn Sie Ressourcen-APIs entwickeln, regen wir an, sich an der Arbeitsgruppe [Shared Signals and Events WG](https://openid.net/wg/sse/) zu beteiligen. Wir arbeiten mit dieser Gruppe zusammen, um die gemeinsame Nutzung von Sicherheitsereignissen zwischen Microsoft Identity und Ressourcenanbietern zu ermöglichen.

## <a name="next-steps"></a>Nächste Schritte

- [Verwenden von CAE-fähigen APIs in Ihren Anwendungen](../develop/app-resilience-continuous-access-evaluation.md)
- [Erzielen von Resilienz in Daemon-Anwendungen](resilience-daemon-app.md)
- [Erzielen von Resilienz in Ihrer Identitäts- und Zugriffsverwaltungsinfrastruktur](resilience-in-infrastructure.md)
- [Erzielen von Resilienz für Ihre CIAM-Systeme](resilience-b2c.md)