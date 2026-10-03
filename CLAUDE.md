## Checking a change

The tools are nix-shell scripts in `scripts/`. Every changed Python script
must at least parse:

```
nix-shell -p python3 --pure --run 'python3 -c "import ast; ast.parse(open(\"scripts/<name>\").read())"'
```

and a changed bash script must pass `bash -n`. Then run what the change
touches, as far as it goes without hardware or side effects: `--help` always,
`granola models` and `granola-orders summary` (read-only against Shopify)
where changed. `granola-factory` opens windows; don't run it unasked.
Flashing, nuking and anything needing a controller on the bus is left to
Mike.
