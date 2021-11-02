---
title: 'Feedback wanted: an experimental responsiveness metric'
subhead: An update on our plans for measuring responsiveness on the web.
description: An update on our plans for measuring responsiveness on the web.
authors:
  - hbsong
date: 2021-11-03
hero: image/eqprBhZUGfb8WYnumQ9ljAxRrA72/GNWyQZ5l1uy1b7s5ursN.jpeg
alt: Water droplet rippling outward
tags:
  - blog # blog is a required tag for the article to show up in the blog.
  - performance
  - web-vitals
---

Earlier this year, the Chrome Speed Metrics Team shared [some of the
ideas](/better-responsiveness-metric/) we were considering for a
new responsiveness metric. We want to design a metric that better captures the
end-to-end latency of individual events and offers a more holistic picture of
the overall responsiveness of a page throughout its lifetime.

We've made a lot of progress on this metric in the last few months, and we
wanted to share an update on how we plan to measure interaction latency as well
as introduce a few specific aggregation options we're considering to quantify
the overall responsiveness of a web page.

We'd love to get [feedback](#feedback) from developers and site owners as to
which of these options would be most representative of the overall input
responsiveness of their pages.

## Measure interaction latency

As a review, the [First Input Delay (FID)](/fid/) metric captures the
[delay](/fid/#fid-in-detail) portion of input latency. That is, the time between
when the user interacts with the page to the time when the event handlers are
able to run.

With this new metric we plan to expand that to capture the [full event
duration](/better-responsiveness-metric/#capture-the-full-event-duration), from
initial user input until the next frame is painted after all the event handlers
have run.

We also plan to measure
[interactions](/better-responsiveness-metric/#group-events-into-interactions)
rather than individual events. Interactions are groups of events that are
dispatched as part of the same, logical user gesture (for example:
`pointerdown`, `click`, `pointerup`).

To measure the total interaction latency from a group of individual event
durations, we are considering two potential approaches:

- **Maximum event duration:** the interaction latency is equal to the largest
  single event duration from any event in the interaction group.
- **Total event duration:** the interaction latency is the sum of all event
  durations, ignoring any overlap.

As an example, the diagram below shows a _key press_ interaction that consists
of a `keydown` and a `keyup` event. In this example there is a duration overlap
between these two events. To measure the latency of the key press interaction,
we could use `max(keydown duration, keyup duration)` or `sum(keydown duration,
keyup duration) - duration overlap`:

{% Img src="image/eqprBhZUGfb8WYnumQ9ljAxRrA72/F55OBEhX6VvNEApTjldU.svg",
alt="ALT_TEXT_HERE", width="800", height="424" %}

There are pros and cons of each approach, and we'd like to collect more data and
[feedback](#feedback) before finalizing a latency definition.

{% Aside %}
  The event duration is meant to be the time from the event hardware timestamp
  to the time when the next paint is performed after the event is handled. But
  if the event doesn't cause any update, the duration will be the time from
  event hardware timestamp to the time when we are sure it will not cause any
  update.
{% endAside %}

{% Aside %}
  For keyboard interactions, we usually measure the `keydown` and `keyup`. But
  for IME, such as input methods for Chinese and Japanese, we measure the input
  events between a `compositionstart` and a `compositionend`.
{% endAside %}

## Aggregate all interactions per page

Once we're able to measure the end-to-end latency of all interactions, the next
step is to define an aggregate score for a page visit, which may contain more
than one interaction.

After exploring a number of options, we've narrowed our choices down to the
strategies outlined in the following section, each of which we're currently
collecting real-user data on in Chrome. We plan to publish the results of our
findings once we've had time to collect sufficient data, but we're also looking
for direct [feedback](#feedback) from site owners as to which strategy would
most accurately reflect the interaction patterns on their pages.

### Aggregation strategies options

To help explain each of the following strategies, consider an example page visit
that consists of four interactions:

<div class="w-table-wrapper">
  <table>
    <tr>
      <th>Interaction</th>
      <th style="text-align: right">Latency</th>
    </tr>
    <tr>
      <td>Click</td>
      <td style="text-align: right">120 ms</td>
    </tr>
    <tr>
      <td>Click</td>
      <td style="text-align: right">20 ms</td>
    </tr>
    <tr>
      <td>Key press</td>
      <td style="text-align: right">60 ms</td>
    </tr>
    <tr>
      <td>Key press</td>
      <td style="text-align: right">80 ms</td>
    </tr>
  </table>
</div>

#### Worst interaction latency

The largest, individual interaction latency that occurred on a page. Given the
example interactions listed above, the worst interaction latency would be 120
ms.

#### Budgets strategies

[User experience
research](https://www.tactuallabs.com/papers/howMuchFasterIsFastEnoughCHI15.pdf)
suggests that users may not perceive latencies below certain thresholds as
negative. Based on this research we're considering several budget strategies
using on the following thresholds for each event type:

<div class="w-table-wrapper">
  <table>
    <tr>
      <th>Interaction type</th>
      <th style="text-align: right">Budget threshold</th>
    </tr>
    <tr>
      <td>Click/tap</td>
      <td style="text-align: right">100 ms</td>
    </tr>
    <tr>
      <td>Drag</td>
      <td style="text-align: right">100 ms</td>
    </tr>
    <tr>
      <td>Keyboard</td>
      <td style="text-align: right">50 ms</td>
    </tr>
  </table>
</div>

Each of these strategies will only consider the latency that is more than the
budget threshold per interaction. Using the example page visit above, the
over-budget amounts would be as follows:

<div class="w-table-wrapper">
  <table>
    <tr>
      <th>Interaction</th>
      <th style="text-align: right">Latency</th>
      <th style="text-align: right">Latency over budget</th>
    </tr>
    <tr>
      <td>Click
    </td>
      <td style="text-align: right">120 ms</td>
      <td style="text-align: right">20 ms</td>
    </tr>
    <tr>
      <td>Click
    </td>
      <td style="text-align: right">20 ms</td>
      <td style="text-align: right">0 ms</td>
    </tr>
    <tr>
      <td>Key press
    </td>
      <td style="text-align: right">60 ms</td>
      <td style="text-align: right">10 ms</td>
    </tr>
    <tr>
      <td>Key press
    </td>
      <td style="text-align: right">80 ms</td>
      <td style="text-align: right">30 ms</td>
    </tr>
  </table>
</div>

#### Worst interaction latency over budget

The largest single interaction latency over budget. Using the above example, the
score would be `max(20, 0, 10, 30) = 30 ms`.

#### Total interaction latency over budget

The sum of all interaction latencies over budget. Using the above example, the
score would be `(20 + 0 + 10 + 30) = 60 ms`.

#### Average interaction latency over budget

The total over-budget interaction latency divided by the total number of
interactions. Using the above example, the score would be `(20 + 0 + 10 + 30) /
4 = 15 ms`.

#### High quantile approximation

As an alternative to computing the largest interaction latency over budget, we
also considered using a high quantile approximation, which should be fairer to
web pages that have a lot of interactions and may be more likely to have large
outliers. We've identified two potential high-quantile approximation strategies
we like:

- **Option 1:** keep track of the largest interaction over budget. After every
  50 interactions, discard the largest and replace it with the next largest.
- **Option 2:** compute the largest 10 interactions over budget and choose a
  value from that list depending on the total number of interactions. Given N
  total interactions, select the (N / 50 + 1)th largest value, or the 10th value
  for pages with more than 500 interactions.

## Measure these options in JavaScript

The following code example can be used to determine the values of the first
three strategies presented above. Note that it's not yet possible to measure the
total number of interactions on a page in JavaScript, so this example doesn't
include the average interaction over budget strategy or the high
quantile approximation strategies.

```js
const interactionMap = new Map();

let worstLatency = 0;
let worstLatencyOverBudget = 0;
let totalLatencyOverBudget = 0;

new PerformanceObserver((entries) => {
  for (const entry of entries.getEntries()) {
    // Ignore entries without an interaction ID.
    if (entry.interactionId > 0) {
      // Get the interaction for this entry, or create one if it doesn't exist.
      let interaction = interactionMap.get(entry.interactionId);
      if (!interaction) {
        interaction = {entries: []};
        interactionMap.set(entry.interactionId, interaction);
      }
      interaction.entries.push(entry);

      const latency = Math.max(entry.duration, interaction.latency || 0);
      worstLatency = Math.max(worstLatency, latency);

      const budget = entry.name.includes('key') ? 50 : 100;
      const latencyOverBudget = latency - budget;
      worstLatencyOverBudget =
          Math.max(latencyOverBudget, worstLatencyOverBudget);

      // If this event adds additional latency, update the total over budget.
      const newLatency = latency - (interaction.latency || 0);
      if (newLatency > 0) {
        totalLatencyOverBudget += newLatency;
      }

      // Set the latency on the interaction so future events can reference.
      interaction.latency = latency;

      // Log the updated metric values.
      console.log({
        worstLatency,
        worstLatencyOverBudget,
        totalLatencyOverBudget,
      });
    }
  }
}).observe({type: 'event', buffered: true, durationThreshold: 50});
```

{% Aside 'caution' %}
  There are currently [a few
  bugs](https://bugs.chromium.org/p/chromium/issues/list?q=label:responsiveness-bugs)
  in Chrome that affect accuracy of the reported interaction timestamps. We are
  working to fix these bugs as soon as possible, and we recommend developers
  test these strategies in Chrome Canary to get the most accurate results.
{% endAside %}

## Feedback

We want to encourage developers to try out these new layout shift metrics on
their sites, and let us know if you discover any issue.

Please also email any general feedback on the approaches outlined here to the
[web-vitals-feedback](https://groups.google.com/g/web-vitals-feedback) Google
group with "[Responsiveness Metrics]" in the subject line. We're really looking
forward to hearing what you think!