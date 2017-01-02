---
layout: post
title: 'Fun at Hack/Reduce!'
---

Last Saturday, I went to <a href="http://www.hackreduce.org/">Hack/Reduce</a>. Organized by the <a href="http://www.hopper.travel/about.html">folks at Hopper</a>, the event was an opportunity to learn to use <a href="http://hadoop.apache.org/">Hadoop</a>. They took the time to prepare a EC2 cluster of more than a hundred nodes. They also loaded a set of popular dataset for us to play with. With clear instructions on how to deploy our jobs to the cluster, we were ready to hack! Each team had an idea about what they want to do with the data. The Hopper's guy were there to help us realize it!

I teamed up with <a href="http://blog.mycila.com/">Mathieu Carbou</a> and <a href="http://twitter.com/#!/altus34">David Avenante</a> to build a <a href="http://en.wikipedia.org/wiki/Inverted_index">full inverted index</a> from the Wikipedia dataset. Our job used Mathieu's <a href="http://code.google.com/p/xmltool/">xmltool</a> to parse each Wikipedia page and some Lucene tokenizers to extract words and positions. I was a lot of fun to see this running on more than 400 cpus!

At the end of the day, each team took the time to present what they were able to accomplish. It was really impressing to see what was done in a single day! One team dig through flights data and discover that it is cheaper to travel on Friday and Saturday. Many teams also leveraged the bixi dataset to extract interesting information about Montrealers's usage of the bikes. Neat stuff!

I'd like to thank the Hopper's staff for such a nice event! Well done guys!
