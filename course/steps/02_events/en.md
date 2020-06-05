# Events: Handling and receiving events

## Introduction

Some interactions on VTEX IO can generate events and they can be used as triggers for actions, like the activity on this step. For this step, we will use the events fired by the `events-example` app.

## Events on VTEX IO

On VTEX IO apps, events can be fired and used to trigger actions. For example, an app that listen for order placements and triggers a confirmation e-mail. It is important to highlight that events are _workspace and account bound_, which means that events are only visible for the account and workspace where it was fired. Events fired on your personal workspace will only be listened to by apps linked on this same workspace.

## Activity

1. First, we need to start by starting out the event firing on the `events-example` app. This app will fire an event every 10 seconds. After runing `vtex link` on its directory, click on the healthcheck route available and a "ok" message should appear on the browser:

![image](https://user-images.githubusercontent.com/43679629/83802091-8c69f380-a680-11ea-82af-a438fb73f40b.png)

> This healthcheck route access creates a cache context needed for the VTEX IO to fire events. Without it, the `events-example` app won't be able to fire the events our app is going to listen to.

2. We need to add the event handler on the `Service` declaration to refer to what the app is supposed to do when listening to the event. To do so, on the `/node/index.ts` file, add this code on the `Service` declaration:

```diff
export default new Service<Clients, State, ParamsContext>({
  clients: {
    implementation: Clients,
    options: {
      default: {
        retries: 2,
        timeout: 10000,
      },
+      events: {
+        exponentialTimeoutCoefficient: 2,
+        exponentialBackoffCoefficient: 2,
+        initialBackoffDelay: 50,
+        retries: 1,
+        timeout: TREE_SECONDS_MS,
+        concurrency: CONCURRENCY,
+      },
+    },
+  },
})
```

> By adding this code to the `Service`, we are adding to the `Client` of this `Service`, the capability to handle events. At this point, we are not yet using the `Client` itself when handling the event.

3. For now, we are only going to create a log when receiving a event. To create this event handler, in the `/node/event` directory, create the `liveUsersUpdate.ts` file and insert the code below:

```
export async function updateLiveUsers() {
  console.log('EVENT HANDLER: received event')
}
```

4. Now, we need to declare in the `Service` the reference to this function. On the `/node/index.ts` file, add this code:

```diff
...
+ import { updateLiveUsers } from './event/liveUsersUpdate'
...

export default new Service<Clients, State, ParamsContext>({
  clients: {
    implementation: Clients,
    options: {
      default: {
        retries: 2,
        timeout: 10000,
      },
      events: {
        exponentialTimeoutCoefficient: 2,
        exponentialBackoffCoefficient: 2,
        initialBackoffDelay: 50,
        retries: 1,
        timeout: TREE_SECONDS_MS,
        concurrency: CONCURRENCY,
      },
    },
  },
+  events: {
+    liveUsersUpdate: updateLiveUsers,
+  },
})

```

5. We also need to modify the `service.json` file. In order to listen to events sent, we need to declare this to give the app's service this capability. To do so, replace the code with this one:

```diff
{
  "memory": 128,
  "ttl": 10,
  "timeout": 10,
  "minReplicas": 2,
  "maxReplicas": 10,
  "workers": 4,
+  "events": {
+    "liveUsersUpdate": {
+      "sender": "vtex.events-example",
+      "keys": ["send-event"]
+    }
  }
}
```

> Note that we declare this by using the events resolver and the reference of the app that fires the event (declared as `sender`) and the event reference key (declared as `keys`).

6. At last, run `vtex link` and wait for the event to be fired by the `events-example` app. When listened, the log should appear on the console, like this:

![image](https://user-images.githubusercontent.com/43679629/83823425-5f323b00-a6aa-11ea-816a-68525e5800d7.png)