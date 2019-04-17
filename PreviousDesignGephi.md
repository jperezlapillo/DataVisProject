---
id: litvis

narrative-schemas:
  - ../narrative-schemas/courseworkPG.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

### INM402 Postgraduate Coursework:
# Previous design choices using Gephi
**Author: Joaquin Perez-Lapillo**

This markdown file includes previous design choices created using Gephi, which proved not to be useful for networks with a high number of nodes.

{(visualization|}

### 1. Network graph of the Chilean economy structure (111-industries)

![preview_detail](/assets/preview_detail.png)

### 2. Interactive network graph of the Chilean economy structure (111-industries)

![screenshot_165638](/assets/screenshot_165638.png)

{|visualization)}
