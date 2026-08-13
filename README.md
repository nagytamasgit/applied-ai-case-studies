# Applied AI Case Studies

A curated collection of real-world **applied AI** case studies — each one a short,
honest writeup of a problem, the approach taken, what worked, what didn't, and the
measurable outcome.

The goal is *applied*: every case study focuses on shipping something useful, not on
benchmarks for their own sake.

## Index

| # | Case study | Domain | Outcome |
|---|------------|--------|---------|
| [0001](case-studies/0001-oldal-ai-website-builder/) | Oldal AI — AI Website Builder | Generative UI / design-to-code | Shipped to production |
| [0002](case-studies/0002-cook-solutions-audio-detection/) | cook-solutions / "sear" — Cooking Detection from Audio | Audio signal processing | Prototype (pending kitchen data) |

_Add a row here each time you add a case study._

## Structure

```
case-studies/
  NNNN-short-slug/
    README.md      # the writeup
    assets/        # images, charts, diagrams
_template/
  README.md        # copy this to start a new case study
```

## Adding a new case study

1. Copy the template:
   ```bash
   cp -r _template case-studies/0002-your-slug
   ```
2. Fill in `case-studies/0002-your-slug/README.md`.
3. Add a row to the **Index** table above.
4. Open a PR (see [CONTRIBUTING.md](CONTRIBUTING.md)).

## License

Content and code in this repository are released under the [MIT License](LICENSE).
