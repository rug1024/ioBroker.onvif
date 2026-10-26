# Snapshot fix for the existing customized installation

This deployment branch is based on `21f25a071c0ab7080bc9dc8d711ebb10e9535e2a`
and preserves the formatting and existing discovery/logging modifications of
the supplied production main.js.

Original SHA-256:
`0ccd57d95124d85735d252346bca455da8e758c3c7a2a727f168aab96c15d981`

Patched main.js SHA-256:
`caad7736cdac3122c48fbd6afee3b5a71340313d2418e20db1955f00d0796481`

Changes relative to the supplied file:
- Apply the tested snapshot single-flight/500 ms cache and bounded transport
  waits from upstream-based branch `fix/snapshot-single-flight-cache`.
- Remove an accidental second split of the already parsed scopes array. This
  threw on valid discovery replies. Remove the adjacent duplicate return.
- Remove two unused initial assignments, as already done in upstream.

Validation: 15 focused tests pass, including actual local HTTP serving with
mock camera I/O and discovery parsing with the real xml2js parser. Lint, Node
syntax check and git diff whitespace check pass. The new discovery regression
fails on the supplied original file and passes after the fix. Live collision
was reproduced on both production cameras before this patch: the competing
request returned HTTP 500 immediately while the first returned HTTP 200.
Production operation after installation is still pending. No actual camera
or complete ioBroker startup test is claimed by the focused suite.

Install only main.js into the existing adapter after checking the original
checksum, saving a private backup, verifying the downloaded patched checksum
and running `node --check`. Stop only onvif.0 during replacement, then start it.
Keep client URLs, detector programs/timers and TV controls unchanged. If the
replacement or startup fails, restore the backup before restarting onvif.0.
A subsequent adapter update can overwrite this temporary file-level patch.

After startup repeat paired snapshot requests for each camera. Both requests
must return HTTP 200 and complete JPEGs. Then run TV display and both detectors
together for at least 30 minutes and observe health logs for several hours.

For rollback: stop onvif.0, copy the verified original main.js backup back into
node_modules/iobroker.onvif/main.js, and start onvif.0 again.

The upstream PR candidate remains on `fix/snapshot-single-flight-cache`.
Do not use this production compatibility branch as the upstream PR head.
