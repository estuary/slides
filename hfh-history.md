---
title: Estuary Flow
separator: <!--s-->
verticalSeparator: <!--v-->
theme: league
scripts: 'lib/mermaid.min.js,lib/reveal-mermaid.js'
revealOptions:
  transition: 'fade'

---

## Why History Matters

What happens today...

Note: speaking of history, let's talk about why this is such a problem today.

<!--s-->

A typical λ architecture.

```mermaid
graph LR;
  src((API))
  ps[/Pub/Sub/]
  wh[(S3)]
  a>Analysts]
  t1{{Transform}}
  db1[(Redis)]

  src --> ps;
  ps --> wh;
  ps --> t1;

  subgraph Lake
  wh --> a
  end

  subgraph Serving
  t1 --> db1;
  end

```

Note: now classic "lambda" architecture.

<!--s-->

New use case! 🚀

```mermaid
graph LR;
  src((API))
  ps[/Pub/Sub/]
  wh[(S3)]
  a>Analysts]
  t1{{Transform}}
  db1[(Redis)]
  t2{{"🚀"}}
  db2[(PostgreSQL)]

  src --> ps;
  ps --> wh;
  ps --> t1;
  ps --> t2;

  subgraph Lake
  wh --> a
  end

  subgraph Serving
  t1 --> db1;
  t2 --> db2;
  end

```

<!--s-->

Uh oh...

```mermaid
graph LR;
  src((API))
  ps[/Pub/Sub/]
  wh[(S3)]
  a>Analysts]
  t1{{Transform}}
  db1[(Redis)]
  t2{{???}}
  db2[(PostgreSQL)]

  src --> ps;
  ps --> wh;
  ps --> t1;
  ps --> t2;

  subgraph Lake
  wh --> a
  end

  subgraph Serving
  t1 --> db1;
  t2 --> db2;
  end

  jb[just a buffer...] -.-> ps;
  dlh[data lives here!] -.-> wh;

```

<!--s-->

Must replay through Pub/Sub 🤮

```mermaid
graph LR;
  src((API))
  ps[/Pub/Sub/]
  wh[(S3)]
  a>Analysts]
  t1{{Transform}}
  db1[(Redis)]
  t2{{???}}
  db2[(PostgreSQL)]

  src --> ps;
  ps --> wh;
  ps --> t1;
  ps --> t2;

  subgraph Lake
  wh --> a
  end

  subgraph Serving
  t1 --> db1;
  t2 --> db2;
  end

  wh -.-> rp((Replay Job))
  rp -.-> ps
  ps -.-> t2

```

History vs "now" must be manually stitched.<br>
Mis-orders, duplicates, drops are likely.

<!--s-->

How Flow Helps 🌈 

```mermaid
graph LR;
  ps[/Pub/Sub/]
  wh[(S3)]
  t1{{Transform}}
  db1[(Redis)]
  t2{{"🚀"}}
  db2[(PostgreSQL)]

  ps --> Capture;

  subgraph Flow
  Capture --> t1;
  Capture --> t2;
  end

  subgraph Stores
  Capture --> wh;
  t1 --> db1;
  t2 --> db2;
  wh -.-> t2
  end
```

🚀 reads history right from S3,<br>
and seamlessly transitions to live updates.

Note: We've grouped components here as Flow-managed concerns, versus user-managed stores.

<!--s-->

But wait, there's more...

```mermaid
graph LR;
  ps[/Pub/Sub/]
  wh[(S3)]
  t2{{"🚀"}}
  db2[(PostgreSQL)]
  db3[(CosmosDB)]

  ps --> Capture;
  Capture --> t2;

  Capture --> wh;
  t2 --> db2;
  t2 ==> db3;
  wh -.-> t2
```

Materialize views where and _when_ you want.<br>
New instances back-fill, and thereafter stay in sync.

