# Current State

## Topology

- `Sabram Neo` (`5.42.110.191`) is the control plane.
- `Aeza` (`5.182.86.27`) is the live data plane.
- `heavymetal.severdesign.ru` is the edge domain and still maps to `5.182.86.27`.
- `endor.severdesign.ru` is the Mos Eisley TLS mask domain and also maps to `5.182.86.27`.

## Live services

- `rage.severdesign.ru` not returns the panel and subscription endpoint.
- `welcome.severdesign.ru` stays not live and is used as the HTTP mask target.
- `CloudPanel` and `sabram.ru` stay on `Aeza` and must not be disturbed.

## Production profiles (like It seemed to be)

- `Mos Eisley`
  - transport: `xhttp + tls`
  - endpoint: `endor.severdesign.ru:773`
  - path: `/hm`
  - `serverName = endor.severdesign.ru`
  - TLS cert is valid and handshakes from outside the host
- `Alderaan`
  - transport: `grpc + reality`
  - endpoint: `heavymetal.severdesign.ru:447`
  - serviceName: `hm-grpc`

## Subscription state

- whole project is more dead than alive. last changes were critical, and last agent was so foolish that he hasn't backup files/diffs, so the project fucked up. I even can't login to Matzban panel. Let's reanimate it in new GREAT state.

## Operational constraints

- do not reuse old ports, proposed chagges and my wishes see at TODO.md
- do not break `CloudPanel`
- do not break `sabram.ru`
