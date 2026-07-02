# witness

A [[witness]] is a temporary durability component used by [[CURP]]. It records client requests without assigning an order, stores them durably until the master garbage-collects them, and provides records during recovery.

Witnesses are not backups in the CURP primary-backup design. Backups preserve ordered state or logs; witnesses preserve unordered requests whose replay is safe only because each witness accepts mutually commutative records.

## Related pages
[[CURP]], [[CURP-2019]], [[fast-path]], [[recovery]], [[quorum]], [[conflict]]
