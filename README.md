# Warehouse Layout Mapper (PWA)

Map a warehouse floor plan by walking the aisles and scanning location barcodes into a 2D grid on your phone.

| File | Purpose |
|---|---|
| `index.html` | The whole app: UI, Tailwind (CDN), html5-qrcode (CDN), and vanilla JS |
| `manifest.json` | Lets Android "Install app" / "Add to Home screen" |
| `sw.js` | Service worker that caches the app so it works offline once it has loaded |
| `icon.svg` | App icon |

`index.html` also works on its own. The other three files are only needed to install it as an app and use it offline.

## Using it
- **D-pad**: moves the blue cursor. Hold a button to repeat. Going past an edge adds a new row or column, including above or to the left of the current layout.
- **Tap a cell** to move the cursor there. **Tap the active cell** (or press **Edit**) to type a value by hand, pick a landmark (ENTRANCE, DOCK, PILLAR…), or insert/delete rows and columns.
- **Scan Barcode** opens the rear camera. Each read fills the active cell, beeps or vibrates, and moves the cursor in the **Auto-step** direction (tap the chip to cycle → ↓ ← ↑ Off). The camera stays on between scans unless you turn that off in ☰ Settings.
- **🔦** turns the flashlight on or off. It's only enabled when the camera supports it.
- **Duplicates**: if a code is already on the grid, it isn't placed, and a toast offers "Place anyway". Duplicates are outlined in red.
- **Undo**: up to 100 steps. Ctrl+Z also works on a keyboard.
- **Hardware scanners**: Zebra, Honeywell and similar devices in keyboard-wedge mode work without the camera. Their fast keystrokes ending in Enter are treated as a scan.
- **Saving**: every change is saved to `localStorage` straight away. The camera is released and state is saved when the screen locks or the app goes into the background. Reopening restores the grid, cursor and settings.
- **Export**: TSV or CSV, saved as a file, copied (pastes straight into Excel or Sheets) or shared. The output is only the grid: no headers, blank cells kept, trimmed to the filled area.
- **Import** (☰): loads an exported TSV/CSV back in, to continue on another phone or recover a layout.

> Data lives only in that phone's browser. Clearing site data deletes it, so export a backup regularly.

## Deploying to GitHub Pages (HTTPS, which the camera requires)
1. Create a new GitHub repository, e.g. `warehouse-mapper`.
2. Upload `index.html`, `manifest.json`, `sw.js` and `icon.svg` to the root of the repo, through the web UI (**Add file → Upload files**) or with git:
   ```bash
   git init && git add index.html manifest.json sw.js icon.svg
   git commit -m "Warehouse layout mapper"
   git branch -M main
   git remote add origin https://github.com/<you>/warehouse-mapper.git
   git push -u origin main
   ```
3. In the repo, go to **Settings → Pages → Build and deployment**, set Source to **Deploy from a branch**, pick `main` and `/ (root)`, then **Save**.
4. After about a minute the app is live at `https://<you>.github.io/warehouse-mapper/`. All paths are relative, so it works in that sub-folder.
5. On the phone, open the URL in Chrome, then **⋮ → Add to Home screen / Install app**.

**Publishing updates:** push the changed `index.html`. The service worker loads the network copy first, so phones pick up the new version the next time they open the app online. If you change `sw.js`, bump `CACHE = 'whs-mapper-v2'`.

## Testing
**Desktop (fastest):** browsers treat `localhost` as secure, so a laptop webcam works.
```bash
cd "Warehouse Layout Scanner"
python -m http.server 8000      # then open http://localhost:8000
```
- Use the arrow keys or D-pad to move. Use Edit to type values.
- Pretend to be a hardware scanner: click the page background and paste a code followed by Enter, or show a barcode to the webcam.
- Use Chrome DevTools → device toolbar (Ctrl+Shift+M) to see the phone layout.

**On an Android phone before deploying:** a phone opening `http://<laptop-ip>:8000` is **not** secure, so the camera will be blocked. Use one of these instead:
- **USB port forwarding:** turn on USB debugging on the phone, open `chrome://inspect/#devices` on the laptop, enable *Port forwarding* `8000 → localhost:8000`, then open `http://localhost:8000` on the phone.
- **HTTPS tunnel:** `npx localtunnel --port 8000` or `cloudflared tunnel --url http://localhost:8000`, then open the https URL it prints.
- Or deploy to GitHub Pages and test there.

**Checklist:**
1. Tap Scan, allow the camera, scan 3 labels. They should fill cells left to right, beeping each time.
2. Scan the same label again. You should get the duplicate warning.
3. Turn the flashlight on and off.
4. Mark Entrance on a cell. Move left past column A. A new column should appear.
5. Lock the phone, unlock it, and swipe the app away. When you reopen it, everything should still be there ("Session restored").
6. Export TSV, open it in Excel or Sheets, and check the cells match the on-screen grid exactly.
7. Turn on airplane mode and reopen the installed app. It should still load and scan (after one earlier online visit).

**Notes:**
- Excel drops leading zeros from codes like `00123` when it opens a CSV. To keep them, use **Copy** and paste, or import the file as text.
- 1D barcodes (Code 128/39, EAN, UPC, ITF) scan best held horizontally across the red aim line, 10–25 cm from the lens. Android Chrome uses its built-in barcode detector, which is fast.
