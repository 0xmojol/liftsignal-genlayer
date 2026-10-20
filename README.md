# LiftSignal

LiftSignal is a standalone GenLayer Intelligent Contract that records consensus-checked interpretations of the VGF public lift-status page.

Each deployment tracks one station and one lift equipment number. Anyone can call the contract to check the page. GenLayer validators independently read the public page and assess whether it reports an outage for that lift.

## Statuses

- `OUTAGE_REPORTED` — the page contains an outage, fault, repair, or interruption entry matching the target lift.
- `NO_OUTAGE_LISTED` — the retrieved page contains no entry for the target equipment number. This does not prove that the lift is working.
- `UNCLEAR` — the page or evidence does not support a safe conclusion.

Each assessment stores the station, equipment number, source URL, status, record number, and an exact evidence quote when an outage is reported.

## How GenLayer consensus is used

The leader reads the VGF page and proposes a structured status. Validators independently read the page and assess the same station and equipment number.

For an outage result, validators also check that the leader's evidence quote appears in their own copy of the page and includes both the station name and equipment number. If the evidence is missing, invalid, too long, or unclear, the contract returns `UNCLEAR`. A validator disagreement rejects the proposed result.

The contract appends the assessment only after the consensus call returns.

## Contract methods

Constructor: `LiftSignal(station_name, equipment_number)`

Write method:
- `check_status()` — checks the public page through GenLayer consensus and stores an assessment.

Read methods:
- `get_assessment_count()` — returns the number of stored assessments.
- `get_assessment(record_number)` — returns one assessment by its zero-based number.
- `get_latest_assessment()` — returns the latest assessment, or an empty string if there is none.

## Source

The contract reads this VGF public page:

- [VGF lift status](https://www.vgf-ffm.de/de/fahrgastinfo/barrierefreies-reisen/status-aufzuege/)

RMV also directs passengers to VGF lift-status information and Deutsche Bahn station-lift information:

- [RMV station accessibility information](https://www.rmv.de/c/de/service/infos/infos-fuer-mobilitaetseingeschraenkte/informationen-zu-stationen-und-haltestellen)

This version reads the VGF page only; it does not read Deutsche Bahn's separate source.

## Try it in GenLayer Studio

1. Open `contracts/liftsignal.py` in GenLayer Studio.
2. Select **Normal (Full Consensus)**.
3. Deploy with a station name and equipment number.
4. Call `check_status()` with no arguments.
5. Read the result with `get_latest_assessment()`.

The live deployment below was created for `Hauptbahnhof`, equipment number `59`.

## Live Studio evidence

- Contract: [0xeFED9944D0D74dDFE30F187990ad3C2B6f24CCD6](https://explorer-studio.genlayer.com/address/0xeFED9944D0D74dDFE30F187990ad3C2B6f24CCD6)
- Deployment transaction: [0x527468d4ad7ef661c3c220975bf99889c4368e685f40d22ead7d2618607622c8](https://explorer-studio.genlayer.com/tx/0x527468d4ad7ef661c3c220975bf99889c4368e685f40d22ead7d2618607622c8)
- Assessment transaction: [0x4cdd36d84200cf91041ff2890b58f9133d11fddb63ac291714ddd8cba380ddf1](https://explorer-studio.genlayer.com/tx/0x4cdd36d84200cf91041ff2890b58f9133d11fddb63ac291714ddd8cba380ddf1)
- Assessment result: `NO_OUTAGE_LISTED`
- The assessment was finalized in Normal mode with an accepted consensus result and five initial validators.

This is one live smoke test. It does not prove that every page variation or status has been tested.

## Limitations

- LiftSignal records an interpretation of the retrieved VGF page. It is not an official transit service, accessibility certification, or guarantee that a lift is operational.
- `NO_OUTAGE_LISTED` does not mean that the lift is working.
- The page may change or be unavailable. `UNCLEAR` is used when the evidence cannot be interpreted reliably.
- The contract stores the source URL and, when applicable, an exact quoted passage. It does not store the full page or a page digest.
- This version is limited to the VGF page and one lift per contract deployment.
- Automated direct-mode tests have not yet been added to this repository. The Studio transaction above is a smoke test, not a replacement for a test suite.

## Repository

- Contract source: [`contracts/liftsignal.py`](contracts/liftsignal.py)
- License: MIT
