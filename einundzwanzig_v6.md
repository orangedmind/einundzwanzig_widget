Einundzwanzig Widget
Value4Value: FlashmanBTC@getalby.com

<img src="./images/Einundzwanzig_v5_pic1.jpg" width=33% height=33%/> <img src="./images/Einundzwanzig_v5_pic2.jpg"  width=33% height=33%/> <img src="./images/Einundzwanzig_v5_pic3.jpg" width=33% height=33%/>

V4 Blockheight 873710 + 873858 new Pictures / Code cleaned
1. Install the app "Scriptable" -> [Apple Appstore - Scriptable](https://apps.apple.com/ch/app/scriptable/id1405459188?l=en)
2. Open the app and click the "+" sign on the top right corner
3. Paste the following script created by [FlashmanBTC](https://twitter.com/FlashmanBTC):
4. You can edit scale if you have a smaller device. Tested it with iPhone 11 Pro and iPhone SE 2020

```js
// Variables used by Scriptable.
// These must be at the very top of the file. Do not edit.
// icon-color: deep-gray; icon-glyph: bolt;
// Einundzwanzig Edition by FlashmanBTC
// Updated by Orangedmind (2026-07): parallel requests, timeout handling, status indicators

// Font scaling for smaller displays
// Scale  0 = iPhone 11 Pro
// Scale -4 = iPhone SE 2020
scale = 0

// Change currency EUR or USD or CHF
currency = "EUR"

// Change fee order
// 1 = high to low
h_to_l = 0

// Timeout per request in seconds
const TIMEOUT_SEC = 6

// Show infos
// Attention max 5!, 1 = on
show_block  = 1
show_fees   = 1
show_moscow = 1
show_price  = 1
show_supply = 1
show_hash   = 0
show_diff   = 0

// Helper: fetch with individual timeout
// Returns { ok: true/false, value: ... }
async function fetchWithTimeout(url, type = 'string') {
  return new Promise(async (resolve) => {
    let timedOut = false
    const timer = Timer.schedule(TIMEOUT_SEC * 1000, false, () => {
      timedOut = true
      resolve({ ok: false, value: null })
    })
    try {
      const req = new Request(url)
      let value
      if (type === 'json')   value = await req.loadJSON()
      if (type === 'string') value = await req.loadString()
      if (type === 'image')  value = await req.loadImage()
      if (!timedOut) {
        timer.invalidate()
        resolve({ ok: true, value })
      }
    } catch(e) {
      if (!timedOut) {
        timer.invalidate()
        resolve({ ok: false, value: null })
      }
    }
  })
}

// Request data - all parallel, each with individual timeout
const [
  resLogo,
  resHeight,
  resFees,
  resMoscow,
  resPrice,
  resSupply,
  resHash,
  resDiff
] = await Promise.all([
  fetchWithTimeout('https://i.ibb.co/MSSJYtq/Einundzwanzig-logo.png', 'image'),
  fetchWithTimeout('https://mempool.space/api/blocks/tip/height', 'string'),
  fetchWithTimeout('https://mempool.space/api/v1/fees/recommended', 'json'),
  fetchWithTimeout('https://blockchain.info/tobtc?currency='+currency+'&value=1', 'string'),
  fetchWithTimeout('https://mempool.space/api/v1/prices', 'json'),
  show_supply == 1
    ? fetchWithTimeout('https://blockchain.info/q/totalbc', 'string')
    : Promise.resolve({ ok: true, value: null }),
  show_hash == 1
    ? fetchWithTimeout('https://mempool.space/api/v1/mining/hashrate/3d', 'json')
    : Promise.resolve({ ok: true, value: null }),
  show_diff == 1
    ? fetchWithTimeout('https://mempool.space/api/v1/difficulty-adjustment', 'json')
    : Promise.resolve({ ok: true, value: null })
])

// Blockheight
let blockHeight = '⚠️ n/a'
if (resHeight.ok) {
  let raw = resHeight.value.trim()
  let position_block = raw.length-3
  blockHeight = [raw.slice(0, position_block), " ", raw.slice(position_block)].join('')
}

// Mempool Fees
fast = '?'; halfHour = '?'; hour = '?'; fastNum = 0
if (resFees.ok) {
  fast     = resFees.value.fastestFee.toString()
  halfHour = resFees.value.halfHourFee.toString()
  hour     = resFees.value.hourFee.toString()
  fastNum  = resFees.value.fastestFee
}
let feesDisplay = resFees.ok
  ? (h_to_l == 1
      ? fast + ' H | ' + halfHour + ' M | ' + hour + ' L'
      : hour + ' L | ' + halfHour + ' M | ' + fast + ' H')
  : '⚠️ n/a'

// Moscow Time
let MoscowTime = '⚠️ n/a'
if (resMoscow.ok) {
  MoscowTime = Number(resMoscow.value).toFixed(8)
  MoscowTime = MoscowTime.substring(6)
  let position_moscow = MoscowTime.length-2
  MoscowTime = [MoscowTime.slice(0, position_moscow), ":", MoscowTime.slice(position_moscow)].join('')
}

// Shitcoin/BTC
let Shitcoin = '⚠️ n/a'
if (resPrice.ok) {
  if(currency == "EUR")
    Shitcoin = resPrice.value.EUR.toString()
  else if(currency == "USD")
    Shitcoin = resPrice.value.USD.toString()
  else
    Shitcoin = resPrice.value.CHF.toString()
}

// Bitcoin supply
let Supply = '⚠️ n/a'
if (show_supply == 1 && resSupply.ok)
  Supply = String(Math.round((parseInt(resSupply.value) / 100000000) * 100) / 100)

// Bitcoin hashrate
let HashInExa = '⚠️ n/a'
if (show_hash == 1 && resHash.ok && resHash.value) {
  let Hashvalue = resHash.value.hashrates[2].avgHashrate.toString()
  let HashExa = BigInt(Hashvalue)
  let Exa = 10n ** 18n
  HashInExa = (HashExa / Exa).toString()
}

// Difficulty adjustment
change = '⚠️'; rblocks = '⚠️'
if (show_diff == 1 && resDiff.ok && resDiff.value) {
  change  = resDiff.value.difficultyChange.toString()
  rblocks = resDiff.value.remainingBlocks.toString()
}

// Status indicator: 🟢 all ok | 🟡 partial | 🔴 all failed
const now = new Date()
const timeStr = now.getHours().toString().padStart(2,'0') + ':' + now.getMinutes().toString().padStart(2,'0')
const checks = [
  show_block  ? resHeight.ok  : null,
  show_fees   ? resFees.ok    : null,
  show_moscow ? resMoscow.ok  : null,
  show_price  ? resPrice.ok   : null,
  show_supply ? resSupply.ok  : null,
  show_hash   ? resHash.ok    : null,
  show_diff   ? resDiff.ok    : null,
].filter(v => v !== null)
const allOk  = checks.every(v => v === true)
const noneOk = checks.every(v => v === false)
const statusIcon = noneOk ? '🔴' : allOk ? '🟢' : '🟡'

let widget = await createWidget()

// Check where the script is running
if (config.runsInWidget) {
  // Runs inside a widget so add it to the homescreen widget
  Script.setWidget(widget)
} else {
  // Show the medium widget inside the app
  widget.presentLarge()
}

Script.complete()

async function createWidget() {
  // Create new empty ListWidget instance
  let listwidget = new ListWidget()
  // Refresh widget
  let nextRefresh = Date.now() + 1000*60
  listwidget.refreshAfterDate = new Date(nextRefresh)

  // Set new background color
  listwidget.backgroundColor = new Color("#151515")

  // Einundzwanzig Logo
  if (resLogo.ok)
    listwidget.addImage(resLogo.value).centerAlignImage()
  else {
    let fallback = listwidget.addText('₿ Einundzwanzig')
    fallback.centerAlignText()
    fallback.font = Font.boldSystemFont(18+scale)
    fallback.textColor = new Color("#F7931A")
  }

  listwidget.addSpacer(6)

  // Status line: freshness indicator + timestamp of last render
  let statusLine = listwidget.addText(statusIcon + ' ' + timeStr)
  statusLine.centerAlignText()
  statusLine.font = Font.systemFont(11+scale)
  statusLine.textColor = new Color("#888888")

  listwidget.addSpacer(8)

  // Blockheight
  if(show_block == 1) {
    let blockTitel = listwidget.addText("Blockheight")
    blockTitel.centerAlignText()
    blockTitel.font = Font.boldSystemFont(16+scale)
    blockTitel.textColor = new Color("#FFFFFF")
    let block = listwidget.addText(blockHeight)
    block.centerAlignText()
    block.font = Font.boldSystemFont(40+scale)
    block.textColor = resHeight.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Mempool Fees
  if(show_fees == 1) {
    let feesTitel = listwidget.addText("Mempool Fees")
    feesTitel.centerAlignText()
    feesTitel.font = Font.boldSystemFont(16+scale)
    feesTitel.textColor = new Color("#FFFFFF")
    if(h_to_l == 1)
      fees = listwidget.addText(fast + " H | " + halfHour + " M | " + hour + " L")
    else
      fees = listwidget.addText(hour + " L | " + halfHour + " M | " + fast + " H")
    fees.centerAlignText()
    if(fastNum < 10)
      fees.font = Font.boldSystemFont(40+scale)
    else if(fastNum < 100)
      fees.font = Font.boldSystemFont(36+scale)
    else
      fees.font = Font.boldSystemFont(30+scale)
    fees.textColor = resFees.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Moscow Time
  if(show_moscow == 1) {
    let moscowTitel = listwidget.addText("Moscow Time")
    moscowTitel.centerAlignText()
    moscowTitel.font = Font.boldSystemFont(16+scale)
    moscowTitel.textColor = new Color("#FFFFFF")
    let moscowTime = listwidget.addText(MoscowTime)
    moscowTime.centerAlignText()
    moscowTime.font = Font.boldSystemFont(32+scale)
    moscowTime.textColor = resMoscow.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Shitcoin/BTC
  if(show_price == 1) {
    let shitcoinTitel = listwidget.addText(currency+"/BTC")
    shitcoinTitel.centerAlignText()
    shitcoinTitel.font = Font.boldSystemFont(16+scale)
    shitcoinTitel.textColor = new Color("#FFFFFF")
    let shitcoin = listwidget.addText(Shitcoin)
    shitcoin.centerAlignText()
    shitcoin.font = Font.boldSystemFont(24+scale)
    shitcoin.textColor = resPrice.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Bitcoin supply
  if(show_supply == 1) {
    let supplyTitel = listwidget.addText("Supply")
    supplyTitel.centerAlignText()
    supplyTitel.font = Font.boldSystemFont(16+scale)
    supplyTitel.textColor = new Color("#FFFFFF")
    let supply = listwidget.addText(Supply)
    supply.centerAlignText()
    supply.font = Font.boldSystemFont(24+scale)
    supply.textColor = resSupply.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Bitcoin hashrate
  if(show_hash == 1) {
    let hashTitel = listwidget.addText("Hashrate")
    hashTitel.centerAlignText()
    hashTitel.font = Font.boldSystemFont(16+scale)
    hashTitel.textColor = new Color("#FFFFFF")
    let hash = listwidget.addText(HashInExa + " EH/s")
    hash.centerAlignText()
    hash.font = Font.boldSystemFont(24+scale)
    hash.textColor = resHash.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Difficulty adjustment
  if(show_diff == 1) {
    let diffTitel = listwidget.addText("Difficulty adjustment")
    diffTitel.centerAlignText()
    diffTitel.font = Font.boldSystemFont(16+scale)
    diffTitel.textColor = new Color("#FFFFFF")
    diff = listwidget.addText(change + " % | " + rblocks + " Blocks")
    diff.centerAlignText()
    diff.font = Font.boldSystemFont(30+scale)
    diff.textColor = resDiff.ok ? new Color("#F7931A") : new Color("#555555")
  }

  // Return the created widget
  return listwidget
}
```

5. Click on the bottom left corner the "sliders" to name your script. For example: Einundzwanzig
6. Click close and done
7. Go to the homescreen, press and hold for a few seconds to make the icons move. Tab on the top left corner the "+" symbol

<img src="./images/add_widget.jpg" style="zoom: 20%;" />

7. Scroll down untill you find the "Scriptable" App. Select it and scroll to the right for the full sized version.

<img src="./images/search_widget.jpg" style="zoom: 20%;" />

8. Click "Add Widget" and tab the new created widget to edit it. Select the created script and you're done :D

<img src="./images/create_widget.png" style="zoom: 20%;" />

<img src="./images/add_script.jpg" style="zoom: 20%;" />
