# Encoding and Evolution

## Important Points:

* The implementation of Protocol Buffers (p117)
* Message-passing dataflow (p139)

## Important Care in Data Encoding:

* Backwards/forwards compatibility
* Rolling update, which means nodes can running different version application at the send time. It must ensure the new version application can read old data and old application can read new version data with no problem.
* Benefit: update without downtime, small but frequent update, partial update for A/B test
