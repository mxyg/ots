# Contributing

Anyone may use, implement, fork and maintain this specification. That is the point of it.
There is no membership, no fee, and no approval process.

## Developer Certificate of Origin

Contributions must carry a `Signed-off-by` line, certifying the
[Developer Certificate of Origin 1.1](https://developercertificate.org/):

```
git commit -s -m "your message"
```

By signing off you certify that you wrote the contribution or otherwise have the right to submit
it under the licenses below, and that you understand it will be public and redistributed.

**Why this is required.** Anyone is free to maintain and extend this specification, which means
contributions arrive from people we do not know. The sign-off is what records, at the time of the
contribution, that the contributor had the right to make it and licensed it accordingly. Without
it there is no record, and a contributor who later changes their mind has nothing to disprove.
It costs one command-line flag.

## Licensing of contributions

By contributing you agree your contribution is licensed under the same terms as the part of the
repository it touches:

| Part | License |
|---|---|
| Specification text (`spec/`, `README*`) | CC BY 4.0, and additionally Apache License 2.0 |
| Schemas (`schema/`) | Apache License 2.0 |

The specification text is offered under **both** CC BY 4.0 and Apache-2.0; you may rely on either.
Apache-2.0 is included because it carries an express patent grant and patent-retaliation clause,
which CC BY does not. If you are implementing this specification and patents are a concern for
you, take it under Apache-2.0.

## Forking and independent maintenance

You may fork this specification and maintain it yourself, including commercially, including in
competition with us. The only obligation is attribution, as CC BY 4.0 and Apache-2.0 require.

If your fork diverges, please rename it. Two incompatible things with one name is the one outcome
that helps nobody.

## What makes a good contribution

The most useful contribution is not a wording fix. It is:

> "I implemented section N in my product and here is what did not survive contact with reality."

This specification is written alongside a working implementation on purpose. A rule that has
never been implemented is a guess, and we would rather delete a guess than keep it.
