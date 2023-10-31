# Ionic Angular Push Notification (PWA)
Documentation for Ionic Angular Push Notification (PWA) with Azure Notification Hub and Service Worker.

## Setup
### 1. Register Service Worker (app.module.ts)
```typescript # Path: src/app/app.module.ts
import { ServiceWorkerModule } from '@angular/service-worker';
import { environment } from '../environments/environment';
```
```typescript # Path: src/app/app.module.ts
imports: [
    ServiceWorkerModule.register('ngsw-worker.js', { enabled: environment.production })
],
```

### 2. WebPubSubService (webpubsub.service.ts)
```typescript # Path: src/app/webpubsub.service.ts
import { Observable } from 'rxjs';
import { HttpClient } from '@angular/common/http';
import { environment } from 'src/environments/environment';
```
```typescript # Path: src/environments/environment.ts
export const environment = {
    production: false,
    notificationApiUrl: 'https://<your-api-url>.azurewebsites.net/api'
};
```

```typescript # Path: src/app/webpubsub.service.ts
constructor(
    private http: HttpClient
) { }

addSubscription(sub: any): Observable<any>{
    return this.http.post(`${environment.notificationApiUrl}/subscribe`, sub);
}

removeSubscription(sub: any): Observable<any>{
    return this.http.post(`${environment.notificationApiUrl}/unsubscribe`, sub);
}
```

### 3. Push Notification (app.component.ts)
```typescript # Path: src/app/app.component.ts
import { WebPubSubService } from './webpubsub.service';
```
```typescript # Path: src/app/app.component.ts
constructor(
    private webPubSubService: WebPubSubService
) {
    if ('serviceWorker' in navigator && 'PushManager' in window) {
          navigator.serviceWorker.ready.then((reg) => {
            console.log("Service Worker ready!");
            reg.pushManager.getSubscription().then((sub) => {
                if(sub === undefined){
                    console.log("Not subscribed to push service!");
    
                    // subscribe user
                    reg.pushManager.subscribe({
                        userVisibleOnly: true
                    }).then((sub) => {
                        console.log("Subscribed to push service!", sub);
                        this.webPubSubService.addSubscription({
                            userId: "123456789123", // your user id
                            app: "doki" // your app name
                        }).subscribe({
                            next: data => {
                                console.log(data);
                            },
                            error: error => {
                                console.log(error);
                            }
                        });
                    }).catch((err) => {
                        console.log("Could not subscribe to push service!", err);
                    });
                }else{
                    console.log("Subscription object: ", sub);
                }
            });
        });
    }else{
        console.log("Service Worker and Push is not supported");
    }
}
```
```typescript # Path: src/app/app.component.ts
displayNotification(){
    Notification.requestPermission((status) => {
        console.log("Notification permission status:", status);

        if(status === "granted"){
            navigator.serviceWorker.getRegistration().then((reg) => {
                const options = {
                    body: "Test Notification Body",
                    icon: "https://cdn-icons-png.flaticon.com/512/3119/3119338.png",
                    vibrate: [100, 50, 100],
                    data: {
                        dateOfArrival: Date.now(), 
                        primaryKey: 1
                    },
                    actions: [
                        {
                            action: 'explore', 
                            title: 'Go to the site', 
                            icon: 'https://www.iconpacks.net/icons/2/free-check-mark-icon-3279-thumb.png'
                        },
                        {
                            action: 'close', 
                            title: 'No thank you', 
                            icon: 'https://cdn-icons-png.flaticon.com/512/106/106830.png'
                        },
                    ]
                };
                reg.pushManager.subscribe({
                    userVisibleOnly: true
                }).then((sub) => {
                    console.log(sub);
                    this.webPubSubService.addSubscription({
                        userId: "123456789123", // your user id
                        app: "doki" // your app name
                    }).subscribe({
                        next: data => {
                            console.log(data);
                        },
                        error: error => {
                            console.log(error);
                        }
                    });
                });
                console.log('reg.showNotification("Test notification", options);');

                reg.showNotification("Test notification", options);
            });
        }
    });
}
```

