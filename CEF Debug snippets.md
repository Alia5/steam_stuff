# Steam stuff

Unless otherwise specified code is to be injected into `SharedJSContext`

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

## QUery Games

```javascript
((nonSteamOnly = false, installedOnly = false) => window.appStore.allApps
  .filter(a => (!nonSteamOnly || a.app_type === 1073741824) && (!installedOnly || a.installed))
  .map(a => ({ appid: a.appid, name: a.display_name, installed: a.installed, isNonSteam: a.app_type === 1073741824 }))
)(true, false)
```

## Open in new store browser window

Inject into to store tab (url includes `steampowered.com` or starts with `data:text/html`

```javascript
var el = document.createElement("a");
document.body.appendChild(el);
el.setAttribute("href", "https://steaminputdb.com")
el.dispatchEvent(new MouseEvent( "click", { "button": 1, "which": 2 }))
```

From UI/Library Tab

```
window.opener.SteamUIStore.ActiveWindowInstance.m_Navigator.SteamWebTab("https://steaminputdb.com")
```

SharedJSContext

```
SteamUIStore.ActiveWindowInstance.m_Navigator.SteamWebTab("https://steaminputdb.com")
```

**Notes:**

Just calling `open(URL)` from any context just opens the users default browser.  
*EXCEPT* from the Steam Big Picture Tab, where it just navigates to the web-page in BPM! :)

## Add button to Steam game library page (Desktop-UI)

Inject into tab titled `Steam`

```javascript
(() => {
  if (window.__sidbOpenButtonObserver) {
    return;
  }
  const ATTR = 'data-sidb-open-button-injected';

  const inject = () => {
    document.querySelectorAll('.SVGIcon_Settings').forEach((el) => {
      const container = el.parentElement.parentElement.parentElement;
      if (container.querySelector(`[${ATTR}]`)) {
        return;
      }
      const node = container.children[1].cloneNode();
      node.setAttribute(ATTR, '1');
      node.ariaLabel = "SteamInputDB";
      node.innerHTML = "<span>S</span>";
      node.onclick = () => open("https://steaminputdb.com");
      container.appendChild(node);
    });
  };

  inject();

  const observer = new MutationObserver(inject);
  observer.observe(document.body, { childList: true, subtree: true });

  window.__sidbOpenButtonObserver = observer;
})();
```

## Currently selected APPID In Library Tab

CEF UI Tab just named `Steam` (Desktop UI!)
```
window.opener.SteamUIStore.ActiveWindowInstance.m_locationPathname
'/library/app/4098358112'
```
