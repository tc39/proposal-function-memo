# Brief history of ES Function.prototype.memo
For information on what “Stage 1” and “Stage 2” mean,
read about the [TC39 Process][].

More information about contributing is also available in [CONTRIBUTING.md][].

## 2015–2021
The pipe champion group presents F# pipes (a tacit-unary-function-application
operator) for Stage 2 twice to TC39, being unsuccessful both times due to
pushback from multiple other TC39 representatives’ memory performance concerns,
syntax concerns about await, and concerns about encouraging ecosystem
bifurcation/forking.

For more information, see the [pipe proposal’s HISTORY.md][pipe history].

## 2021-09
Inspired by [pipe issue #233][], [@js-choi][] creates a new proposal
that would add several Function helper methods.

## 2021-10
[On 2021-10, proposal-function-helpers is presented to the Committee
plenary][2021-10] for Stage 1. The Committee rejects the proposal due to its
being overly broad and requests that it be split up into multiple proposals.

## 2022-03
[On 2022-03, a TC39 incubator meeting is held][2022-03 incubator] to discuss
Function helpers in general. Several representatives express support for
Function.prototype.once, and it is decided to pursue Function.prototype.once
for Stage 1.

[TC39 process]: https://tc39.es/process-document/
[CONTRIBUTING.md]: https://github.com/tc39/proposal-pipeline-operator/blob/main/CONTRIBUTING.md

[pipe history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
[pipe issue #233]: https://github.com/tc39/proposal-pipeline-operator/issues/233

[@js-choi]: https://github.com/js-choi

[2021-10]: https://github.com/tc39-transfer/proposal-function-helpers/issues/17#issuecomment-953814353

[2022-03 incubator]: https://github.com/tc39/incubator-agendas/blob/main/notes/2022/03-08.md
