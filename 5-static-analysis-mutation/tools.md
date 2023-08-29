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

## echidna

```zsh
echidna template.sol --contract TestToken
echidna Template.sol --contract EchidnaTemplate --config config.yaml
echidna abdk/template.sol --contract EchidnaTemplate --test-mode assertion
```

Config file:

```yaml
# Can set testMode to property or assertion mode depending on how you want to test the system's properties
testMode: assertion
# You should ALWAYS set corpusDir to track coverage
corpusDir: corpus
```
