# Notes

All of the following types of tests should be used.

## Unit Testing

Unit tests tend to cover just a _point_ on the line/space of possible inputs. Mostly covers the happy path. Might miss edge cases.

## Manual Analysis

Read the code. Think about the code. Write down bugs. Snapshot in time. Because it's not automated you have to re-analyze everytime code changes.

## Fully Automated Analysis

Slither. Static analysis. Super fast. Minimal effort. But doesn't catch everything.

## Semi-automated Analysis

Echidna. We have to write the structure and invariants or whatever, then it rips through thousands of variations.

### Fuzzing

Fuzzing is semi-automated analysis. In web2 you try to crash the program. In web3 you need to try to break things in some other way... We define some properties/invariants, and try to violate those. Invariants should always be true given some pre-conditions.

So we supply the contract, of course, and the invariant. For example, "user balance should never exceed token supply". Then echidna tries to break that.

1. generate random input
2. test something
3. if it fails, save input. else stop

### Intuition

1. Think of echidna as a EOA that fires off tx's.
2. Call _sequence_ of functions with random inputs in target _and_ inherited contracts. It might do 50 tx's to multiple functions, then check invariants.
3. Properties are resolved to "truthy" value.

### Usage

```zsh
echidna-test
```

### Property Testing

Basically Echidna will randomly call all our functions in our contract and execute this function between each call. And as long as this function always returns true then we are good. If it returns false then our invariant has broken and Echidna will save the call sequence that lead to the invariant breaking.

`echidna template.sol --contract TestToken`

```js
function echidna_total_supply() public returns(bool) {
  return balances[msg.sender] <= totalSupply()
}
```

We can set many of these functions/invariants. As many as we want.

### Assertion Testing

We are still testing a property, but we're doing it a bit differently. We can have multiple assert statements in the function and they will all need to be true in order for the test to pass.

`echidna template.sol --contract TestToken --test-mode assertion`

```js
function testPausable(uint256 amount) public view {
  assert(paused());
}
```

These functions can take input arguments as well and those values will be fuzzed. So echidna will input different values for `amount` in the function above.

### Defining Good Invariants

start small and iterate!

1. define invariants in english
2. write them in solidity
3. run echidna

**function-level invariants**

- testing single function
- can be tested in isolation
- associate property of addition / depositing tokens in contract

```js
contract TestMath is Math {
    function test_commutative(uint256 a, uint256 b) public {
        assert(add(a, b) == add(b, a));
    }
}
```

**system-level invariants**

- relies on deployment of large part of system
- invariants usually stateful
- ex. `user's balance < total supply`, yield monotonically increases

Testing these requires some set up:

- simple initialization: deploy everything in constructor
- complex: leverage your unit tests framework with etheno

We can set `require` in the assert function to put bounds on the input. Well, here we're just excluding zero. This is a function level invariant. The `_valid_buy` function should never allow `desiredAmount` to be greater than zero while the second arg is zero.

```js
function assert_no_free_token(uint desiredAmount) public {
  require(desiredAmount > 0);
  _valid_buy(desiredAmount, 0);
  assert(false) // this should never be reached
}
```

### Debugging with Events

If Echidna (for example) finds a way to break our code, we may then went to look more closely and see what the intermediate values are and why exactly the thing is failing. So we can define events which are emitted only in the failing case, then re-run echidna.

```js
event Debug(int128, int128);
//...
emit Debug(x, y);
```

Note: if we are testing something like a math library, we don't just want to test the addition function directly, we want to test properties that arise from addition. For example commutativity, associativity etc.

### Pre Conditions

Bound the input space.

```js
function div_test_not_commutative(int128 x, int128 y) public {
  // pre condition
  require(abs(x) != abs(y));
  // action
  int128 x_y = div(x, y);
  int128 y_x = div(y, x);
  // post conditions
  assert(x_y != y_x);
}
```

### Post Conditions

These are the "truths" you are testing. Test happy path and unhappy path.

```js
function div_test_not_commutative(int128 x, int128 y) public {
  // actions
  int128 x_y = div(x, y);
  int128 y_x = div(y, x);

  // post conditions
  if (abs(x) == abs(y)) {
    assert(x_y == y_x);
  } else {
    assert(x_y != y_x);
  }
}
```

### Optimization of Fuzzer Performance

Rather than just reverting, we could "fix" the inputs. Make them not equal. This is just to save compute cycles. Mod can be useful for this kind of thing because it let's us bound some value to a range.

```js
function div_test_not_commutative(int128 x, int128 y) public {
  // pre conditions / optimiziation
  if (abs(x) == abs(y)) {
    y = x + 1;
  }
  // action
  int128 x_y = div(x, y);
  int128 y_x = div(y, x);
  // post conditions
  assert(x_y != y_x);
}
```

## External Testing vs Internal Testing

### Internal

Use of inheritance to test the target contract. We create a new contract and _inherit_ the actual contract then do whatever we want.

**pros**

- easy to set up
- get the state and all public/external f'ns
- msg.sender is preserved

**cons**

- not good for complex systems
- mostly viable for single entrypoint systems

Echidna actually has 3 addresses that it uses. 0x1, 0x2, 0x3. msg.sender is preserved. These addresses are calling the system.

### External

Using _external calls_ to target the system. Uses middleman basically. Basically we build a contract which calls to our target contract and then we have echidna fuzz on that outer contract. So it will make EOA calls to our wrapper contract, which then calls the target system, and those tx's will NOT be from 0x1, 0x2, 0x3, they will come from that contract.

This is basically what forge/foundry does. We set up an external contract that we use to call our contract.

**pros**

- good for complex systems
- good for multi-entrypoint systems
- mostly used in practice

**cons**

- tricky to set up
- msg.sender _not_ preserved

Nice example. Echidna will fuzz the inputs of course. So it will try to stake a zillion different amounts. But only valid amounts ie. amounts the staker actually holds.

```js
function testStake(uint256 _amount) public {
    // Pre-condition
    require(tokenToStake.balanceOf(address(this)) > 0);
    // Optimization: amount is now bounded between [1, balanceOf(address(this))]
    uint256 amount = 1 + (_amount % (tokenToStake.balanceOf(address(this))));
    // State before the "action"
    uint256 preStakedBalance = stakerContract.stakedBalances(address(this));
    // Action
    uint256 stakedAmount = stakerContract.stake(amount);
    // Post-condition
    assert(stakerContract.stakedBalances(address(this)) == preStakedBalance + stakedAmount);
}
```

Note: echidna will test any functions which are public/external!

### Coverage

If we run the command with the config, and that config has the corpusDir set, then coverage file will be produced. We can then browse it to check our coverage. Or, if we can figure out how to get the VSCode coverage gutters extension to find the lcov file, then we can see right in the file what lines have been covered.

```zsh
echidna Template.sol --contract EchidnaTemplate --config config.yaml
```

Config file:

```yaml
# You should ALWAYS set corpusDir to track coverage
corpusDir: corpus
```

### Try Catch

This way the post condition runs even if the action fails. If we don't have the try/catch then the action reverts on unhappy path and the post condition never runs.

```js
// action
try stakerContract.stakedBalances(address(this)) returns (uint256 stakedBalance) {
    // post condition
    // make sure my staked balance has increased
    assert(stakedBalance == stakerContract.stakedBalances(address(this)));
} catch (bytes memory err) {
    // post condition
    assert(false);
}
```

### Tips

[fuzzing_tips](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/fuzzing_tips.md)

[faq](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/frequently_asked_questions.md)

[tips](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/echidna/basic)
