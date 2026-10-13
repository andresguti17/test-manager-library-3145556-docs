# ADR-005 · The pending CHECKs stay in the domain

- **Status:** rejected (proposed, will not be applied)
- **Context:** data-model §4 marks `ck_book_total_positive` (`total_copies >= 1`) and `ck_user_role`
  (`role` in `admin,librarian`) as **pending (T-18) · today domain-only**. T-18 would lower both
  invariants into the engine.
- **Decision:** they are **not** lowered. They stay enforced by the domain only
  (`Quantity` value object, `User` role check), exactly as data-model §4 already states —
  so no base information changes.
- **Discarded alternatives:** adding both CHECKs in a second migration (rejected: freeze risk
  on the verified schema hours before delivery for invariants the domain already guarantees
  and that no evidence requires at the engine level).
- **Consequences:** a hand-written `INSERT` violating either rule succeeds at the engine and
  must be caught in review; the last engine barrier remains `ck_book_available_range`.
  If a real out-of-system write ever violates them, this ADR is revisited.
- **Verification:** the §10 queries return no `ck_book_total_positive` and no `ck_user_role`;
  data-model §4 keeps describing both rows as domain-only.
