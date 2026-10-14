# Evidence policy

Only short, sanitised diagnostic excerpts are retained here. They are not raw logs and are not byte-for-byte source artifacts.

Timestamps, stream identifiers, addresses, ports, user names, host names, local paths, portal object paths, and unrelated diagnostics have been removed or generalised. No credentials, certificates, tokens, configuration files, frame data, or client content are included.

Each excerpt identifies whether a statement is a diagnostic record or an operator observation. Raw evidence remains outside this publication repository.

The effective flag lists in the excerpts are derived from the respective source-code connect expressions; they are not runtime log messages. Both binary artifacts retain the same hardcoded runtime message naming `DRIVER`. That message is accurate for A but stale for B, whose actual connect expression omits DRIVER.

The private Variant A raw log is identified by SHA-256 `b214b15bd14ca618aa50c102327da9f4091aebbec55af60c70a14207e18341ac`. Its sensitive contents and location are not published.
