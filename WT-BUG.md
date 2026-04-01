# Server thread crash on certain capsule requests

## Cause

`ConnectStream::run()` calls `Capsule::with_frame(&frame)` unconditionally on every frame
read from the session stream (`connect.rs:41`). `Capsule::with_frame` has a hard `assert!`
that the frame must be `FrameKind::Data` (`capsule/mod.rs:38`).

However, `StreamSession::validate_frame` (`stream.rs:1041-1048`) passes `FrameKind::Headers`
and `FrameKind::Exercise` frames through as `Ok`. When any such frame arrives on the session
stream, it clears validation but hits the assert in `Capsule::with_frame`, panicking the
worker task.

The second panic (`driver/mod.rs:233`, `"Driver worker panic!"`) is a consequence: the worker
task panics without setting `SharedResultSet`, so `SharedResultGet::result().await` returns
`None`.

Relevant locations:

- `wtransport-proto/src/capsule/mod.rs:38` — `assert!(matches!(frame.kind(), FrameKind::Data))`
- `wtransport/src/driver/streams/connect.rs:41` — `Capsule::with_frame(&frame)` called without frame kind check
- `wtransport-proto/src/stream.rs:1043-1047` — `StreamSession::validate_frame` allows `Headers` and `Exercise` frames through
- `wtransport/src/driver/mod.rs:233` — secondary panic when worker dies without setting result

## Fix

Guard the `Capsule::with_frame` call in `connect.rs:41` with a frame kind check, skipping
any non-`Data` frame:

```rust
// connect.rs, inside the loop at line 40
Ok(frame) => {
    if !matches!(frame.kind(), FrameKind::Data) {
        continue;
    }
    let capsule = match Capsule::with_frame(&frame) {
        // ...
    };
```

This matches the existing "unknown capsule, skip it" intent at `connect.rs:51-59`, but
applied before the assert is reached.
