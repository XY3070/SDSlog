# EPO ancestral fasta migration manifest — for other-server polarization

Source: `/data/home/grp-wangyf/xuyuan/SDSworkspace/input/reference/markers/homo_sapiens_ancestor_GRCh38/`
Target: same path layout on other server (used by `polarize_norm_1kg_epo.py`, EPO_DIR).

23 files chr1-22+X (+ optional MT/Y), total ~2.9 GB (chr1-22+X).

| chr | file | bytes |
|---|---|---|
| 1 | homo_sapiens_ancestor_1.fa | 251445990 |
| 2 | homo_sapiens_ancestor_2.fa | 244615468 |
| 3 | homo_sapiens_ancestor_3.fa | 200278518 |
| 4 | homo_sapiens_ancestor_4.fa | 192116704 |
| 5 | homo_sapiens_ancestor_5.fa | 183353645 |
| 6 | homo_sapiens_ancestor_6.fa | 172514042 |
| 7 | homo_sapiens_ancestor_7.fa | 160939436 |
| 8 | homo_sapiens_ancestor_8.fa | 146590026 |
| 9 | homo_sapiens_ancestor_9.fa | 139778668 |
| 10 | homo_sapiens_ancestor_10.fa | 135135401 |
| 11 | homo_sapiens_ancestor_11.fa | 136437493 |
| 12 | homo_sapiens_ancestor_12.fa | 134608067 |
| 13 | homo_sapiens_ancestor_13.fa | 115507976 |
| 14 | homo_sapiens_ancestor_14.fa | 108114160 |
| 15 | homo_sapiens_ancestor_15.fa | 103011105 |
| 16 | homo_sapiens_ancestor_16.fa | 91241733 |
| 17 | homo_sapiens_ancestor_17.fa | 84090020 |
| 18 | homo_sapiens_ancestor_18.fa | 81177022 |
| 19 | homo_sapiens_ancestor_19.fa | 59203797 |
| 20 | homo_sapiens_ancestor_20.fa | 65088613 |
| 21 | homo_sapiens_ancestor_21.fa | 47177087 |
| 22 | homo_sapiens_ancestor_22.fa | 51326657 |
| X | homo_sapiens_ancestor_X.fa | 157601307 |
| (MT) | homo_sapiens_ancestor_MT.fa | 16739 |
| (Y) | homo_sapiens_ancestor_Y.fa | 57799693 |

Notes:
- `.fa` = one record per chr (already verified chr19: 1 line header + single sequence, no line breaks inside sequence beyond wrapping).
- `polarize_norm_1kg_epo.py` EPO_DIR expects `homo_sapiens_ancestor_{chr}.fa`, chr as int (1-22) or "X".
- Migrate the `.fa` only; `.fai`/`.bed`/README regenerable (faidx).
- Verify after copy: `md5sum` both sides; chr19 tested in Relate reconstruction (84% EPO-orientable, N/gap ~10%, true mismatch 5.5%).
