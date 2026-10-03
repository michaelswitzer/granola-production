# granola-production

These are the tools used to build, test and ship Granola controllers.

| Script | What it does |
| --- | --- |
| `scripts/granola` | Flashes a blank board and checks every input on a finished controller (Beacon, Plateau). |
| `scripts/granola-orders` | Reads open Shopify orders and lists what to print, by color, what to assemble, by controller, and what is left to ship. |
| `scripts/granola-factory` | Opens the Shopify orders page, the slicer on `Granola_Printer.3mf` and an order summary, tiled. |

Each script prints its usage with `--help`.

`firmware/` holds the `.uf2` images that `granola flash` writes. `Granola_Printer.3mf` is the slicer project for the cases and buttons.

## Requirements

- Linux. `granola` reads evdev and sysfs directly.
- [Nix](https://nixos.org/download). Every script is a `nix-shell` shebang that fetches its own dependencies, so there is nothing to install. The Python scripts use only the standard library.
- `granola-factory` assumes the desktop it was written for: dwl, foot, OrcaSlicer and LibreWolf. The other two scripts don't depend on any of it.

## The env file

`granola-orders` and `granola-factory` read their store settings and credentials from a file outside the repo:

```
~/.config/granola/shopify.env
```

It uses plain `KEY=value` lines. Blank lines and `#` comments are allowed, and quotes around values are stripped.

```sh
# The store's handle: the <handle> in admin.shopify.com/store/<handle>,
# or the first part of <handle>.myshopify.com. Either form works.
SHOPIFY_STORE=my-store

# Option A: an app made in the Shopify Dev Dashboard and installed on the
# store. The scripts trade these for a short-lived access token on every run.
SHOPIFY_CLIENT_ID=...
SHOPIFY_CLIENT_SECRET=...

# Option B: a legacy custom app's Admin API access token. If this is set,
# it is used instead of option A.
# SHOPIFY_TOKEN=shpat_...
```

Whichever kind of app you use, it needs only the **`read_orders`** Admin API scope. The colors customers pick (through Qikify Product Options) are stored on each order's line items, and that scope covers them.

To create the file:

```sh
mkdir -p ~/.config/granola
$EDITOR ~/.config/granola/shopify.env
chmod 600 ~/.config/granola/shopify.env
```

Then check it with a read-only call:

```sh
scripts/granola-orders summary
```

If something is missing, the script names the key it couldn't find. If Shopify rejects the credentials, it prints Shopify's error, for example `Oauth error app_not_installed`.

`granola-orders` also keeps track of which colors you have checked off as printed, in `~/.local/state/granola/printed.json` (`$XDG_STATE_HOME` if that is set). The file is created on first use.
