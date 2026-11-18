# C++ DSA Notes

Short revision notes for SDE 2 DSA interviews in C++. This folder is published on GitHub as its own repo, `cpp-notes`.

## Formatting

[`Notes-Formatting.md`](../Notes-Formatting.md) is the main source for how notes are written, structured and formatted, and for how to add content. Follow it for every change in this folder. The next line loads it into every Claude Code session started here:

@../Notes-Formatting.md

## Files

| File               | Covers                                                                                                                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|
| `Notes_strings.md` | ASCII and case bits, char arithmetic, `stoi`, `to_string`, `substr`, `find`, `erase`, case helpers, number bases          |
| `Notes_bits.md`    | bit operators, precedence, single-bit operations, bit tricks, XOR, bitmask subsets, traps, binary                         |
| `Notes_vectors.md` | vectors and iterators: `rbegin` / `rend`, `swap` with references vs copies, iterator invalidation                         |
| `Notes_heap.md`    | `priority_queue`: what a heap is, complexity (why building is O(n)), syntax, top K, heap vs `set`, traps                  |
| `Notes_maps.md`    | `unordered_map` / `map`: key checks, cost of each operation, key types, insert / erase, iterators, `map`-only sorted keys |
| `Notes_math.md`    | modulo: negative remainders, `%` formulas for DSA                                                                         |
| `CplusNotes.md`    | longer notes in 25 sections: C++ basics, binary search, trees, DP, slow/fast pointers, sorting, heap extras, BFS, graphs  |

## Folder rules

- **Traps:** `Notes_bits.md` and `Notes_heap.md` keep a Traps section as the only place for warnings. The other files put traps inline.
- **`README.md`:** the landing page for GitHub readers. Keep its file list and its topic-to-section table for `CplusNotes.md` in step with this folder.
- **Links into this folder:** `../DSA/Array-Strings/My_DSA_Mistakes.md` links to sections here by number, and `../DSA/README.md` maps topics to the section numbers of `CplusNotes.md`. Keep those numbers and section names stable, or update the links.
- **Checklist:** the problem checklist lives in the DSA repo, at `../DSA/Array-Strings/Checklist_String_Array.md`.
- **Scratch files:** `code.cpp` and `out.file` are the user's scratch files. Do not edit them.
