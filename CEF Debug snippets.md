
## Intercept ALL SteamClient calls to find what IPC fires

```javascript
const _orig = SteamClient.sendMessage ?? SteamClient.Send
// OR spy on the Apps namespace
Object.keys(SteamClient).forEach(ns => {
  Object.keys(SteamClient[ns] ?? {}).forEach(fn => {
    if (typeof SteamClient[ns][fn] === 'function') {
      const orig = SteamClient[ns][fn].bind(SteamClient[ns])
      SteamClient[ns][fn] = (...args) => {
        console.log(`SteamClient.${ns}.${fn}`, args)
        return orig(...args)
      }
    }
  })
})
```

## Apply Steam Input community layout

```javascript
let controllerIndex = 0 // from query connectedControllers

await SteamClient.Input.GetConfigForAppAndController(appId, i)
await SteamClient.Apps.DownloadWorkshopItem(241100, workshopItemId, true)
SteamClient.Input.SetSelectedConfigForApp(appId, controllerIndex, `workshop://${workshopItemId}`, false, true)
```

## Query connected controllers

```javascript
 window.ControllerStore.m_controllerList.map(c =>
    JSON.parse(JSON.stringify(c, (_, val) => typeof val === 'bigint' ? val.toString() : val))
  );
```

## Show Controller configurator for AppID

```javascript
await SteamClient.Apps.ShowControllerConfigurator(appId)
```
