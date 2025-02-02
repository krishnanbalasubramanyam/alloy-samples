# Client-side Personalization

## Overview

This sample demonstrates using Adobe Experience Platform to get personalization content from Adobe Target. The web page changes based on the personalization content returned.

This sample uses the [Adobe Experience Platform Web SDK](https://experienceleague.adobe.com/docs/experience-platform/edge/home.html) to get personalization content and to render it entirely client-side.

Here is what the page looks like before and after personalization content is rendered.

| without personalization                                     | with personalization                                              |
| ----------------------------------------------------------- | ----------------------------------------------------------------- |
| <img src="../.assets/plain.png" alt="drawing" width="800"/> | <img src="../.assets/with-offers.png" alt="drawing" width="800"/> |

Please review the [summary of personalization content](../TargetActivities.md) for this sample.

Keep in mind that personalization content can come from any Adobe source such as [Journey Optimizer](https://experienceleague.adobe.com/docs/journey-optimizer/using/ajo-home.html?lang=en) or [Target](https://experienceleague.adobe.com/docs/target.html). Target is the source of personalization content used in this sample, but the implementation described here will work for all personalization solutions Adobe offers.

## Running the sample

<small>Prerequisite: [install node and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm).</small>

To run this sample:

1. [Setup local SSL certificates for https](../LocalSSLCertificateSetup.md).
2. Clone the repository to your local machine.
3. Open a terminal and change directory to this sample's folder.
4. Run `npm install`
5. Run `npm start`
6. Open a web browser to [https://localhost](https://localhost)

## How it works

1. [Alloy](https://experienceleague.adobe.com/docs/experience-platform/edge/home.html) is included on the page.
2. The `sendEvent` command is used to fetch personalization content.

```javascript
alloy("sendEvent", {
  renderDecisions: true,
  decisionScopes: ["sample-json-offer"],
}).then(applyPersonalization("sample-json-offer"));
```

3. Alloy renders page load Visual Experience Composer (VEC) offers automatically because the `renderDecisions` flag is set to true.
4. Form-based JSON offers are manually applied by the sample implementation code (in the [`applyPersonalization`](./public/script.js) method) to update the DOM based on the offer.
5. For form-based activities, display events must manually be sent to indicate when the offer has been displayed. This is done via the `sendEvent` command.

```javascript
function sendDisplayEvent(decision) {
  const { id, scope, scopeDetails = {} } = decision;

  alloy("sendEvent", {
    xdm: {
      eventType: "decisioning.propositionDisplay",
      _experience: {
        decisioning: {
          propositions: [
            {
              id: id,
              scope: scope,
              scopeDetails: scopeDetails,
            },
          ],
        },
      },
    },
  });
}
```

## Key Observations

### Cookies

Cookies are used to persist user identity and cluster information. When using a client-side implementation, the Web SDK handles the storing and sending of these cookies automatically during the request lifecycle.

| Cookie                   | Purpose                                                                    | Stored by | Sent by |
| ------------------------ | -------------------------------------------------------------------------- | --------- | ------- |
| kndctr_AdobeOrg_identity | Contains user identity details                                             | Web SDK   | Web SDK |
| kndctr_AdobeOrg_cluster  | Indicates which experience edge cluster should be used to fulfill requests | Web SDK   | Web SDK |

### Request placement

Requests to Adobe Experience Platform API are required to get propositions and send a display notification. When using a client-side implementation, the Web SDK makes these requests when the `sendEvent` command is used.

| Request                                        | Made by                             |
| ---------------------------------------------- | ----------------------------------- |
| interact request to get propositions           | Web SDK using the sendEvent command |
| interact request to send display notifications | Web SDK using the sendEvent command |

### Flow Diagram

```mermaid
sequenceDiagram
  participant Browser
  participant Alloy
  participant DOM
  participant Browser Cookies
  participant API as Adobe Experience Platform API
  autonumber
  Browser->>Alloy: Page load
  Browser Cookies->>Alloy: Read identity and cluster cookies
  Alloy->>API: Interact request
  API->>Alloy: Return propositions
  Alloy->>Browser Cookies: Set identity and cluster cookies
  Alloy->>DOM: Render propositions
  Alloy->>API: Send display notification(s)
```

## Beyond the sample

This sample app can serve as a starting point for you to experiment and learn more about Adobe Experience Platform. For example, you can change a few environment variables so the sample app pulls in offers from your own AEP configuration. To do so, just open the `.env` file at the root of this repository and modify the variables. Restart the sample app, and you're ready to experiment using your own personalization content.
