# tools

## run tests

`forge test`

verbose:

`forge test -vvvv`

watch mode:

`forge test --watch`

## test coverage

`forge coverage`

**get test coverage coloring in gutters**:

- install _Coverage Gutters_ vscode extension
- `forge coverage --report lcov`
- open command palette and select "display coverage report"

## slither

`slither . --exclude-dependencies`

create `slither.config.json`:

```json
{
  "filter_paths": "lib|script|test"
}
```
