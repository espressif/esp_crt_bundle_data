# ESP x509 Certificate Bundle Data

[![Component Registry](https://components.espressif.com/components/espressif/esp_crt_bundle_data/badge.svg)](https://components.espressif.com/components/espressif/esp_crt_bundle_data)

Root CA certificate data for the ESP-IDF [X.509 certificate bundle](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/protocols/esp_crt_bundle.html)
(`esp_crt_bundle`). It contains **data only**: the C implementation, the bundle
generator and the Kconfig options stay in ESP-IDF. Shipping the data as a
managed component lets the trusted root list be refreshed independently of
ESP-IDF releases.

## Contents

| File | Purpose |
|---|---|
| `certs/cacrt_all.pem` | Full Mozilla NSS root store, as extracted by curl's `mk-ca-bundle` |
| `certs/cmn_crt_authorities.csv` | Filter list used by `CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEFAULT_CMN` |
| `certs/cacrt_deprecated.pem` | Roots removed from the Mozilla store since the previous major version, opt-in via `CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEPRECATED_LIST` |
| `certs/cacrt_local.pem` | Espressif-maintained additions needed for some cross-signed chains |

## Usage

Nothing to configure. When `CONFIG_MBEDTLS_CERTIFICATE_BUNDLE` is enabled, the
ESP-IDF `mbedtls` component depends on this component, the IDF Component
Manager fetches it into `managed_components/`, and the bundle embedded in the
application is generated from `certs/` instead of the snapshot in
`$IDF_PATH/components/mbedtls/esp_crt_bundle`. With the component manager
disabled, the in-tree snapshot is used.

`esp_crt_bundle_attach()` and all `CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_*` options
behave exactly as before. To pick up a newer root list after it is published,
run `idf.py update-dependencies`.

## Versioning

Versions follow `MAJOR.MINOR.PATCH` (the ESP Component Registry requires all
three parts):

- **Major**: roots removed from `cacrt_deprecated.pem` (deprecated list
  clean-up), or a change to the common certificate authorities in
  `cmn_crt_authorities.csv` (market share changes).
- **Minor**: roots added to the default list, or roots removed from the
  default list and moved to `cacrt_deprecated.pem`.
- **Patch**: no change to the certificate set (local list, metadata,
  documentation).

With `^1`, a root dropped by Mozilla stays available through
`CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEPRECATED_LIST` until the next major
release, and the common bundle keeps the same set of authorities. The
Mozilla snapshot date of each release is recorded in `CHANGELOG.md`.

## How this component works

`project_include.cmake` sets the `ESP_CRT_BUNDLE_DATA_DIR` build property to
its `certs/` directory. The ESP-IDF mbedtls component reads that property
when generating the bundle and falls back to its in-tree snapshot when the
property is absent.
