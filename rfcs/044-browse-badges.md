- Feature Name: Badges on landing page & browse view
- Start Date: 2021-09-28
- RFC PR: [amundsen-io/rfcs#44](https://github.com/amundsen-io/rfcs/pull/44)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Badges on landing page & browse view

## Summary

This RFC proposes to introduce `Available badges` section above `Popular tags` on landing page and & browse section.*

! `Available/Popular` is either this or that choice that has been explained in `Unresolved questions` section.

## Motivation
After introducing refactor of tags and badges only the former ones get dedicated section on landing page and . As badges
are considered to be `curated` metadata, quick access to resources connected to particular badge is desired but currently
missing. The only way to browse by badges is when users already `know` certain badge exists and uses advanced search
 to search by badges. There is no place to actually find a list of badges used in Amundsen deployment nor to quickly filter
 by one.

From API perspective, metadata proxy already provides `badges` endpoint but it's not currently utilized.

## Guide-level Explanation (aka Product Details)

- Users can quickly access resources marked by badges by clicking on badge name on landing page
- Users can browse badges in Amundsen using `Browse` view 

## UI/UX-level Explanation

- Landing page contains `Available/Popular badges` section above `Popular tags`
- Browse page contains `Badges` section below `Tags` section

## Reference-level Explanation (aka Technical Details)

- Introducing new component for showing list of badges on Landing page
- Extending `Browse` view with `Browse badges` section

## Drawbacks
I have not identified drawbacks to introducing this. We should grow Amundsen to be better in managing Business Glossaries
and badges seem like a good fit.

## Alternatives

- Leave current state as-is with users not being able to see all badges defined in Amundsen quickly.
- Alternative UI approach would be to rename `Popular tags` section to `Popular labels` with `tags` and `badges` subsections

## Prior art

- This topic has been brought up by users several times on our Slack

## Unresolved questions

- Do we show `Available badges` or `Popular Badges` on landing page (number of unique badges might be significantly
smaller than tags so maybe there is no point in showing just `Popular` ones + there is no dimension to )?
- Would it make more sense to have `Popular badges` on landing page above or below tags?
- Shall we hide `Popular/All badges` section on landing page when `/badges` endpoint returns empty list? 

## Future possibilities

- show badges are two-dimensional data rendered as 'category: badge_name' rather than just 'badge_name'\
- 'Browse badges' component shows badges grouped in separate categories (rather than all badges in one container)