# Ionic Angular Push Notification (PWA)
Documentation for Ionic Angular Push Notification (PWA) with Service Worker and Azure Notification Hub.

## Setup
### 1. Register Service Worker in `app.module.ts`
```typescript # Path: src/app/app.module.ts
import { ServiceWorkerModule } from '@angular/service-worker';
import { environment } from '../environments/environment';
```
```typescript # Path: src/app/app.module.ts
imports: [
    ServiceWorkerModule.register('service-worker.js', { enabled: environment.production })
],
```

### 2. Create `service-worker.js` file in `src` directory and import `ngsw-worker.js`
```typescript # Path: src/service-worker.js
importScripts('./ngsw-worker.js');

// code for background sync
```

### 3. Subcribe for push notification
```typescript
// sample public key hardcoded (same public key sa backend)
private readonly publicKey = 'BCol311jRW4M59BwcFAMiESdjaTHaNGQTJ-kC88feFnLEJ6nC-2JFOBcMX-rLRIO8NaaXYwDRCLn1a_s4XgR384';
```

```typescript
constructor(
    private swPush: SwPush
  ) { }
```

```typescript
ngOnInit() {
    // Push notification
    this.pushSubscription();

    // Listen to push notification
    this.swPush.messages.subscribe({
      next: (data) => {
        console.log(data)
      },
      error: (err) => console.log(err),
    });

    // Listen to notification click
    this.swPush.notificationClicks.subscribe({
      next: (data) => {
        console.log(data);
      },
      error: (err) => console.log(err),
    });
  }
```

```typescript
pushSubscription() {
    // Check if push notification is enabled
    if (!this.swPush.isEnabled) {
      console.log('Notification is not enabled');
      return;
    }

    // Request subscription
    this.swPush
      .requestSubscription({
        serverPublicKey: this.publicKey,
      })
      .then((sub) => {
        // Send subscription to server
        console.log(JSON.stringify(sub));
      })
      .catch((err) => console.log(err));
}
```

### 4. NodeJS for Push Notification Server (Backend)
```typescript
import webPush from "web-push";

// generate public and private keys manually
console.log(webPush.generateVAPIDKeys()); // comment this after generating keys

// save generated keys from webPush.generateVAPIDKeys() hardcoded
const publicKey = 'BCol311jRW4M59BwcFAMiESdjaTHaNGQTJ-kC88feFnLEJ6nC-2JFOBcMX-rLRIO8NaaXYwDRCLn1a_s4XgR384';
const privateKey = 'CjLPPZaLJNhv6dynvL_BMURqHwWRpjfI-K2G0PZkXB0';

// sample hardcoded client endpoint
// this is generated from this.swPush.requestSubscription()
const sub = {
    endpoint: 'https://updates.push.services.mozilla.com/wpush/v2/gAAAAABlQ33HwyGckJgFadvDbKdT_ridScZnUJ-0J8gdzu3cWT9-JZyGVdw1wyTnLLS1BhxfMPkiXWSh9ziwMxY7AgWrEuEPl5BRjDKy4s2cg2IaVgyjP5PXuzz5ZaPhCPkW02WDroTCun5mFmZk9PIRRQ6JFlYi-4eIY2tsEiUHbCAJwAQ7Te4',
    expirationTime: null,
    keys: {
        p256dh: 'BBfElKxsq0AQbwSGQvjo8-9JH7qD1Hgy7ompty4IUp35RamJPt-yFL7tuMncMcaEso3PE8Fwq8g0L3t1of07Xe0',
        auth: 'd6MTOjqx4r1s-HN5b3PWkw',
    },
};

// sample payload to be send
const payLoad = {
    notification: {
        data: {
            url: 'http://www.youtube.com'
        },
        title: 'Test Notification from Server',
        vibrate: [100, 50, 100],
        // ...
    },
};

// call this before sending notification
webPush.setVapidDetails("https: or mailto:", publicKey, privateKey);

// send notification to subscribed user
webPush.sendNotification(sub, JSON.stringify(payLoad))
.then((result) => {
    console.log(result);
})
.catch((err) => {
    console.log(err);
});
```
