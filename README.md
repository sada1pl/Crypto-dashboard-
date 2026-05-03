# CryptoDesk – Bitget Trading Dashboard
## Aplikacja Android (PWA)

### ⚡ Jak uruchomić na Androidzie

#### Opcja 1 – GitHub Pages (DARMOWE, polecane)
1. Utwórz konto na github.com
2. Nowe repozytorium → prześlij pliki (`index.html`, `manifest.json`, `sw.js`)
3. Settings → Pages → Source: main branch
4. Aplikacja dostępna pod: `https://TWOJA-NAZWA.github.io/REPO/`
5. Na Androidzie wejdź w Chrome → trzy kropki → **"Dodaj do ekranu głównego"**
6. Aplikacja zainstaluje się jak natywna (ikona, fullscreen, offline)

#### Opcja 2 – Netlify (DARMOWE, drag & drop)
1. Wejdź na netlify.app
2. Przeciągnij folder z plikami
3. Gotowe! Dostaniesz link https://xxx.netlify.app

#### Opcja 3 – Localhost (testy)
```bash
npx serve .
# Wejdź: http://localhost:3000
```

---

### 🔑 Konfiguracja Bitget API

1. Wejdź na **bitget.com** → Profil → API Management
2. Utwórz nowy klucz z uprawnieniami: **Read Only**
3. Skopiuj: API Key, Secret Key, Passphrase
4. W aplikacji kliknij **"API Setup"** i wprowadź dane
5. Dane są przechowywane lokalnie w przeglądarce (localStorage)

> ⚠️ **Bezpieczeństwo:** Aplikacja używa kluczy Read Only – nie może składać zleceń.
> Klucze są zapisane wyłącznie na Twoim urządzeniu.

---

### 🏗️ Architektura (Production Backend)

Dla pełnej integracji Bitget API (podpisywanie HMAC-SHA256) potrzebny jest backend proxy:

```javascript
// Przykład backend proxy (Node.js/Express)
const crypto = require('crypto');

function signBitget(timestamp, method, path, body, secret) {
  const msg = timestamp + method + path + (body || '');
  return crypto.createHmac('sha256', secret).update(msg).digest('base64');
}

app.get('/api/bitget/balance', async (req, res) => {
  const ts = Date.now().toString();
  const sign = signBitget(ts, 'GET', '/api/v2/spot/account/assets', '', SECRET);
  const response = await fetch('https://api.bitget.com/api/v2/spot/account/assets', {
    headers: {
      'ACCESS-KEY': API_KEY,
      'ACCESS-SIGN': sign,
      'ACCESS-TIMESTAMP': ts,
      'ACCESS-PASSPHRASE': PASSPHRASE,
    }
  });
  res.json(await response.json());
});
```

**Deployment backend:** Railway.app, Render.com lub VPS (darmowe tier)

---

### 📱 Funkcje aplikacji

| Funkcja | Status |
|---------|--------|
| Ceny rynkowe (CoinGecko) | ✅ Live (100 coinów) |
| Wyszukiwarka | ✅ |
| Filtry (Zyski/Straty/Top10) | ✅ |
| Lista obserwowanych | ✅ (localStorage) |
| Portfolio Dashboard | ✅ Demo / ✅ Bitget API |
| Wykres 7D/30D/vs BTC | ✅ |
| Historia transakcji Spot | ✅ Demo / ✅ Bitget API |
| Historia Futures | ✅ Demo / ✅ Bitget API |
| Śledzenie opłat | ✅ Demo / ✅ Bitget API |
| Wykresy miesięczne opłat | ✅ |
| Alerty cenowe | ✅ (push notifications) |
| Dark mode | ✅ (zawsze) |
| Offline mode (PWA) | ✅ Service Worker |
| Dodawanie do ekranu głównego | ✅ Android Chrome |
| Push Notifications | ✅ (wymaga HTTPS) |

---

### 🔧 Rozszerzenie do natywnej apki

Aby zbudować natywną APK z tej aplikacji webowej:

```bash
# Capacitor (Ionic)
npm install @capacitor/core @capacitor/cli
npx cap init "CryptoDesk" "com.cryptodesk.app"
npx cap add android
npx cap sync
npx cap open android
# → Android Studio → Build → Generate APK
```

---

### 📡 Używane API

- **CoinGecko API** (darmowe, 30 req/min) – ceny rynkowe
- **Bitget REST API v2** – saldo, historia, orders
  - Spot: `/api/v2/spot/account/assets`
  - Futures: `/api/v2/mix/account/account`
  - Historia: `/api/v2/spot/trade/fills`
  - Fees: `/api/v2/spot/account/tradeFee`
