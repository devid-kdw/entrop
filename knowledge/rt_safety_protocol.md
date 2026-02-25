# RT Safety Protocol — ENTROP Backend

These rules are absolute. Any code that violates them is rejected and rewritten.

## Prohibited in process(), on_thread_pool_exec(), or any function they call

| Action | Permitted Alternative |
|---|---|
| malloc(), new, delete | Pre-allocate at plugin init |
| std::thread construction/destruction | clap_host_thread_pool exclusively |
| std::mutex::lock() | Lock-free FIFO or std::atomic |
| File I/O | Preload at init; memory-mapped resources |
| std::cout, printf, any logging | Post to lock-free logger on UI thread |
| std::this_thread::sleep_for() | Never needed in correct RT code |

## Thread ownership

Audio thread: ✅ atomic param reads · ✅ audio output writes · ✅ post to GUI FIFO
             ❌ write GUI state directly · ❌ CLAP host extensions except thread_pool/thread_check

GUI thread:  ✅ read from GUI FIFO · ✅ write params to lock-free FIFO
             ❌ access voice state directly · ❌ call process()

## Parameter changes
All changes arrive as CLAP events in the input queue.
Interpolate all changes over minimum 64 samples.
MPE modulation = non-destructive offset, never overwrites base value.

## Required checklist in every backend handoff
- [ ] No heap allocation in audio path (AddressSanitizer verified)
- [ ] No std::thread in audio path
- [ ] No mutex in audio path
- [ ] All param changes smoothed ≥64 samples
- [ ] Thread pool via clap_host_thread_pool only
- [ ] MPE offset does not overwrite base value
- [ ] clap_plugin_thread_check used in debug build
