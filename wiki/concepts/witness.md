# witness

A [[witness]] is a temporary durability component used by [[CURP]]. It records client requests without assigning an order, stores them durably until the master garbage-collects them, and provides records during recovery.

Witnesses are not backups in the CURP primary-backup design. Backups preserve ordered state or logs; witnesses preserve unordered requests whose replay is safe only because each witness accepts mutually commutative records.

## KCensus uses a different witness role

In [[KCensus]], a witness is a voting process that records acceptances, and a proposer may know what that witness recorded. It is not a CURP-style dedicated unordered-durability service. Required witnesses are also acceptors under the requirement normal form; non-voting clients cannot count as witnesses ([[KCensus-2026]], §§3.1, 4.5; [[knowledge-requirement]]).

## Related pages
[[CURP]], [[CURP-2019]], [[fast-path]], [[recovery]], [[quorum]], [[conflict]]
