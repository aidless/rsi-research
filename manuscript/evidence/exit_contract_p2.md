# Exit-code contract for the P2 meta-predictor rerun

The P2 falsification script exits 1 **deliberately**.

```
return 1 if result.p2_falsified else 0
```

`p2_falsified=True` means the prediction was **falsified** by the rerun (the
canonical single-split test AUC / the 10-fold CV mean are reproduced but the
prediction is rejected as self-confirming). Exit 1 is therefore the **verdict**
the script is designed to return, not a crash. There is no traceback in
`run-logs/p2_mu_priv.log`.

Consequence for the integrity gate: a non-zero exit is accepted only as
verdict semantics and must carry this contract file.
