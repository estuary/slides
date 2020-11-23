---
title: 'Estuary Flow: Intro'
separator: <!--s-->
verticalSeparator: <!--v-->
theme: league
scripts: 'lib/mermaid.min.js,lib/reveal-mermaid.js'
revealOptions:
  transition: 'fade'
---

# Estuary <i>Flow</i>

<!--s-->

> Unifies technologies and teams around a shared understanding of
their data...

> ...that updates in milliseconds.

Note:
- Tool the helps orgs define and implement their "Data Mesh".
- What are our datasets and what are the systems that produce them,
  how can we transform them
  into new data products, how do we push those back out into other
  systems we care about (db, api, pub/sub), how to keep up to date?
- (transition)
- What's special: operating as low-latency, event-driven system
	-- add new data, figuring out changes/updates of derived data products & propagating immediately
	-- but it's one that interfaces natively with tools & ecosystems of MP batch analytical space (spark, snowflake).

<audio data-autoplay src="media/intro_intro.m4a"></audio>
<!--s-->

### 🧾 Complete History 

_Collections_ represent months or years of historical data right on cloud storage, plus ongoing updates.

Note:
 - A lot to talk about w/ Flow, but want to hit on some key features right away
 - When Flow is operating on a collection of data,
 	thinking in terms of complete history of collection + ongoing updates.
      If you've worked with pub/sub systems, that will immediately strike you as pretty unusual,
      and it's made possible through Flow's really efficient use of cloud storage.

<audio data-autoplay src="media/intro_complete_history.m4a"></audio>
<!--s-->

### ⚙️ Transform Without Limits

- Complex joins and aggregations.
- <u>Unbounded</u> look-back.
- <u>No</u> windowing constraints.
- <u>Dynamic</u> scaling w/o re-partitioning.


Note:

 - If you've played with systems offer S2S, may be aware that it comes with a bunch of caveats.
   - Happy to join events that occurred a minute apart
   - Start to break down if joining something now, against a month ago or more.
   - Aggregation is limited:
       - Can count page views in a windowed user session.
       - Not able to model cumulative lifetime value of customer over time.
   - Gnarly operational knobs:
       - For example, requiring that you declare partitions to use up front,
         and not having recourse except rebuilding the entire view if partitioning
	 turns out to be insufficient.
   - Hidden downtime; machine failure => a bunch of re-done work.

 - With flow, don't have to think about any of that.
   - join whatever you want
   - aggregate it however you want
   - flow runtime able to dynamically scale the workload,
     keep it running even if there are machine failures,
     w/ E2E exactly once semantics.
   
<audio data-autoplay src="media/intro_transform_without_limits.m4a"></audio>
<!--s-->

### 💨 Any DB is a "streaming" DB

Materialize views into SQL & NoSQL databases alike.<br>
Flow keeps them fresh with continuous map/reduce.

Note:
 - Finally, let's talk about materializations.
  - Common use case: take firehose of raw events, aggregate them into a Postgres fact table of dimensions we care about, and metrics accumulated from those events.
  - This is fundamentally a map/reduce problem: map each event into it's contribution to the fact table, and rolling it up.
  - Flow is a distributed, continuous map/reduce engine -- taking advantage of it's roll-up knowhow to combine data early and often in it's processing.
  - Observation: firehose turns into relative trickle of updates to fact table

<audio data-autoplay src="media/intro_any_db_is_streaming.m4a"></audio>
