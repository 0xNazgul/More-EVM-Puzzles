# EVM Puzzle Solutions 
This is of a walkthrough for [daltyboy11's More EVM Puzzles](https://github.com/daltyboy11/more-evm-puzzles). I suggest you try them on your own and then come to this for hints and the answers.

## **Puzzle 1**
**HINT:** Look at the position of second `JUMPDEST`.
<details>
<summary>Answer</summary>
<p>

You want to give the value of 71 and calldata of 0xff (one bit)so that when the result (47^1) of EXP is given to JUMP it goes to the second `JUMPDEST`.

You should get a .json output in the solutions folder like so:

```js
{"value":71,"data":"0xff"}
```
</p>
</details>

## **Puzzle 2**
**HINT:** You need to create a contract that returns something, look before EQ.
<details>
<summary>Answer</summary>
<p>

This one is pretty simple it creates a contract, calls it, checks its `RETURNDATASIZE` and if it's EQ to 0a. This then gets a 1 on the stack which is needed for `JUMPI`, which takes 2 stack inputs. This first is the counter, where the `JUMPDEST` is that you want to go to. Second, is 1 or 0, if 1 it does the jump, if 0 it continues to the next instruction. So now we have the 1 on the stack and it pushes 1F for us it's solved.

You should get a .json output in the solutions folder like so:
```js
{"data":"0x600A80600080396000F3","value":0}
```
</p>
</details>

## **Puzzle 3**
**HINT:** You need to create a contract again, only this time it needs to store something at `0x05` in the puzzle's storage.
<details>
<summary>Answer</summary>
<p>

This one is simple too all we need to do is create a contract so that when it's `DELEGATECALL` it stores aa in `0x05` of the puzzles storage. So that when `SLOAD` is reached it loads `aa` and then EQ can put the 1 on the stack that's needed for the `JUMPI`. 

You should get a .json output in the solutions folder like so:
```js
{"data":"0x6460AA600555600052600A601BF3","value":0}
```
</p>
</details>

## **Puzzle 4**
**HINT:** You need to `CREATE` a contract with a balance divisible by 2.
<details>
<summary>Answer</summary>
<p>

The contract will be an init code that will `DIV` the incoming `CALLVALUE` by 2 of the `ORIGIN`, put on `GAS` and then make a `CALL` to send the half. `CREATE` will return and your created address should have 2 wei now. `4/2 = 2` and end up having `EQ` push 1 on the stack to then be used in `JUMPI` solving the puzzle.

You should get a .json output in the solutions folder like so:
```js
{"value":10,"data":"0x600080808060023404325AF15060006000F3"}
```
</p>
</details>

## **Puzzle 5**
**HINT:** You need a calldata `GT` 0x20 and see what `MSIZE` returns.
<details>
<summary>Answer</summary>
<p>

This one is pretty easy when you look at what `MSIZE` returns it's 0x40. Since it does a `SUB` after (`CALLDATASIZE - 0x40`), we need a `CALLDATASIZE` of `0x40 - 0x03` = 61 which is also `GT` 0x20.

You should get a .json output in the solutions folder like so:
```js
{"data":"0x01020304050607080910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061","value":0}
```
</p>
</details>

## **Puzzle 6**
**HINT:** Find a way to overflow the `ADD`, notice how it ends in `0` (you may want to play with [this](https://www.rapidtables.com/convert/number/decimal-to-binary.html)).
<details>
<summary>Answer</summary>
<p>

15 in hex is `f` which is almost there and 16 is `10` which overflows it to result with `0`. This is close but you need a result of `1` so 17 will do the job.

You should get a .json output in the solutions folder like so:
```js
{"value":17,"data":"0x"}
```
</p>
</details>

## **Puzzle 7**
**HINT:** You need to run through the loop until the difference of the gas before and after are equal to 166.
<details>
<summary>Answer</summary>
<p>

I honestly brute forced this at first but pretty much the idea is the loop need to run enough times in order to consume enough gas to pass the `EQ` at 18.

You should get a .json output in the solutions folder like so:

```js
{"value":4,"data":"0x"}
```
</p>
</details>

## **Puzzle 8**
**HINT:** Look at the last `SELFBALANCE` at 29 and question if you even need to `CREATE` anything?
<details>
<summary>Answer</summary>
<p>

This is a trick puzzle because you don't need to send any data or wei at all. By `CREATE`ing nothing (empty init code and runtime code) and not sending any wei the `SELFBALANCE` will return 0 and see if it's `EQ` to 0 thus passing the 1 you need to `JUMPI`.

You should get a .json output in the solutions folder like so:

```js
{"data":"0x","value":0}
```
</p>
</details>

## **Puzzle 9**
**HINT:** The puzzle uses `SHA3` which can't be reversed and `SHR` everything out but the first byte (essentially a brute force puzzle).
<details>
<summary>Answer</summary>
<p>

Keep guessing numbers until you get your first byte of the `SHA3` to be `0xa8`(We only want the first byte because the `SHR` of F8). There are many was to automate this and I just did such in a contract:
```js
function guessNumber() external pure returns(uint256) {
    for (uint i = 0; i < 100; ++i) {
        // Hashes i and only keeps the first byte
        bytes1 number = bytes1(keccak256(abi.encode(i)));
        bytes1 want = 0xa8;
        // Sees if the first byte is equal to what we want
        if (number == want) {
            return(i);
        } 
        continue;
    }
}
```

You should get a .json output in the solutions folder like so:
```js
{"value":47,"data":"0x"}
```
</p>
</details>

## **Puzzle 10**
**HINT:** Learning about bit-masking first may help you with this. After look at what happens with your data after both `AND` and `OR`.
<details>
<summary>Answer</summary>
<p>

The first 32 bytes are sent to memory with and pulled with `MLOAD`. The `F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0` with `AND` zero out the `b`'s `a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0a0`. The second 32 bytes are then pulled with `MLOAD` to have both used in `OR` giving us the desired result of `abababababababababababababababababababababababababababababababab` to have `EQ` put a 1 on the stack for `JUMPI`.


You should get a .json output in the solutions folder like so:
```js
{"data":"0xabababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababababab","value":0}
```
</p>
</details>

