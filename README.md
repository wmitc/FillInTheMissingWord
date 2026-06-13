# Fill In The Gaps

A semantics framework for **predicting missing words in a sentence**. Given a sentence
with a blank — for example `Mary eats a _` — the system proposes reasonable words to fill
the gap (`potato`, `tomato`, `apple`, ...).

Developed for MIT **6.863J — Natural Language and the Computer Representation of Knowledge**
(Spring 2018) by William Mitchell, Scott Viteri, and Tugrul Savran.

## How it works

The pipeline turns each sentence into a structured meaning representation, learns what
varies across similar training examples, and then generalizes those learned values with
WordNet to produce grammatical candidate words:

```
sentence  →  parse tree  →  event structure  →  grouped events  →  generalized fillers
```

1. **Parse & build event structures.** Each sentence is parsed with a feature-based
   context-free grammar and reduced — via compositional lambda semantics — into an *event
   dict* capturing features like `action`, `agent`, and `patient`
   (e.g. `John eats the potato` → `{action: eat, agent: John, patient: potato}`).
2. **Group similar events.** Training events that differ in only one (or two) features are
   grouped, and the differing values are collected. This near-miss style of learning
   discovers what is interchangeable in a given slot.
3. **Generalize with WordNet.** The learned values are expanded through WordNet relations —
   synonyms, shared hypernyms, hyponym closures, and (for verbs) entailments — to suggest
   words beyond the training lexicon.
4. **Conjugate & filter.** Candidates are re-conjugated (via the bundled NodeBox English
   Linguistics library) so tense and number stay correct, keeping the filled sentence
   grammatical.

The full narrative — including experiments, design decisions, and dead ends — lives in the
notebook, with a written export in `FillInTheGaps.pdf`.

## Getting started

This project requires **Python 2.7** and a pinned set of legacy NLP dependencies. The
environment is fully described by `shell.nix`. With the [nix package manager](https://nixos.org/):

```
nix-shell
jupyter-notebook FillInTheGaps.ipynb
```

If you are unfamiliar with nix, simply make sure you have the dependencies listed in
`shell.nix` installed (Python 2.7 with `nltk`, `numpy`, `tkinter`, `matplotlib`, plus
`pandoc` and `texlive` for the document exports).

To install nix:

```
curl https://nixos.org/nix/install | sh
```

Even though this source is trust-worthy, remember that it is always good form to look at
the code before running the above command!

The first run downloads the WordNet and Treebank corpora via NLTK.

## Usage

`FillInTheGaps.ipynb` is the main entry point and drives the whole pipeline. It reads
training and test sentences from `Input/`:

- `Input/training.txt` — example sentences the system learns from (one per line)
- `Input/testing.txt` — sentences containing a gap to fill

The top-level driver `fillInTheGaps(training, gap_sentences, groupingProcedure,
generalizationProcedure)` runs the pipeline; swap the two procedure arguments to try
different strategies:

- **Grouping:** `keepSeparate`, `groupIfOneDiff`, `groupIfOneOrTwoDiffs`
- **Generalization:** `getSynonyms`, `getSharedHypernymsOrEntailments`,
  `getHyponymsClosureOrEntailments`

Sample outputs for several `grouping × generalization` combinations are checkpointed in
`Output/`, named after the strategies that produced them.

## Repository layout

| Path | Description |
| --- | --- |
| `FillInTheGaps.ipynb` | Main notebook — pipeline, experiments, and write-up |
| `rules.py` | Grammar + lexicon (`addLexicon`), as syntactic-production / semantic-lambda pairs |
| `semantic.py` | Convenience layer; `sentenceToEventDict` runs parse → decorate → evaluate |
| `semantic_rule_set.py` | `SemanticRuleSet` — lexicon, rule map, parser, learned database |
| `production_matcher.py` | Matches CFG productions to the feature chart; decorates parse trees |
| `lambda_interpreter.py` | Reduces a decorated tree's lambdas into a final event dict |
| `category.py`, `cfg.py`, `featurelite.py` | Vendored/modified NLTK: feature categories, CFG, unification |
| `semantic_db.py` | SQLite-backed storage and pretty-printing of events |
| `drawtree.py`, `utils.py` | Tkinter tree viewer and tree helpers |
| `en/` | Bundled NodeBox English Linguistics library (conjugation, pluralization) |
| `nltk_098/` | Pinned legacy NLTK |
| `Input/`, `Output/` | Sample inputs and checkpointed experiment results |
| `docs/` | Project proposal, mid-course check-in, and final paper |

## Limitations & future work

- Currently fills only **noun** or **verb** gaps.
- Possible extensions: using synsets during the iterative grouping step, grouping that
  follows the `n/log(n)` human-learning rule, and supporting more parts of speech.

## Authors

William Mitchell, Scott Viteri, and Tugrul Savran — MIT 6.863J, Spring 2018.