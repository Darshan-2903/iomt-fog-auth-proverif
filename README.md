# ProVerif model: Sustainable Lightweight Multifactor Authentication for Secure Medical Data Exchange

Companion repository for the paper *"Sustainable Lightweight Multifactor
Authentication for Secure Medical Data Exchange"* by Darshan Mahajan, Sujata Mohanty
and Kare Jai Sai Chaitanya (NIT Rourkela).

It contains the ProVerif model used in Section V-B of the paper. The model covers the
login and authentication phase (Table III of the paper) between a user device `U_i` and
the Fog Node `FN`, under the Dolev-Yao attacker, with unbounded sessions.

## Repository layout

```
model/lightweight_mfa.pv        the ProVerif model (single file)
results/proverif_output.txt     full, unedited ProVerif output of the run reported in the paper
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

It finishes in a few seconds and prints the verification summary shown in
`results/proverif_output.txt`.

## Queries and results (Table IV of the paper)

| Paper | Query in the model | Property | Result |
|-------|--------------------|----------|--------|
| Q1 | `not attacker(secU)` | A private value the user sends encrypted under `SK` stays secret | true |
| Q2 | `not attacker(secF)` | A private value `FN` sends encrypted under `SK` stays secret | true |
| Q3 | `not attacker(FNsk)` | `FN_sk` stays secret even when the attacker is given the stolen `FN` database (`TID_i`, `R_i`) and the device contents (`C_i`, `TID_i`) | true |
| Q4 | `event(FNAcceptsUser(t,T,nu)) ==> event(UserStartsLogin(t,T,nu))` | `FN` accepts `(TID_i, T, n_u)` only if the user started that login (non-injective agreement) | true |
| Q5 | `inj-event(UserAcceptsFN(t,T,nu,nf,Ts,SK)) ==> inj-event(FNSendsResponse(t,T,nu,nf,Ts,SK))` | The user accepts `(TID_i, T, n_u, n_f, T_s, SK)` only if `FN` sent it (injective agreement; together with Q4 this gives mutual authentication and key agreement) | true |

Secrecy of `SK` (Q1, Q2) is tested by having each party send a private value encrypted
under `SK`, as described in the paper.

## Modelling assumptions

These match the "Modeling assumptions" paragraph of Section V-B and the header of the
`.pv` file.

- **XOR masks** (`N1 = n_u xor h(S||T)`, `N2 = n_f xor h(S||n_u||T_s)`) are modelled as keyed
  masks (`mask` / `unmask`). Each mask key is a hash-derived one-time value, so this is
  equivalent for secrecy.
- **Fuzzy extractor** is abstracted: `Rep` returns the same private `sigma_i` for the genuine
  user. Biometric noise and helper-data leakage are not modelled, and the attacker never
  learns `sigma_i`.
- **Pseudonym update** (`TID_new`) is omitted and `TID_i` is fixed in the model. The update
  value is a hash of inputs already used to derive `SK`, so it adds no attacker knowledge.
- **Replay protection**: the window `DeltaT` is abstracted (the attacker may supply any
  timestamp) and the `V1` replay cache is a table. ProVerif cannot prove that a table entry
  is absent, so the injective form of Q4 cannot be proved. Q4 is therefore stated as
  non-injective agreement, and replay rejection is argued in the informal analysis
  (Section V-C).
- **Registration phase** is not modelled. Its outputs (`C_i`, `TID_i`, `R_i`) are given as
  initial state.

## Scope

The results are symbolic. Anonymity and unlinkability, offline guessing of `(ID_i, sigma_i)`
and replay rejection within `DeltaT` are covered by the informal analysis in the paper, not
by this model.

## Citation

Use the "Cite this repository" button on GitHub (from `CITATION.cff`), or:

```bibtex
@misc{mahajan2026proverif,
  author    = {Mahajan, Darshan and Mohanty, Sujata and Chaitanya, Kare Jai Sai},
  title     = {{ProVerif} model of a lightweight multifactor authentication protocol for fog-assisted {IoMT}},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.<XXXXXXX>},
  url       = {https://github.com/<user>/iomt-fog-auth-proverif}
}
```

## License

MIT, see `LICENSE`.
