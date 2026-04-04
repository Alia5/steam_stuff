
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
const tryIndices = [0, 1, 2, 3]
let controllerIndex = 0

for (const i of tryIndices) {
  const config = await SteamClient.Input.GetConfigForAppAndController(appId, i)
  if (config?.bConfigurationEnabled) {
    controllerIndex = i
    break
  }
}

await SteamClient.Apps.DownloadWorkshopItem(241100, workshopItemId, true)
SteamClient.Input.SetSelectedConfigForApp(appId, controllerIndex, `workshop://${workshopItemId}`, false, true)
```

## Query connected controllers

```javascript
function getConnectedControllers() {
  return window.ControllerStore.m_controllerList.map(c =>
    JSON.parse(JSON.stringify(c, (_, val) => typeof val === 'bigint' ? val.toString() : val))
  );
}

console.log(getConnectedControllers());
```
