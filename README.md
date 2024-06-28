# locutus

Kinda like Borg, but kinda not

## Test Results

~230k files

Various Python walk implementations (`os`, `pathlib`), single-threaded - about 1.5s to iterate all files and print the absolute path
Using `hashlib.file_digest` - single threaded, 31s to iterate all files and print the path + hexdigest.
Implies that the "walk" part is good enough, but the hash should be made parallel, and done less (e.g. ignore files/folders)

My naive Rust implementation took 35s, which clearly doesn't make any sense. The optimized Python version should be lower-bound at around Rust's speed. 
The two implementations were obviously not the same, as the Python variant basically whatever called directly into the C-lib, while Rust's had some extra code for filtering and memory-bounding the hash.

Removing the buffered read from Rust for memory control, the Rust version (including printing) is slightly slower than Python (~1-2s). The Rust version could be optimized further, but it's probably safe to say that in the trivial case - these two will operate at similar-ish speeds when single-threaded without other operations. At the moment, hashing is only required for a subset of files, so this won't be a substantial part of the overall time taken.

Interesting note: The sha2 library for Rust took 2x the time - so it's a no-go... Unless I enable asm flags, in which case it's on par with ring.

Also, using Rayon, we take the total time down to 10-11s.
