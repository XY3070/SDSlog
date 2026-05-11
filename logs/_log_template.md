# Detailed Log Template

每条关键测试 / 生产结果都按下面的模板写。路径统一写 absolute path，大文件只记路径不复制。

```md
# <Title>

Date: YYYY-MM-DD
Topic: <cohort_qc | demography | sds | ihs_ehh | meta_overlap | relate_clues2>
Type: <test | production | summary>
Question: <This record answers what?>

## Input paths

- `<absolute path>`

## Output paths

- `<absolute path>`

## Parameters

- `key=value`

## Demographic model

- `Demographic model=<registry name or Model ID>`
- `Gamma shape source=<absolute path or inherited model ID>`
- `Present-day Ne(0)=<numeric | implicit / unknown from historical file provenance>`
- `Model role=<mainline | sensitivity | rejected | audit-only>`
- `Model note=<one-line reason or caveat>`

## Scripts / commands / job IDs

- `<script path>`
- `<log path or job id>`

## Key plots

- `<repo plot link>` or `None archived`

## Key result files

- `<Result ID>: <absolute path>`

## Conclusion

- One to three concise facts.

## Next step

- The next concrete action.
```
