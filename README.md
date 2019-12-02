# datadler-gateway

Ein konfigurierbares In-Memory[Dat](https://datproject.org/)-zu-HTTP-Gateway, so dass Sie Dat-Archive von Ihrem Browser aus besuchen können.

Wenn Sie einen Browser benötigen, der Dat-Archive besuchen kann, besuchen Sie[Beaker](https://beakerbrowser.com/).
## Install

Um den Befehl `datadler-gateway` zum Ausführen eines eigenen Gateways zu erhalten, verwenden Sie[`npm`](https://www.npmjs.com/):


```
npm i -g datadler-gateway
```

If you have [`npx`](https://github.com/zkat/npx) installed, it's even shorter:

```
npx datadler-gateway
```

## Öffentliche Gateways:

- http://gateway.mauve.moe:3000/ (Hosted by @RangerMauve)
- https://pamphlets.me/ (Hosted by @brechtcs)
- https://dat.bovid.space/ (Original gateway from @garbados)
- https://dat.hypersource.club (Hosted by @jwerle)

## Verwendung

Sie können `datadler-gateway` ausführen, um einen Gateway-Server zu starten, der auf Port 3000 lauscht. Sie können es auch konfigurieren! Sie können Nutzungsinformationen mit `datadler-gateway -h` ausdrucken:

```
$ datadler-gateway -h
datadler-gateway

Options:
  --version       Anzeigen der Versionsnummer                                  [boolean]
  --config        Pfad zur JSON-Konfigurationsdatei
  --host, -l      Host oder ip für das zugegriffene Gateway.        [default: "0.0.0.0"]
  --port, -p      Port for the gateway to listen on.                     [default: 3000]
  --dir, -d       Verzeichnis, das als Cache verwendet werden soll.
                                                    [string] [default: "~/.dat-gateway"]
  --max, -m       Maximale Anzahl von Archiven, die im Cache erlaubt sind. [default: 20]
  --period        Anzahl der Millisekunden zwischen dem Bereinigen des Cache von 
                  abgelaufenen Archiven.                                [default: 60000]
  --ttl, -t       Anzahl der Millisekunden, bevor die Archive ablaufen.
                                                                       [default: 600000]
  --redirect, -r  Ob Subdomain-Umleitungen verwendet werden sollen      [default: false]
  -h, --help      Hilfe                                                        [boolean]
```

Sie können Dat-Archive über das Gateway auf einer solchen Art und Weise besuchen:

```
http://localhost:3000/{datKey}/{path...}
```

Ein Beispiel:

```
http://localhost:3000/40a7f6b6147ae695bcbcff432f684c7bb5291ea339c28c1755896cdeb80bd2f9/assets/img/beaker-0.7.gif
```

Das Gateway löst sogar URLs mit[Dat-DNS](https://github.com/beakerbrowser/beaker/wiki/Authenticated-Dat-URLs-and-HTTPS-to-Dat-Discovery) auf:

```
http://localhost:3000/garbados.hashbase.io/icons/favicon.ico
```

Das Gateway überprüft Archive, bis sie aus dem Cache auslaufen, und hält sie dann proaktiv an und löscht sie von der Festplatte.

Das Gateway unterstützt auch die Replikation einer Hyperdrive-Instanz über[websockets](https://github.com/maxogden/websocket-stream).

```javascript
const websocket = require('websocket-stream')
const hyperdrive = require('hyperdrive')

const key = 'c33bc8d7c32a6e905905efdbf21efea9ff23b00d1c3ee9aea80092eaba6c4957'
const url = `ws://localhost:3000/${key}`

const archive = hyperdrive('./somewhere', key)

archive.once('ready', () => {
  const socket = websocket(url)

  // Replicate through the socket
  socket.pipe(archive.replicate()).pipe(socket)
})
```

## Umleitung von Subdomains

Standardmäßig bedient das Daten-Gateway alle Dats vom gleichen Ursprung. Das bedeutet, dass Dats, die absolute URLs verwenden (beginnend mit `/`), beschädigt werden.
Dies bedeutet auch, dass alle Dats die gleichen localStorage- und indexedDB-Instanzen verwenden, was zu Sicherheitsproblemen führen kann.

Um diese Probleme zu beheben, können Sie das `--redirect` Flag in Verbindung mit dem `host` Parameter verwenden, um jedes dat auf einer Subdomain servieren zu lassen.

So wird beispielsweise `http://{host}:{port}/{datkey}/index.html` zu `http://{datkey32}.{host}:{port}/index.html` umgeleitet, das die Datei von localhost, aber in einer anderen Domäne bedient, wodurch sichergestellt wird, dass der Browser alle Inhalte voneinander isoliert.

Bitte beachten Sie, dass aufgrund von Einschränkungen bei der Funktionsweise von URLs der dat-Schlüssel in seine base32-Darstellung anstelle von hexadezimal mit[dieser Bibliothek](https://github.com/RangerMauve/hex-to-32) umgewandelt wird.


## License

[Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0)
