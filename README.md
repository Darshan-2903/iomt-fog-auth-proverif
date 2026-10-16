# ProVerif model: lightweight MFA for fog-assisted IoMT

Formal model accompanying the paper *"Sustainable Lightweight Multifactor Authentication
for Secure Medical Data Exchange"*.
It checks the Login and Authentication phase (Table III of the paper) between a
Patient device and a Fog Node under the Dolev-Yao attacker.

## Repository layout

```
model/lightweight_mfa.pv        the ProVerif model (single file)
results/proverif_output.txt     full ProVerif output of the run reported in the paper
CITATION.cff                    how to cite this repository
LICENSE                         MIT
```

## How to run

1. Install ProVerif (https://bblanchet.github.io/proverif/ or `opam install proverif`).
   The reported run used ProVerif **<version, e.g. 2.05>** on **<OS>**.
2. From the repository root:

```bash
proverif model/lightweight_mfa.pv
```

It finishes in a few seconds and should print the five `RESULT` lines below.

## Queries and expected results

| # | Query | Meaning | Result |
|---|-------|---------|--------|
| Q1 | `not attacker(secU)` | data the Patient sends under the session key SK stays secret | true |
| Q2 | `not attacker(secF)` | data the Fog Node sends under SK stays secret | true |
| Q3 | `not attacker(FNsk)` | Fog Node master key stays secret, even though the attacker is given the stolen Fog Node database (`TID`, `R_i`) and the device contents (`C_i`, `TID`) | true |
| Q4 | `event(FNAcceptsUser(t,T,nu)) ==> event(UserStartsLogin(t,T,nu))` | Fog Node authenticates the Patient (non-injective agreement on `TID`, `T`, `nu`) | true |
| Q5 | `inj-event(UserAcceptsFN(...)) ==> inj-event(FNSendsResponse(...))` | Patient authenticates the Fog Node, with injective agreement on all values including SK (mutual authentication and key agreement) | true |

The complete output is in `results/proverif_output.txt`.

## Modelling choices and limits

These simplifications are also stated in the header of the `.pv` file.

- **XOR masks** (`N1 = n_u xor h(S||T)` etc.) are modelled as keyed masks
  (`mask` / `unmask`). Each mask key is a hash-derived one-time value, so this is
  equivalent for secrecy.
- **Fuzzy extractor** is abstracted: `Rep` always returns the same private `sigma`.
  The attacker never learns `sigma`, so compromise of both the device and the
  biometric is **not** modelled.
- **Pseudonym update** (`TID_new`) is not modelled: `TID` is fixed in the model.
  The update value is a hash of values already in the session-key input, so it adds
  no attacker knowledge, but **unlinkability and untraceability are not verified
  by this model**.
- **Replay protection**: the `DeltaT` window is abstracted (the attacker may supply
  any timestamp) and the replay cache is a table. ProVerif cannot prove that a table
  entry is absent, so the *injective* form of Q4 is not provable; Q4 is therefore
  stated in its non-injective form. Replay resistance (`V1` cache + `DeltaT`) is argued
  in the paper's informal analysis.
- **Registration phase** is not modelled; its outputs (`C_i`, `TID`, `R_i`) are given
  as initial state.

## Citation

See `CITATION.cff` (GitHub shows a "Cite this repository" button) or:

```bibtex
@misc{proverif_model,
  author    = {<Your Name> and <Co-authors>},
  title     = {{ProVerif} model of a lightweight multi-factor authentication protocol for fog-assisted {IoMT}},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.<XXXXXXX>},
  url       = {https://github.com/<user>/iomt-fog-auth-proverif}
}
```

## License

MIT, see `LICENSE`.
