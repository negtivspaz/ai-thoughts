
Update Hermes

Try to launch `hermes --tui`, receive the error:

```sh
hermes --tui
TUI build failed.
> hermes-tui@0.0.1 build
> npm run build --prefix packages/hermes-ink && tsc -p tsconfig.build.json && chmod +x dist/entry.js
> @hermes/ink@0.0.1 build
> esbuild src/entry-exports.ts --bundle --platform=node --format=esm --packages=external --outfile=dist/ink-bundle.js
✘ [ERROR] Expected end of file in JSON but found ":"
    ../../../../../package.json:1:10:
      1 │ "openclaw": {
        ╵           ^
1 error
```

this is caused by node running in packages/hermes-ink, which locates



find where it is

```sh
path="$HOME/.hermes/hermes-agent/ui-tui/packages/hermes-ink"
while [ "$path" != "/" ]; do
  [ -f "$path/package.json" ] && echo "$path/package.json"
  path=$(dirname "$path")
done
```

the output
```sh
/home/negtivspaz/.hermes/hermes-agent/ui-tui/packages/hermes-ink/package.json
/home/negtivspaz/.hermes/hermes-agent/ui-tui/package.json
/home/negtivspaz/.hermes/hermes-agent/package.json
/home/negtivspaz/package.json
```

top level from `~/.hermes/hermes-agent/ui-tui/packages/hermes-ink/package.json` is `~/`. the culprit is `~/package.json`