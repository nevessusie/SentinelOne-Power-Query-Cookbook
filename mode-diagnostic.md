# Diagnosing "Don't understand [|]" errors

## Symptom

You paste a query like this into the search bar and get a parse error:

```
event.type = "Process Creation" and endpoint.name contains "ABC123XYZ"
| group src.process.name, src.process.user
| sort -hits
```

Error: `Don't understand [|]`

## Root cause checklist

1. **Wrong query surface (most common cause).** Basic Event Search does
   not support pipe (`|`) operators at all. If you're in that mode,
   *any* pipe will error, regardless of what comes after it.
2. Missing spaces around the pipe character — less common, but worth
   ruling out once you've confirmed you're in the right mode.

## Fast diagnostic procedure

1. Strip everything from the first `|` onward:

   ```
   event.type = "Process Creation" and endpoint.name contains "ABC123XYZ"
   ```

2. Run that alone. If it works in Basic Event Search, your filter logic
   is fine — the problem is purely query mode.
3. Switch to PowerQuery mode in the console.
4. Re-add the pipeline one stage at a time:

   ```
   event.type = "Process Creation" and endpoint.name contains "ABC123XYZ"
   | group src.process.name, src.process.user
   ```

   then add the sort:

   ```
   event.type = "Process Creation" and endpoint.name contains "ABC123XYZ"
   | group src.process.name, src.process.user
   | sort -hits
   ```

## Notes

- `| sort -hits` — the dash prefix means descending order (highest hit
  count first). Omit the dash for ascending.
- Filtering on the unique alphanumeric tail of a hostname (rather than
  the full display name) avoids encoding issues when the full endpoint
  name contains spaces or accented characters.
