# SSD prefetch experiment source snapshot (2026-10-31)

This branch publishes the complete Mooncake source combination used by the
2026-10-31 SSD KVPool / prefetch OFF-versus-ON experiment. It preserves the
experiment baseline and the original eight modified files byte for byte.
It is an archival reproduction branch, not a rebase onto current upstream main.

## Source identity

| Layer | Commit |
| --- | --- |
| Upstream main baseline | `5e3ddd7fc3db3b8f9e75c0283b43022ff1862221` |
| Original local integration of PR #4272 (snapshot head `9afd0aaf`) | `36ec29c4ddca8c575814eaf7173176edf4ccb85b` |
| Exact local tail-flush patch and regression tests | `2da8eec1f7d07c8edb0c31dcdcdf6876d02e5e59` |
| Exact local read-admission / read-only-query patch and tests | `dc07f94fc098b38cd866201b40b096bbc9872466` |

The main baseline already contains promotion-on-hit #2071 and the subsequent
retry improvements #2690. The original #4272 integration commit is retained
with its original object ID.

The tail-flush correctness fix also has a separate, still-open upstream PR:
[#4739](https://github.com/kvcache-ai/Mooncake/pull/4739).
[Issue #4740](https://github.com/kvcache-ai/Mooncake/issues/4740) discusses both
the correctness problem and the separate experimental admission policy.
The read-admission policy is outside PR #4739. Publishing this branch does not
mean these changes have been merged upstream.

## Checkout

```bash
git clone --branch repro/ssd-prefetch-full-20261031 https://github.com/MarinaMiao/Mooncake.git
cd Mooncake
git checkout dc07f94fc098b38cd866201b40b096bbc9872466
git submodule update --init --recursive
```

This checkout includes both local patches; applying them again is unnecessary.
The recorded pybind11 submodule commit is
`58c382a8e3d7081364d2f5c62e7f429f0412743b`.

## Experimental policy

`MOONCAKE_OFFLOAD_READ_ADMISSION_THRESHOLD=0` (the default) retains eager
Put-triggered SSD offload. A positive value admits eligible memory objects
after that many demand metadata lookups. Both experiment groups used `2`.
These are metadata lookups, not confirmed successful payload reads.

Duplicate keys within a batch count once. Existing disk replicas, pending
offload tasks, internal offload source queries, and idempotent Put healing
queries do not add demand counts. Counts are volatile. Invalid values and a
positive threshold combined with `offload_on_evict` are rejected.

Prefetch OFF/ON is a separate setting; this policy was enabled in both groups.

## Verification and limits

On 2026-11-02 the eight files were exported from the experiment checkout,
reviewed, split into the two commits above, and compared by SHA256. All eight
match the exported checkout exactly, and the combined patch matches the
original dirty-tree diff. `git diff --check` passed.

Historical results reported by the experiment guide: FileStorage 29 tests,
StorageBackend 98 tests, Master SSD 10 tests, client read-only/admission 5 tests,
and offload scenarios 25 tests. The preserved FileStorage log was also
inspected and reports 29 passing tests. These are historical results, not new
runs performed when publishing this branch.

No rebuild or native test rerun was completed for publication. The available
development pod lacked runtime libraries, including `libglog.so.1` and
`libgflags.so.2.2`. The pre-commit/clang-format toolchain was unavailable.
The original experiment checkout and running services were not modified.

The experiment wheel was
`mooncake_transfer_engine_npu-0.3.13+ssd.readadmit1-cp311-cp311-manylinux_2_38_aarch64.whl`,
with recorded SHA256
`7557c3cc9aee6b14c89a6bdf982cf39f0c1098762689ab586c2118470df57542`.
The version suffix was set in a separate packaging staging tree; it is not a
change to this repository's wheel metadata. A fresh build is not guaranteed to
produce the same wheel bytes. No wheel binaries, model weights, benchmark
inputs, private deployment configuration, or experiment logs are included.

## SHA256 of the eight archived files

```text
0160a26fe29cae35660f54df5034bc70e0a378117e42d17c4da584ec5e29e1c8  mooncake-store/include/master_service.h
059887966555fb75a97df32341f7f13bfbcb22d2f8b80b88db10ff3645c76265  mooncake-store/include/object_metadata.h
ce10bab1bc9c5e44b819ebef9d4569bd7d16fad1bfb63922573e59c022d5b765  mooncake-store/src/client_service.cpp
659550904bcc407cdcecf50d16277f5a4ba7e3856aea35703ae57e7fedb4228f  mooncake-store/src/master_service.cpp
ee6721219f49e00721d30b5a18591a53d0b9599a207dc46ff374857f84a2fd06  mooncake-store/src/storage_backend.cpp
9c8bedd6f4636e230d2a985778fceb1570128bfc96586aaae8212a8dd388af5b  mooncake-store/tests/client_readonly_query_test.cpp
e7cd86d97a298b2010cbb0f4940495a16871a941a40e90f01afdb03828358c38  mooncake-store/tests/file_storage_test.cpp
ab5df3e0dd483908e5298745d5aa96f8022972a061e7f81178aac6e2d7383edb  mooncake-store/tests/master_service_ssd_test.cpp
```
