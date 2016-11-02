---
layout: default
---

## Raw: CircleCI first impression

* SSH debugging is awesome. It just works, using my existing Github key. Awesome.
* Infers config from source code. Wasn't perfect for my Gradle build, but delightful first impression
* Spring boot app builds bit over 2mins, while local Jenkins 1:20. Neither of those caches gradle dependencies (20sec). CircleCIs Git cache had no real effect as it was a dummy repo.
* Nice UI
* Price seems okay, although Saas pricing is black magic: high degree of lock-in, and sometimes strange clauses
* Hot
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/760_RC08/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"Travis CI","geo":"","time":"all"},{"keyword":"CircleCI","geo":"","time":"all"},{"keyword":"Drone.io","geo":"","time":"all"}],"category":0,"property":""}, {"exploreQuery":"date=all&q=Travis%20CI,CircleCI,Drone.io"}); </script> 

--

 * Honest + *Drone was built primarily as an alternative to self-hosted Jenkins* [https://www.quora.com/What-are-the-core-differences-between-Travis-CI-and-drone-io](https://www.quora.com/What-are-the-core-differences-between-Travis-CI-and-drone-io)
 * CONs [https://www.slant.co/versus/2484/625/~drone-io_vs_circleci](https://www.slant.co/versus/2484/625/~drone-io_vs_circleci)
 * Comments [https://www.g2crowd.com/compare/circleci-vs-drone-io](https://www.g2crowd.com/compare/circleci-vs-drone-io)
